# [Sunday](https://app.hackthebox.com/machines/sunday)

```bash
rustscan 10.10.10.76
```
![Alt text](img/image.png)


We see the open ports (79,111,515,22022) for our target, let's do nmap scan.

```bash
nmap -A -sC -sV -p79,111,515,22022 10.10.10.76 -Pn
```


From here we can see logins to Finger service.

![Alt text](img/image-1.png)


Let's check finger service via 'finger' command.

![Alt text](img/image-2.png)


Let's use this [script](http://pentestmonkey.net/tools/finger-user-enum/finger-user-enum-1.0.tar.gz) to enumerate users from Finger service.

```bash
./finger-user-enum.pl -U /usr/share/seclists/Usernames/Names/names.txt -t 10.10.10.76
```


We also detected on nmap scan that port 22022 runs ssh server, let's try to login with sunny:sunday credentials.

![Alt text](img/image-3.png)


It worked, we are in.

![Alt text](img/image-4.png)


After enumeration, we find interestning file 'shadow.backup'

![Alt text](img/image-5.png)

We grab password hash from here, let's crack this with hashcat tool.

```bash
hashcat -m 7400 hash.txt --wordlist /usr/share/wordlists/rockyou.txt
```

![Alt text](img/image-6.png)


We find password that is 'cooldude!'.

![Alt text](img/image-7.png)

Now we can switch to 'sammy' user with this credentials (sammy: cooldude!)

![Alt text](img/image-8.png)


user.txt

![Alt text](img/image-9.png)


After making some enumeration, I see that we have privilege like this ( (ALL) ALL ).

For using this , I just do **`sudo -s`** and enter password of sammy user and be root.

![Alt text](img/image-10.png)


root.txt

![Alt text](img/image-11.png)