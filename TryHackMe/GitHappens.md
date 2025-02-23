# Git Happens

[Room Link](https://tryhackme.com/room/githappens)

Nmap as usual

`nmap -A -vv -T4 10.10.170.192`

![image](https://github.com/user-attachments/assets/fe25cd6a-95f8-4a07-bb1d-64aa4aa88cc3)

I never came across git previously but I lurked around and found [GitTools](https://github.com/internetwache/GitTools)

We will use dumper from the tool to install everything to our local machine

![image](https://github.com/user-attachments/assets/5b9ce8da-e31f-47c7-a8a9-eda06a412b0e)

After installing `git log` to see what happened

![image](https://github.com/user-attachments/assets/598da2ba-9289-4ac1-b3a2-a738fe9a04ca)

`git log --patch` to see details of each commit

![image](https://github.com/user-attachments/assets/427ea8b2-cbd8-4c6e-9a77-642f859e7087)

and finally we find the password

![image](https://github.com/user-attachments/assets/d541a859-4faf-4a67-a1f3-5ae188fb3424)



