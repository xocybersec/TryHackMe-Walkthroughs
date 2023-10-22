TryHackMe <br>
Agent Sudo
---

Task 2
---
The initial nmap scan showing the open ports on the target machine. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/698def9f-1c09-4861-a0ee-1c77c8654210)

The page on port 80 shows this when visited: <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/35b63a95-e3d9-447c-9cf7-e75abaa3904f)

I tried messing around with the `User-Agent` string in Burpsuite but I didn’t have any luck. <br>
The hint on TryHackMe mentioned using an extension, so I went ahead and used that with <b>C</b> as the User-Agent and it worked. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/d8b0824b-5080-4f81-b158-abb0081d0ee9)

Now that I know there are two other agents to use, I tried them both. <br>
J didn’t return anything but R returned a comment saying if I wasn’t one of the 25 employees that I would be reported. <br>
That could be useful information for later that there are 25 total employees. <br>

---
Task 3
---

I wanted to see if there was a login to the FTP server with `chris` as the user since the password was said to be weak. <br>
I ran  `hydra` to see if I could get in with: <br>
```
hydra -l chris -P /usr/share/wordlists/rockyou.txt ftp://<target_IP>
``` 
Sure enough! <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/c0242bf0-edfc-497c-860b-4e1b6c5b5c8b)

Once in the FTP server I went ahead and used `mget` to get all the files. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/54db8f2c-13a7-4f1a-8d72-94725d8bea7e)

This is the content from the first file: <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/6b7f312e-f637-4811-a4ac-9202abd78e9f)

I used the tool `stegseek` to crack the password on the second file. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/9dd2fe9a-7566-4da4-9afb-e664c6e06fe7)

After getting the password I accessed the content: <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/50da08cc-c5fe-4024-9a9c-ac0f6030c6f4)

`binwalk` helped show the hidden folders in the PNG image. <br>
There was a zip file that needed to be cracked with `zip2john` then with `john` for the password. <br>
Inside the zip file there was a text file with an encoded string that `CyberChef` identified to be identical to the password from the stegseek results earlier. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/f8144211-6d75-4ed9-9e4a-2080fe89323f)

---
Task 4
---

First attempt was to try the FTP server with the found credentials, but that was unsuccessful, so SSH was next and was successful. <br>
The first command, as usual, is `sudo -l` so I can see what exactly the user can run as root and how to use that to my advantage. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/ef6a5de0-e8c5-401a-afef-711b4e60d6aa)

Time to see which version sudo is: <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/f2463bf2-5bce-4237-b3f6-845afafe9579)

On james’ profile these two files were there. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/88a7d2ca-5b03-4e8f-bfca-1dc1ee3ee659)

First flag found! <br>
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/00e99a6d-b144-442d-abeb-48f85aeae415)

---
Task 5
---

I used `searchsploit` and looked up the version of sudo I found and was brought to: <br>
```
https://www.exploit-db.com/exploits/47502
```
Which then showed me a command to use to escalate privileges to root. <br>
```
sudo -u#-1 /bin/bash
```
I was in as root! <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/584bf8e1-439b-4b4b-909a-46f20e515810)

And there was a text file in the root directory. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/f4d5d3ee-cc56-40d0-9745-f4819ef3e7f2)
