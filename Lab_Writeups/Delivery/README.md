# [Delivery](https://app.hackthebox.com/machines/delivery)

```bash
nmap -p- --min-rate 10000 10.10.10.222
```

![Alt text](img/image.png)


After knowing the open ports (22,80,8065), let's do greater nmap scan.

```bash
nmap -A -sC -sV -p22,80,8065 10.10.10.222 -Pn
```
![Alt text](img/image-1.png)

Let's look at the websites.

I see two hostname and add to my '/etc/hosts' file.

```bash
10.10.10.222 delivery.htb helpdesk.delivery.htb
```

![Alt text](img/image-2.png)


They gave me temporary credentials.

![Alt text](img/image-3.png)


3486967@delivery.htb

Then, I try to login via this email and password field I write id of task.

![Alt text](img/image-4.png)

![Alt text](img/image-5.png)


We also discovered web application running on (8065) port. 
For registration from here, I need to use **"@delivery.htb"** email which is valid for their authentication system.


Dr4ks1234$

![Alt text](img/image-7.png)

![Alt text](img/image-6.png)


While I login, I find credentials stuff here.

![Alt text](img/image-8.png)

maildeliverer:Youve_G0t_Mail! 

Let's try to use this creds to authenticate machine via SSH.
Hola , it worked.


user.txt

![Alt text](img/image-9.png)


While enumeration on machine , I find `config.json` file on "/opt/mattermost/config" directory which contains Mysql credentials.

![Alt text](img/image-10.png)


Here's our grabbed credentials.

mmuser:Crack_The_MM_Admin_PW


Let's login into Mysql via this credntials, find what we want.

We find 'mattermost' database and sensitive table which stores information about username and passwords.

![Alt text](img/image-11.png)

We grab root's user hashed password '$2a$10$VM6EeymRxJ29r8Wjkr8Dtev0O.1STWb4.4ScG.anuu7v0EFJwgjjO'

and try to crack this via `hashcat` tool.


```bash
hashcat -m 3200 hash.txt --wordlist /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule 

```

Credentials for root user is "root: PleaseSubscribe!21"


root.txt

![Alt text](img/image-12.png)