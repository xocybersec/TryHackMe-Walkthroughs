TryHackMe <br>
Expose
---

# Reconnaissance/Scanning:

I started off by scanning the network to see which ports were open/services running on the ports.
```
$ nmap -A -O -sC -sV -p- <machine_IP>
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/c0dbd7b4-6121-4550-9a92-f5d479fe6330)

Next, I ran <b>gobuster</b> on the webpage on port 1337.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/a25895d4-ba96-4819-83d0-f632ec96c903)

Then I ran <b>gobuster</b> on the <b>/admin</b> directory.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/e304af0c-3ee3-4a17-86bb-a48db54850c9)

And on the <b>/admin_101</b> directory.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/23bfb0c3-20a0-4ef7-9daf-2f3966814cc8)

# Vulnerability assessment:

I found a potential thing to note in the <i>script.js</i> script located in the <b>/assets</b> directory. That must be used with the <i>chat.php</i> page.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/4f687052-39a4-4d92-bfcd-2bbccffdfa1b)

There was also a login field on the <b>/phpMyAdmin</b> directory. I tried logging in with basic credentials and got an error message.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/b7872666-ee73-4a43-9008-110162498c03)

I used <b>sqlmap</b> and <b>burpsuite</b> to capture a login request to see if I could get any information but had no luck with that page. <br>
I visited <b>/admin</b> to see:

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/6349045a-1d85-46bb-be8c-ed9df27ea70b)

That was the same on <b>/admin_101</b> <br>
However, when trying to login to both pages, <b>/admin_101</b> returned an alert error. <br>

# Exploit:
Since I got an error, I ran another captured request text file from <b>burpsuite</b> in <b>sqlmap</b> and got a few hits!

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/528c42c2-5a87-491d-8844-01f0adea6477)

Now I know there’s a user that starts with the letter “Z”

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/979df328-dc11-4883-8db1-0266347bccda)

Visiting <i>/file1010111/index.php</i>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/858c51b9-b2b9-493b-be12-fa94ab7ec702)

Entering the cracked password then shows this:

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/8b80a029-e21c-4411-b994-e2f199d5566c)

Time to do some parameter fuzzing then. <br>
I used <b>gobuster</b> once more to see if I could locate the <i>/etc/passwd</i> file from that page.
```
# using the FUZZ parameter to pass different words to search for local file inclusion
$ gobuster fuzz -u http://<machine_IP>:1337/file1010111/index.php?FUZZ=/etc/passwd -w /path/to/wordlist
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/411c86b6-db5d-4c4d-9afd-345e13c935fc)

Now I have the username that starts with “Z” with a local file inclusion vulnerability!

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/236f676d-3874-4d60-8fbc-bae101292f38)

Visiting <i>/upload-cv00101011/index.php</i>:

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/65c65c8a-9f90-4869-9497-79c956d9ede7)

Viewing the source code on that page:

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/ed190f79-41ba-4802-8971-aca28b7a6c7e)

After uploading a php reverse shell I got this message: <br>
<b><i>NOTE: I did have to change the extension to .png to upload it.</i></b> <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/c5ad5102-2d7d-4469-a945-84bb2af03678)

I didn’t see anything in the source code, so I uploaded another image and captured the request with <b>burpsuite</b> and found the path the uploads are sent to. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/443f8796-c2ed-4c6c-a941-4f6eb440b4c9)

Now to access the shell, I had to visit 
```
hxxp[:]//<machine_IP>:1337/file1010111/index.php?file=../upload-cv00101011/upload_thm_1001/shell.php.png
```
NOTE: Since the <i>/file1010111/index.php</i> had the LFI vulnerability that’s the injection point to access the reverse shell file. <br>

I used the python one liner to upgrade my shell. <br>
```
$ python3 -c 'import pty;pty.spawn("/bin/bash")'
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/fc796cbf-e2a6-4bd5-844b-0cfe3de8212a)

I checked the home directory for other users aside from the one I already knew of. Just “Z”. <br>
Inside the user Z’s home directory were two files. 

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/3631691b-2808-407d-b8ca-8a6e57c8a239)

I couldn’t read the flag file but even better, for me, I could view the <i>ssh_creds</i> text file.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/ad19adda-a556-4416-aebb-e775b6e937c4)

# Privilege Escalation:
I got an SSH session going and I got the user flag!

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/1a1b7447-9821-40e0-8166-5f0cc381c9b8)


Checking for any low hanging fruits such as cronjobs, what I could run as root or anything in writable directories was a dead end. <br>
However, when checking for binaries with the SUID bit set I had some luck. <br>
```
$ find / -perm -u=s -type f 2>/dev/null
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/ee5ce440-4595-4a24-b4de-7756cfe28f42)

There are a few ways to get the root flag by either privilege escalation <b><i>OR</b></i> just using the SUID binaries as the current user! <br>
Escalating to root via <b>find</b> command: <br>
```
$ /usr/bin/find . -exec /bin/sh -p \; -quit
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/7dffc516-648d-447f-b8f7-a9607c30379a)

As current user: <br>
Using <b>nano</b> to read the root <i>flag.txt</i> file. <br>
```
$ nano /root/flag.txt
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/f133c865-d78e-478b-bde2-a8bef667748c)

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/5c8ecc47-d47a-428b-b02b-777f0fec3e14)

You could also use <b>find</b> to execute a command. <br>

```
# using find to concatenate the file flag.txt in the /root directory
$ /usr/bin/find /root -exec cat "/root/flag.txt" \;
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/1c22e872-cf88-46b9-8456-3ff07a58d63e)

# Reporting:
It is recommened to add more data sanitization for the SQL database to prevent the LFI vulnerability. <br>
Also add sanitization checks to make sure correct files are uploaded and also contents are safe. <br>
Add a salt/pepper to hashes and use a strong crytographic hash function such as SHA256. <br>
Turn off anonymous login for the FTP server. <br>
Check all permissions on files/binaries to be set correctly.






