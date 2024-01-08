# [Blue](https://app.hackthebox.com/machines/blue)

```bash
nmap -p- --min-rate 10000 10.10.10.40 -Pn
```

![Alt text](img/image.png)

After finding many open ports, I grab ports(135,139,445) from here to search vulnerabilities.

```bash
nmap -A -sC -sV -p135,139,445 10.10.10.40
```

![Alt text](img/image-1.png)



From here, I see that 'SMB' is open, let's look at via `smbmap` tool.

```bash
smbmap -H 10.10.10.40 -u dr4ks -p dr4ks
```

![Alt text](img/image-2.png)


Let's access `SHARE` and `USERS` shares of SMB.

![Alt text](img/image-3.png)


I found nothing in here.

Then, again I searched Vulnerability scan for port (445).

```bash
nmap -p 445 -script vuln  10.10.10.40
```

![Alt text](img/image-4.png)

From here, I see that target machine is vulnerable to 'MS-17-010'.

Let's try to access machine via this vulnerability from `msfconsole`.

![Alt text](img/image-5.png)


I gained a shell.

![Alt text](img/image-6.png)


user.txt

![Alt text](img/image-7.png)


root.txt

![Alt text](img/image-8.png)