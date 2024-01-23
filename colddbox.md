TryHackMe  <br>
ColddBox: Easy
---

# Reconnaissance/Scanning:
Let’s start things off with a network scan to see which ports are open and the services running on each.
```
$ nmap -A -O -sC -sV -p- <machine_IP>
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/35d31507-69b3-4de0-a18f-5b1969f5830a)

Scanning for directories with Gobuster:

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/f5be63dd-8bc4-4bf5-bf72-64fd7ece3fd4)

When viewing the main webpage i’m taken to a WordPress site.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/fbccc921-4297-4c80-aded-e85df055654a)

Clicking on the link C0ldd takes me to a twitter page with a username that might be useful later.
 
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/df221a43-fe5d-4f00-a0b2-41e73b0dcf23)

Viewing the /hidden directory:

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/750337d8-0bfa-4a26-9c2c-8d38cc5bc4ad)

Viewing the /wp-trackback.php directory:

 ![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/f0827e9a-e437-4c32-9157-442490fdc320)

Viewing the /xmlrpc.php directory:

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/fc49403a-00cb-4b1d-9fb2-c902e2cdc2d8)
 
# Vulnerability assessment:
Since Wordpress is used, I’ll use the tool wpscan to enumerate users.
```

$ wpscan --url <machine_IP> -e u
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/b8d53e5e-110f-4eca-a222-d6251fcfb06b)

Found usernames are confirmed the names from the /hidden directory. Now I’ll try to enumerate their passwords.
```
$ wpscan --url <machine_IP> -U <username_list> -P <password_list>
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/5ca97340-42f2-4d92-b9fa-ba39bb392637)
 
Awesome, I can log in to the site now! <br>

# Initial foothold:
I needed to find a way to add either a file or the code for a reverse shell.  <br>
I tried looking for a way to upload a profile image, added the script as a new page, add it to comments. <br>
The way I found that worked was going to the appearance section, then editor, and added the script code to those already made pages on the right hand side.
 
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/f4bfe66b-f957-4a8f-a43f-57c2c0776e1b)

I added the code from pentest monkey’s reverse php shell script to the index.php and refreshed the main page and got a reverse shell on my netcat listener!

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/e35296a8-f1be-4cff-9ca5-d70e73f93310)
 
Checking for all the users only returned two that have shell access.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/56dc57b8-5276-4783-990d-ecfc8bae26d1)

# Exploit:
My next step was to see if there were any files/executables I could run as the current user that have the SUID bits set. I’ll use the find command for that.
```
$ find / -perm -u=s -type f 2>/dev/null
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/ebbea48f-f827-4635-a873-b9c207c73727)
 
Since I can run find with higher privileges, I’ll use it to execute the cat command on the user and root flags. <br>
Using find to read the user flag.
```
$ find /home/c0ldd -exec cat "user.txt" \;

```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/f8fc3512-efcc-4f50-9f68-f9accb4b7cda)

Since I could tell it was base64 encoded, I copied it to a file then decoded it on my local machine.
```
$ base64 -d <filename>
```
The decoded message reads:
 
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/a69dabad-f1d1-446d-8767-a4fed8c2164f)

Now for the root flag!
```
$ find /root -exec cat "/root/root.txt" \;
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/d11a1153-75a6-4a32-bd80-3411ca5ffa65)
 
The decoded message reads:

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/a9d0aa42-a4cd-4148-919a-791f4ba9e507)
 
# Reporting:
•	Be sure to use strong passwords/passphrases or even a password manager to help generate and store the password. <br>
•	Make sure files containing sensitive information are accessible by the proper people only and password protected with a strong password. <br>
•	Ensure correct user/group permissions are assigned on all files, especially executable files.
 
