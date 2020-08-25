# pWnOS v2.0
![](https://i.imgur.com/rzZ5szI.png)

![](https://i.imgur.com/aGFjCnK.png)
Set the attack platform to `10.10.10.0/24`

![](https://i.imgur.com/wplUwGF.png)
`ifconfig eth0 10.10.10.200 netmask 255.255.255.0`
Host is `10.10.10.200`
Victim is `10.10.10.100`

`nmap -sS -sV -p- -oN NmappWnOSLv2.txt 10.10.10.100`
![](https://i.imgur.com/MHOGgUp.png)

Home
![](https://i.imgur.com/1NYnLEf.png)

Register
![](https://i.imgur.com/n83pZa9.png)

Login
![](https://i.imgur.com/P99TwFu.png)

`nikto -h 10.10.10.100`
![](https://i.imgur.com/n1MaC3q.png)
A php web service and lots info about the victim system.

`dirb http://10.10.10.100`
![](https://i.imgur.com/TJ1s447.png)
We found a blog path by brutal force enumerate the paths.

![](https://i.imgur.com/xdwfsLV.png)
![](https://i.imgur.com/zfXZcEA.png)
![](https://i.imgur.com/5BdVXUx.png)
![](https://i.imgur.com/Zfa7tSs.png)
Semms a login info, not sure how to use it.

![](https://i.imgur.com/v1P3K7p.png)
![](https://i.imgur.com/VevHIKb.png)
![](https://i.imgur.com/4OtCwpH.png)
Then we found the project name of this blog: `SPHPBlg`, version: `0.4.0`, try searchsploit.

`searchsploit SPHPBlog`
![](https://i.imgur.com/aZ8lI2i.png)

Seems not that useful, let's back to try upload reverse shell.

![](https://i.imgur.com/vvMc6mg.png)
We don't have account to login, then I try to search SPHPBlog login exploit.
Here I found: https://www.rapid7.com/db/modules/exploit/unix/webapp/sphpblog_file_upload

![](https://i.imgur.com/rB9IZvv.png)
Search to use module in msfconsole.

![](https://i.imgur.com/cL4X9ed.png)
After setting required info, I got a new account which allows me to login.

![](https://i.imgur.com/eUMfbws.png)
![](https://i.imgur.com/RSkft0W.png)
![](https://i.imgur.com/VR8f113.png)
Then I can use upload function.

Download the php-reverse-shell sample from [pentestmonkey](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php).
![](https://i.imgur.com/UPxQPtO.png)

![](https://i.imgur.com/gu5ApKq.png)
![](https://i.imgur.com/LaJNtcD.png)

Before execute reverse shell, setup nc listener by `nc -nvlp 8888` on host, then press the link on browser to make php script run.

![](https://i.imgur.com/Ge9shAW.png)

First, get some info.
![](https://i.imgur.com/9eLLwrW.png)

Then switch to web folder to find some database connect config.
![](https://i.imgur.com/TsaZU9G.png)

cat the config out.
![](https://i.imgur.com/BDbdZxa.png)

![](https://i.imgur.com/VghbzPR.png)
Seems not the one we looking for.

Here's another login info:
![](https://i.imgur.com/Ml5wKYp.png)
![](https://i.imgur.com/5hsKxHw.png)

![](https://i.imgur.com/CPtXhKA.png)
Seems we got the correct one here.

![](https://i.imgur.com/dmnq4Gr.png)
