# Kioptrix Level 1
Insert `ethernet0.connectionType = "nat"` to .vmx, and change all `bridged` in the .vmx file of the VM before use VMware to open it.
![](https://i.imgur.com/Q5fiSSv.png)
Host is `192.168.232.145`
![](https://i.imgur.com/pLd1dnk.png)
Victim is `192.168.232.146`
`nmap -sS -sV -A -p- -oN NmapKioptrixLv1.txt 192.168.232.146`
![](https://i.imgur.com/rfBbahj.png)
![](https://i.imgur.com/aW6i7O8.png)
Nothing in 80/443, just a test page.
![](https://i.imgur.com/fRQiCif.png)
Let's switch to recon the port 139 samba: `nmap -p139 --script vuln 192.168.232.146`
![](https://i.imgur.com/THu864x.png)
smb-vuln-cve2009-3103 maybe vul, we use exploit-db to try to find some PoC to use.
[Samba < 2.2.8 (Linux/BSD) - Remote Code Execution](https://www.exploit-db.com/exploits/10)
Download and compile it with gcc, then execute it you will find the help message, use flag `-b` with `0` argv.
![](https://i.imgur.com/siURRca.png)
Now you get the shell.

![](https://i.imgur.com/tnhZGPy.png)
Look at `/etc`, `/var/www`, `/tmp`, `/home`, `/var/mail`
![](https://i.imgur.com/sv63xS2.png)
Afterall, `cat` the `root` under `/var/mail`
![](https://i.imgur.com/6ksgkEr.png)

