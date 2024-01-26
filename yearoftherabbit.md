TryHackMe <br>
Year of the Rabbit
---

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/72f41e89-d46f-463e-ae8d-fcf68fdaa0b3)

# Reconnaissance/Scanning:
Let’s start things off with a network scan to see which ports are open and the services running on each.
```
$ nmap -A -O -sC -sV -p- <machine_IP>
 ```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/ac275401-b55f-46cd-bd2b-114e3393922c)

Scanning for directories with Gobuster: 

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/34a8c51f-4e7d-4f47-b1d0-3e9eefc99e55)

Contents of /assets directory:
 
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/3edb0474-4a30-4090-8a49-26b22c1780f9)

Contents of /css directory:

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/e4ed9b88-a267-4f99-a99d-2c48afce3876)

When visiting /sup3r_s3cret_fl4g.php

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/5f49cc95-4c58-4dad-a1fc-abfb4c9ca2e9)

Okay, i’ll use the devtools to inspect this page.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/f5feaf14-ab2f-41b2-ae53-c1308ef31b08)

That shows that the script makes the video from /assets start playing on youtube. <br>
Now, navigating to the Network tab in devtools, I see the next place I need to look at.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/9b2d9db8-371c-463c-abc5-0680e54d2d97)

When viewing that directory there’s an image file. There might be some steganography to do..

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/e364c303-38d9-4897-8f90-4f3c75a15a21)

I tried using a few tools on the image file; testing for steganography and also using tools like exiftool, binwalk, and strings. <br>
When I used strings, I found the username for the FTP server!

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/ade13569-9465-4c58-931f-d499d0d0027d)

The possible passwords continued for a while, so I made them into a wordlist and fired up hydra to bruteforce the FTP server.
```
$ hydra -l ftpuser -P <password-file> <machine_IP> ftp
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/f9affabd-aafc-47bd-9b4a-f60f6cb5665b)
 
After logging in to the FTP server I found a text file.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/67b4e5a1-1258-4b55-98d2-395408d83d4a)

I downloaded that file and viewed the contents.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/74dacebe-2e37-45d5-b2eb-e0eedb740df4)

At least they’re not plaintext.. Although, they are part of a cipher that’s somewhat common. <br>
If you like to do CTFs I’m sure you know what this is. <br>
If you don’t it’s called Brainfuck. <br> <br>

“Brainfuck (or BF or Brainf**k) is a minimalist programmation language that uses only eight commands to manipulate memory and perform operations.
It takes its name from two words brain and fuck, that refer to a kind of major frustration for your brain (or cerebral masturbation).”

# Initial foothold:
To decode this, I went to dcode[.]fr

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/4b8d4913-6109-46fe-9cd2-abe5cdb1923a)

Now I can SSH into the machine! <br>
Upon logging in I’m greeted with a message.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/c033bc69-8624-4025-ad00-727112cf6225)

If there’s a message for another user I wonder how many other users there are, time to find out.
```
$ cat /etc/passwd | grep "bash"
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/e3361a90-236e-481c-acb4-5c1d6c4b2537)
 
Okay, a secret hiding place, it can’t be in the directory from earlier.  <br>
I used find to see if I could locate a file with that name but that didn’t turn up anything. <br>
Next, I tried looking for a directory named s3cr3t instead of a file. Bingo!
```
$ find / -type d -iname "s3cr3t" 2>/dev/null
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/2807855d-33a4-4ad9-920e-75190a0b888c)
 
From there I found the note!

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/0c09e411-90cd-49a7-96e8-78f83e7656cc)

And the contents of the text file are:

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/77467f33-a2f2-4ba2-9646-7d7a6a092c95)

Now, as the user Gwendoline, I can grab the user flag.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/e718e809-7f11-4ba2-8ea6-913c511cd2ec)

To see what I can run as root, since I have the user’s password.
```
$ sudo -l
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/933a668b-edb1-4f56-bf91-3b1be521f776)
 
Hmm, I can run that as other users except root.

# Exploit:
I tried looking for other ways to get privilege escalation and finally had to resort to uploading a copy of linpeas to the target machine and fired that off.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/f18fc502-1e10-42d6-b4f9-ac2073de7832)
 
Visiting the link took me to a page that showed a quick command for how to exploit that version of sudo.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/c0c20a54-d1e1-48a9-9250-ac4ea0d87f9f)
 
# Privilege escalation:
Great, now that I know how to exploit sudo it’s time to escalate my privileges.

```
$ sudo -u#-1 /usr/bin/vi /home/gwendoline/user.txt
```
Then, when vi opens the file, the command is:
``` 
:!/bin/bash
```
With the root shell it’s now time to grab that root flag!

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/d948ea7e-c15e-42e2-96f4-3f6029dd8bbc)
 
# Reporting:
•	Make sure to remove files from the web server that have sensitive information or are not in use anymore. <br>
•	Ensure correct permissions are set for users/groups and access to files/binaries on system are checked regularly. <br>
•	Keep all software, hardware and systems up to date with security patches/version updates to prevent vulnerabilities on older versions.
