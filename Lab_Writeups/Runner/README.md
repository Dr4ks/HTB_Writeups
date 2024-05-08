# [Runner](https://app.hackthebox.com/machines/runner)

```bash
nmap -p- --min-rate 10000 10.10.11.13 -Pn
```

![alt text](img/image.png)

After detection of open ports, let's do greater nmap scan for these ports.

```bash
nmap -A -sC -sV -p22,80,8000 10.10.11.13 -Pn 
```

![alt text](img/image-1.png)


From nmap scan result, I see that this ip address is resolved into `runner.htb`, let's add this into `/etc/hosts` file for resolving purposes.



While opening web application on port `80`, it returns such an page.

![alt text](img/image-2.png)

Let's do `Subdomain Enumeration` via `wfuzz`.

```bash
wfuzz -u http://runner.htb -H "Host: FUZZ.runner.htb" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt
```

![alt text](img/image-3.png)


Let's add `teamcity.runner.htb` domain name to `/etc/hosts` file.


While I open `teamcity.runner.htb`, it returns `Teamcity` web application and login form.

![alt text](img/image-4.png)


Version is `Version 2023.05.3 (build 129390)`.


I searched publicly known exploit for this version of web application. That's [CVE-2023-42793](https://github.com/H454NSec/CVE-2023-42793)

Let's use this.

```bash
python3 CVE-2023-42793.py -u http://teamcity.runner.htb/
```


![alt text](img/image-5.png)


H454NSec3879:@H454NSec

Let's use this credentials to login into system.


I find one last backup, let's download this and analyze data inside of this.

![alt text](img/image-6.png)



Once I extract data, I find `id_rsa` from here, let's try to use this to authenticate into machine.


![alt text](img/image-7.png)



Let's change mode of this `id_rsa` and use for authentication via `ssh`.

```bash
chmod 600 id_rsa
ssh -i id_rsa
```

![alt text](img/image-8.png)


I brute-force possible users from list of Teamcity application and `john` is correct one for me.

![alt text](img/image-9.png)


user.txt

![alt text](img/image-10.png)


I also find `database_dump` folder after extraction and read `users` file which contains user's information.

![alt text](img/image-11.png)


Let's take password hash of `matthew` user and tries to crack with `hashcat`.

```bash
hashcat -m 3200 hash.txt --wordlist /usr/share/wordlists/rockyou.txt
```

![alt text](img/image-13.png)


matthew: piper123

For `Privilege Escalation`, I just run `netstat -ntpl` to see open ports.

![alt text](img/image-12.png)


I see port `9000`, let's do `Port Forwarding` to see what application is running.

```bash
ssh -L 9000:localhost:9000 -i id_rsa john@runner.htb
```

![alt text](img/image-14.png)


While I open web application port `9000`, it opens up `Portainer`. I authenticat myself into here via `matthew` credentials.

![alt text](img/image-15.png)


Then, I find this [blog](https://rioasmara.com/2021/08/15/use-portainer-for-privilege-escalation/) for `Privilege Escalation`.

I also use vulnerability whose id is `CVE-2024–21626`.

![alt text](img/image-16.png)

![alt text](img/image-17.png)


I deploy container after setting this configuration.


I can read all system sensitive files belong to `root` user.

```bash
cat ../../../../../../root/root.txt
cat ../../../../../../root/.ssh/id_rsa
```

![alt text](img/image-18.png)


root.txt

![alt text](img/image-19.png)



