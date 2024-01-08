# [Blocky](https://app.hackthebox.com/machines/blocky)

```bash
nmap -sT -p- --min-rate 10000 10.10.10.37 -Pn
```

![Alt text](img/image.png)


Let's do nmap for open ports.

```bash
nmap -A -sC -sV -p21,22,80,25565 10.10.10.37 -Pn
```

![Alt text](img/image-1.png)

I also add IP address to my '/etc/hosts' file for resolving purposes.

![Alt text](img/image-2.png)


Directory-brute forcing.
```bash
gobuster dir -u http://blocky.htb/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 40 -x php
```

![Alt text](img/image-3.png)

Here we have possible username 'notch' as because there we have post from 'notch' user.

![Alt text](img/image-6.png)


I found '/plugins' directory. Let's check it.

![Alt text](img/image-4.png)


Let's download these files and enumerate this files via `jd-gui` tool.

Hola

![Alt text](img/image-5.png)


Here, we find **HARD-CODED credentials.** Let's try to login via SSH.

root: 8YsqfCTnvxAUeduzjNSXe22

notch : 8YsqfCTnvxAUeduzjNSXe22


That's it credentials (for notch worked).

![Alt text](img/image-7.png)


user.txt

![Alt text](img/image-8.png)


Let's check sudo rights for notch user. `sudo -l` command

![Alt text](img/image-9.png)



As we see    (ALL : ALL) ALL , let's run `sudo -s` command to be root user.


root.txt

![Alt text](img/image-10.png)