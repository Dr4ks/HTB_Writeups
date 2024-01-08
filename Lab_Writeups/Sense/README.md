# [Sense](https://app.hackthebox.com/machines/sense)

```bash
nmap -p80,443 -A -sC -sV 10.10.10.60 -Pn
```
![Alt text](img/image-1.png)


Let's do directory brute-forcing.
```bash
ffuf -u https://10.10.10.60/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt 
```

![Alt text](img/image-2.png)

Here, we got 'system-users.txt' file and content of this like.
![Alt text](img/image-3.png)


Default password credentials:  rohit:pfsense
![Alt text](img/image-4.png)


We login with above credentials.
![Alt text](img/image-5.png)


We learn that 2.1.3-RELEASE version is used by pfsense.

I already found exploit of this (CVE-2016-10709)=>

![Alt text](img/image-7.png)


Let's use msfconsole. Due to attack vector, we will be root user while injecting payload. That's why we can read both user.txt and root.txt flags.

![Alt text](img/image-8.png)
