# [Mirai](https://app.hackthebox.com/machines/mirai)

```bash
nmap -p- --min-rate 10000 10.10.10.48 -Pn
```

![Alt text](img/image.png)

```bash
nmap -sC -sV -p22,53,80 10.10.10.48 -Pn
```

![Alt text](img/image-1.png)


Directory brute-forcing

```bash
gobuster dir -u http://10.10.10.48 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -x php -t 40
```

We found nothing interesting for us.


Default credentials for RasbperryPI
![Alt text](img/image-2.png)

pi: raspberry

Let's try to connect via SSH, and it worked.

![Alt text](img/image-3.png)


user.txt

![Alt text](img/image-4.png)


I check privileges via `sudo -l` command.

![Alt text](img/image-5.png)


I escalate user to root just one command `sudo -s` command.

Let's look at all mounting

```bash
df -h
```

![Alt text](img/image-6.png)


Let's read '/dev/sdb' via `strings` command.

![Alt text](img/image-7.png)


root.txt is here, we can see flag

![Alt text](img/image-8.png)