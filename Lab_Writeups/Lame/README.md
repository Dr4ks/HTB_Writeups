# [Lame](https://app.hackthebox.com/machines/Lame)

```bash
rustscan 10.10.10.3 
```
![rustscan](img/image.png)


After knowing the ports (21,22,139,445), let's run nmap
```bash
nmap -p21,22,139,445 -sC -sV 10.10.10.3
```
![Alt text](img/image-1.png)


Samba 3.0.20-Debian
![Alt text](img/image-3.png)


Use this vuln on metasploit.
![Alt text](img/image-2.png)


user.txt

![Alt text](img/image-4.png)


root.txt

![Alt text](img/image-5.png)
