# tomghost

[Room Link](https://tryhackme.com/room/tomghost)

Nmap as usual

`nmap -A -sV -vv -T4 10.10.54.146`

```
PORT     STATE SERVICE    REASON  VERSION
22/tcp   open  ssh        syn-ack OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 f3:c8:9f:0b:6a:c5:fe:95:54:0b:e9:e3:ba:93:db:7c (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDQvC8xe2qKLoPG3vaJagEW2eW4juBu9nJvn53nRjyw7y/0GEWIxE1KqcPXZiL+RKfkKA7RJNTXN2W9kCG8i6JdVWs2x9wD28UtwYxcyo6M9dQ7i2mXlJpTHtSncOoufSA45eqWT4GY+iEaBekWhnxWM+TrFOMNS5bpmUXrjuBR2JtN9a9cqHQ2zGdSlN+jLYi2Z5C7IVqxYb9yw5RBV5+bX7J4dvHNIs3otGDeGJ8oXVhd+aELUN8/C2p5bVqpGk04KI2gGEyU611v3eOzoP6obem9vsk7Kkgsw7eRNt1+CBrwWldPr8hy6nhA6Oi5qmJgK1x+fCmsfLSH3sz1z4Ln
|   256 dd:1a:09:f5:99:63:a3:43:0d:2d:90:d8:e3:e1:1f:b9 (ECDSA)
|_ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBOscw5angd6i9vsr7MfCAugRPvtx/aLjNzjAvoFEkwKeO53N01Dn17eJxrbIWEj33sp8nzx1Lillg/XM+Lk69CQ=
53/tcp   open  tcpwrapped syn-ack
8009/tcp open  ajp13      syn-ack Apache Jserv (Protocol v1.3)
| ajp-methods: 
|_  Supported methods: GET HEAD POST OPTIONS
8080/tcp open  http       syn-ack Apache Tomcat 9.0.30
|_http-favicon: Apache Tomcat
|_http-open-proxy: Proxy might be redirecting requests
|_http-title: Apache Tomcat/9.0.30
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_k
```

We see a webpage at port 8080

![image](https://github.com/user-attachments/assets/83d27bca-55c4-4a2e-a154-f3475c1a086e)

`Apache Tomcat/9.0.30`

Upon searching it's exploits
we come across exploitdb, which indicates there's an exploit in metasploit

![image](https://github.com/user-attachments/assets/806befc7-ef70-4773-897d-24328cc4370f)

getting into metasploit console

![image](https://github.com/user-attachments/assets/098ebada-2e16-4b7a-aacf-de8ff44641a3)

we notice that it needs an ajp server, which is present in our 8009 port, so all good

![image](https://github.com/user-attachments/assets/dc177a7e-a267-48da-91dc-ea1d00aab91c)

got them

`skyfuck:8730281lkjlkjdqlksalks`

I looked around for a login page but couldn't find any
Hence, tried for ssh as it was the only available option left

![image](https://github.com/user-attachments/assets/a739ce57-7a66-4344-ac13-561f11142ca8)

worked

![image](https://github.com/user-attachments/assets/22abce8e-6346-4121-afce-1e7f0ae22e73)

imported some credentials

first converting .asc to something john can read

`usr/sbin/gpg2john tryhackme.asc > gpgg.txt`

`tryhackme.asc` is the gpg file to crack, most likely encrypted with a passphrase. The .asc extension suggests that the file is an ASCII-armored (text) version of the encrypted data.
now `gpgg.txt` can be processed by john

`/usr/sbin/john --wordlist=/usr/share/wordlists/rockyou.txt gpgg.txt`

John the Ripper is trying to brute-force the passphrase protecting the GPG file using the rockyou.txt wordlist.

![image](https://github.com/user-attachments/assets/3c132b2e-c486-4051-a8c3-271f86b870c6)

Moving on, as now we got the passphrase

`gpg --import tryhackme.asc`

imports the key to gpg keyring

`gpg --decrypt credential.pgp`

`credential.pgp` is the file to decrypt from the imported keys, the final goal of the process

![image](https://github.com/user-attachments/assets/ea15cc46-4590-4d46-8121-2f45ff7c5bd9)

We got merlin's password
I tried to switch users from skyfuck, but it simply wasn't possible or I didn't know how to
So I ssh'd using merlin's creds directly

Got the user flag

![image](https://github.com/user-attachments/assets/fef99e9c-a4de-4203-a071-57bcff1ee67e)

unfortunately merlin didn't have sudo privileges either

![image](https://github.com/user-attachments/assets/df3d3087-5aa5-4391-9e2a-75388988a86c)

except for zip

time for GTFObins

![image](https://github.com/user-attachments/assets/2f70debf-0544-4efa-a7a0-d6405bfba466)

![image](https://github.com/user-attachments/assets/a98adb55-ab68-47e3-9d85-41f7ab4fb7da)

Done












