# [Popcorn](https://app.hackthebox.com/machines/popcorn)

```bash
nmap -sT -p- --min-rate 10000 10.10.10.6 -Pn
```

![Alt text](img/image.png)


```bash
nmap -sC -sV -p22,80 10.10.10.6 -Pn 
```

![Alt text](img/image-1.png)


Directory brute-forcing

```bash
gobuster dir -u http://10.10.10.6 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -x php -t 40 
```
![Alt text](img/image-4.png)


I find a directory called '/torrent', let's enumerate here.

I login here,

![Alt text](img/image-2.png)


I try to upload a file here, and I get a reverse shell.

While I upload clean php-reverse-shell, it doesn't allow me.

![Alt text](img/image-3.png)


Let's try to bypass it and upload webshell.


For this, I **download sample torrent file**, and upload this , while I upload this, **I click to EDIT torrent file** change the content of file with php reverse shell.

I upload malicious php file into machine by doing below steps.

    1. change file extension to .php.gif 
    2. Add 'GIF89a;' this string at the beginning of the file.
    3. Change 'Content-Type' to 'image/gif' in request header.


![Alt text](img/image-5.png)


![Alt text](img/image-6.png)

While **browsing malicious php** file on '/torrent/upload' directory, I get a reverse shell.

![Alt text](img/image-8.png)

![Alt text](img/image-7.png)


I spawned a interactive shell.

```bash
python -c 'import pty;pty.spawn("/bin/bash")'
Ctrl + Z
stty raw -echo;fg
export TERM=xterm
export SHELL=bash
```

![Alt text](img/image-9.png)


user.txt

![Alt text](img/image-12.png)



After enumeration machine, I see that it is vulnerable to 'DirtyCow' vulnerability. As because below version.

![Alt text](img/image-10.png)


I search for exploit on kali, and I find a script.

![Alt text](img/image-11.png)


Let's upload this script into machine, and run it.

![Alt text](img/image-14.png)

![Alt text](img/image-13.png)


After downloading C file, let's compile it and execute.

![Alt text](img/image-15.png)



root.txt

![Alt text](img/image-16.png)