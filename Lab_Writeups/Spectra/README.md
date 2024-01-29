# [Spectra](https://app.hackthebox.com/machines/Spectra)

```bash
nmap -p- --min-rate 10000 10.10.10.229 -Pn 
```

![Alt text](img/image.png)

After discovering open ports, let's do greater nmap scan.

```bash
nmap -A -sC -sV -p22,80,3306 10.10.10.229
```

![Alt text](img/image-1.png)


There's home page after visiting this ip address about `Testing` field.

I see that this ip address is resolved into `spectra.htb` domain, let's add this into `/etc/hosts` file.


![Alt text](img/image-2.png)


While looking at `/testing` endpoint, I see a lot of files here.

![Alt text](img/image-3.png)


There's file called `wp_config.php.save` which have sensitive credentials.

![Alt text](img/image-4.png)

devtest: devteam01


While looking at main page, I can't find anything.

![Alt text](img/image-5.png)

Let's do `directory enumeration` via `gobuster` tool.

```bash
gobuster dir -u http://spectra.htb/main/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -x php,txt,csv,pdf,config,xml -t 40 
```

![Alt text](img/image-6.png)

While I try login via above credentials, it didn't worked, I changed username from `devtest` into `administrator`, and IT WORKED.

![Alt text](img/image-7.png)

Let's upload our webshell into some location for target web application.


I click `Plugins` -> `Plugin-Editor` and edit file called `akismet.php` , means I add my webshell code into here.

![Alt text](img/image-8.png)


Let's browse our webshell via `curl` command.

```bash
curl http://spectra.htb/main/wp-content/plugins/akismet/akismet.php?cmd=id
```

![Alt text](img/image-9.png)


Let's add our reverse shell here, but `URL-ENCODED`, I use [revshells](https://www.revshells.com/).

![Alt text](img/image-10.png)


I paste this payload into web application.


![Alt text](img/image-11.png)


I got reverse shell from port (1337).

![Alt text](img/image-12.png)


Let's make interactive shell.
```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
Ctrl+Z
stty raw -echo; fg
export TERM=xterm
export SHELL=bash
```

![Alt text](img/image-13.png)


I found interesting file called `autologin.conf.orig` on `/opt` directory.

![Alt text](img/image-14.png)

I read file called `passwd` from `/etc/autologin` directory.

![Alt text](img/image-15.png)

Maybe this password of one user from this time.

Password: SummerHereWeCome!!

There's possible users are listed below.
```bash
chronos
katie
nginx
root
user
```

Let's check this password via this user means `Password Spray` attack by using `crackmapexec` tool.

```bash
crackmapexec ssh 10.10.10.229 -u users -p 'SummerHereWeCome!!' --continue-on-success
```

![Alt text](img/image-16.png)


That's password of `katie` user.

katie: SummerHereWeCome!!


user.txt

![Alt text](img/image-17.png)


For privilege escalation, I just run `sudo -l` command and see privileges of this user.

![Alt text](img/image-18.png)


I found this [blog](https://isharaabeythissa.medium.com/sudo-privileges-at-initctl-privileges-escalation-technique-ishara-abeythissa-c9d44ccadcb9) for privilege escalation.

This binary executes files from `/etc/init` folder.

I change content of `test.conf` file from this folder like below.

So that I give `SUID` privilege to `/bin/bash` binary to get root shell.

Let's run `initctl` binary.
```bash
sudo /sbin/initctl start test
```

![Alt text](img/image-20.png)

If we type `/bin/bash -p` to terminal, we got root shell.

root.txt

![Alt text](img/image-19.png)