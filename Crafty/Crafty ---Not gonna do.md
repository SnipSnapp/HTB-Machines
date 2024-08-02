scans:
```sh
nmap crafty.htb -p 80,25565 --script=http-vhosts
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-06-05 18:35 CDT
Nmap scan report for crafty.htb (10.129.230.193)
Host is up (0.052s latency).

PORT      STATE SERVICE
80/tcp    open  http
| http-vhosts: 
|_128 names had status 301
25565/tcp open  minecraft
```
Gobuster scan. 
yields nothing
And apparently on port 25565 this is a minecraft error.
let's view the source.
```firefox
view-source:http://crafty.htb/coming-soon
```
```html
<!DOCTYPE html>
<html>
<head>
	<title>Crafty - Official Website</title>
	<meta name="viewport" content="width=device-width, initial-scale=1.0">
	<meta charset="utf-8">
        <link rel="icon" type="image/x-icon" href="/img/favicon.ico">
	<link rel="preconnect" href="https://fonts.googleapis.com">
	<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
	<link href="https://fonts.googleapis.com/css2?family=Open+Sans:wght@400;700&display=swap" rel="stylesheet">
	<link rel="stylesheet" href="/css/stylesheet.css">
</head>
<body>
	<div class="container">
		<div class="logo">
			<!-- In the img folder, upload your logo -->
			<!-- Make sure you name it 'logo.png' or update the code below -->
			<img src="/img/logo.png" alt="Crafty logo">
		</div>

		<div class="items">
			<!-- Replace # with your store URL -->
			<a class="item store">
			<div>
				<img src="/img/coming-soon.png" alt="Minecraft store icon" class="img">
			</div>
			</a>
		</div>
	</div>

	<script src="https://ajax.googleapis.com/ajax/libs/jquery/3.6.0/jquery.min.js"></script>
	<script src="/js/firefly.js" type="text/javascript"></script>
	<script src="/js/main.js" type="text/javascript"></script>
</body>
</html>
```
Yeah this involves downloading tlauncher for minecraft, and i'm not about that life.
