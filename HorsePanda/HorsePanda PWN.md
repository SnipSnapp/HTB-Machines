```sh
PORT   STATE SERVICE VERSION
25/tcp open  smtp    hMailServer smtpd
| smtp-commands: mail.smallestcorp.htb, SIZE 20480000, AUTH LOGIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
80/tcp open  http    Microsoft IIS httpd 10.0
|_http-title: GOV.HTB
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
Service Info: Host: mail.smallestcorp.htb; OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 20.16 seconds
```

info@gov.htb.  -- Noted email address from web page
