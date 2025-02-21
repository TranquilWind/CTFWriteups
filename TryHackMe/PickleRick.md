# Pickle Rick


[Room Link](https://tryhackme.com/room/picklerick)

I always start off with nmap scanning 
But for this ctf, this webpage is displayed 
![image](https://github.com/user-attachments/assets/e0f9f77a-5994-4494-8be0-6402991ee0e1)

Anyways, for the nmap results:
`nmap -A -O -sV -vv -T4 10.10.155.42`

![image](https://github.com/user-attachments/assets/4ba81876-9f7e-44d2-b74c-a3884185d0c8)

not of much help

Carrying on,

As for web enumeration, checking standard things like page source, robots file, license file, login, registration page etc..
Moving forward with that

On Inspecting:

![image](https://github.com/user-attachments/assets/ac9864ce-5731-4be1-89df-cc483e522b94)

`R1ckRul3s`

On checking for robots file:

![image](https://github.com/user-attachments/assets/98c51a59-6d76-4d0c-897e-4b5f8e81dd9c)

`Wubbalubbadubdub`


and finally found the login page

![image](https://github.com/user-attachments/assets/1cdc8d67-7c56-4947-86ad-a2bf9634a10a)

we can find these things alternatively using gobuster or dirbuster but when tried it was nothing more than just a waste of time

![image](https://github.com/user-attachments/assets/9b278ad3-a011-4f23-948d-f991a6a604f9)

![image](https://github.com/user-attachments/assets/685e855b-ca97-4fcb-9a66-21189033ace5)

Doesn't get better

For the login page I tried Hydra but no luck

`hydra -l R1ckRul3s -P rockyou.txt http://10.10.155.42/login.php:username=^USER^&password=^PASS^:Invalid username or password.`

but then I realized there was one more info in the form of wubbalubbadubdub
So I tried and it worked

![image](https://github.com/user-attachments/assets/53c92b2e-03b3-4b3a-b592-98778a5a255f)

`ls`

![image](https://github.com/user-attachments/assets/bd063474-6227-4a90-889c-c28d253cc33e)

`cat Sup3rS3cretPickl3Ingred.txt`

![image](https://github.com/user-attachments/assets/b72d805b-855c-430d-ae55-cf524e3c20ec)

cat is blocked?
gotta bypass it

![image](https://github.com/user-attachments/assets/5af9ef0a-8c95-4142-ad03-9ff54595f573)

Other menus in the page also don't seem to work

I then explored other ways to display file contents I came across few like `less`, `more`, `head`, `tail` and also `awk '{print}' filename.txt` out of these only  `less` and `awk` seemed to work.
`less Sup3rS3cretPickl3Ingred.txt`

![image](https://github.com/user-attachments/assets/59bb23a4-4846-4f30-bbec-e3aa72b7c5a8)

Got the first ingredient
Now, there was a clue file
`less clue.txt`

![image](https://github.com/user-attachments/assets/3467ec47-7474-493b-9cb5-09aca9bc596e)

Says to explore more

On checking the base directory:
`ls /`

![image](https://github.com/user-attachments/assets/3c3c5870-9760-407c-9c25-24eb61de7693)

Explore everywhere and head to 
`ls /home/rick`

![image](https://github.com/user-attachments/assets/1fd14bf4-0741-48b7-b67a-1ace90202ed8)


Haha yes second ingredient
`less /home/rick/second\ ingredients`

![image](https://github.com/user-attachments/assets/f49bc343-3822-4b87-a4c2-23c7b848745c)

After exploring other directories (where I couldn't find the third flag) I realized it should be in directories which require sudo privileges 

Checking permissions:
`sudo -l`

![image](https://github.com/user-attachments/assets/4b1c6b27-054c-4e5d-a8ba-5872b36cdaa2)

The output of the `sudo -l` command indicates that it is `www-data` user and has `NOPASSWD` privileges to run all commands as any user on the. This means any command can be executed with sudo without being prompted for a password.

`sudo ls /root`


![image](https://github.com/user-attachments/assets/bdf0ba5e-4972-4bcd-a1ae-8098aaa42eff)

Root is accessible now

`sudo less /root/3rd.txt`

![image](https://github.com/user-attachments/assets/b7e44eb8-f4fe-443d-b1e3-ec9647917574)

Done.
















