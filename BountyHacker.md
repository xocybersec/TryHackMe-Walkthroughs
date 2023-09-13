TryHackMe <br>
Bounty Hacker <br>
---

Let's get started by running an `nmap` scan to see which ports are open and get a quick lay of the land. <br><br>
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/815b0752-71f7-402a-b221-a602e82437c5)

Got some good info knowing that `FTP` supports the anonymous login, the server name & version. <br>
After logging in to FTP we use the `dir` command to see what files are there. <br><br>
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/0e3143a5-6634-4d11-9a12-37f0223b7d56)

The `locks.txt` file looks like it could be a plain text password file, which is straight gold. <br>
At the end of the `task.txt` file it’s signed with a name.. <br>
I decided to try to brute force the SSH login using `hydra` with the name I found and the locks.txt file as my password file. <br><br>
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/2ff567b9-01fd-45bd-9702-a681098d0d7e)

With a successful match it’s time to log in! <br>
We start off in the user’s Desktop and after running the `ls` command we’re shown a text file with the first flag. <br><br>
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/d7914d22-c28b-4ff9-86ac-517bee3ed167)

With that done, I used the `id` command to see which group we were in. <br> 
Next, I ran the command `sudo -l` to see which permissions the user had as root, which were `/usr/tar` <br>
I also checked a few places like `/etc/passwd` to see what other users there were but only saw root. <br>
Also looked the `/tmp` and `/opt` directories just to see. Not a whole lot in them unfortunately.. <br>
Well, after that I wanted to see which processes were running but I didn’t see anything of interest there. <br>
Next, I used my friend Google to see if it had any pointers for me and the directory /usr/tar. <br>
I was brought to `gtfobins[.]github[.]io` and searched there for anything I could use.. <br>
And what do you know, there was a snippet of code for escaping the current user and making a shell as root! <br>
```
Sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```
After getting the shell, I ran the `sudo -l` command again to make sure I was root.  <br><br>
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/051e560f-dac6-4708-a4b7-9ebf78716f4e)

Since we’re verified root user let’s find that flag! <br><br>
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/2cd0ead1-7b66-446e-b104-b43d964a813d)
