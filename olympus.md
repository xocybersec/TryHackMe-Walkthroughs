TryHackMe <br>
Olympus
---

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/43a3e7fc-2806-4ef3-a633-610afb4c46e5)

# Reconnaissance/Scanning:
Let’s start things off with a network scan to see which ports are open and the services running on each.
```
$ nmap -A -O -sC -sV -p- <machine_IP>
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/1fc96369-631d-4f55-9f90-162623100991)

Using gobuster to scan for directories:

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/4bbd079e-9025-4fb0-9985-f6ace598131e)

After adding olympus.thm to the hosts file I ran another scan and got another hit.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/eac46f6b-932e-4695-9350-ab4936eb6b82)

After scanning those two for directories, /static only returned /images, but /~webmaster returned quite a bit more.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/c5fe4611-7fc7-440d-b4fe-33b4fa6cc8e6)

I poked around for a bit and tried to fuzz a few parameters to see if there was a LFI vulnerability, tried some basic credentials on the login area, and some XSS but didn’t get very far.

That means it’s time for me to fire up Burpsuite, capture a login request and let sqlmap try some SQL injection.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/059d7b67-1003-46bb-935d-35fd597a3a88)

To save a request in Burpsuite, after capturing the request, click on the 3 bars in the request window and in the drop-down menu click on Save Item. <br>
After that you can use that file in sqlmap to test for SQL injection.
```
$ sqlmap -r <filename> --dump –batch
```
Awesome, got some usernames, hashed passwords, and even a subdomain!

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/3325d43e-9248-4173-8cd8-5ce388a2a47d)

Even the first flag is retrieved!

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/a56067a6-cb7d-48b7-a460-337582f560c6)

Scanning the subdomain with gobuster:

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/5ad35f17-753a-4065-9744-c711e283f795)

I was able to crack Prometheus’s password with John.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/40c3aa93-410d-414b-80ed-6766528b25df)

When logging in to the chat.olympus.thm site, there’s a chat message displayed.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/5aa11822-e742-40ae-a2ce-3d34d2f67c8b)

With the upload button on the bottom I uploaded a reverse shell but couldn’t find it anywhere, even with another directory scan.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/45de6b68-ee9e-4a7a-8d38-a87716498f36)

I got stuck at this point trying to find out how the uploaded files were changed/where to access them so I tried to enumerate some more for missing clues.

# Initial foothold:
I checked out more of the tables from the sqlmap enumeration and found one of interest.
```
$ sqlmap -r r.txt - batch - dump -T chats -D Olympus -C file
```
That table showed the locations of the uploaded files!

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/b46791e3-be3e-4d8a-9750-265ce6ce57c1)

I checked on the uploaded password text file, however, there were no passwords.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/7371995a-2968-4958-8ef0-e92537b231e3)

Maybe the reverse shell will work..

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/22de4099-52f9-4b1f-bc03-e8f343df9e58)

Success!

I wanted to see who else could run a shell on the machine.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/41c01173-6b80-4b73-a4ac-cc9921448849)

Navigating to Zeus’s home directory I can get the second flag.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/abfd3955-36d3-4f0e-977c-4f2102c48ab3)

There was also a text file in that directory.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/8f8b8d78-172d-4bf8-bcc1-f10a30b12941)

# Lateral pivoting:
I inspected for any low hanging fruits, cronjobs, files/executables with SUID bits and I found one that ran with privileges of user zeus!

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/0ed18d5d-edd2-4c4a-bcfb-4f329a667a4d)

After a quick google search on how to use/exploit cputils I found a quick tip.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/d05cf0c6-6bbc-4e2c-903f-a73971a9e429)

Cool, I can run cputils as zeus and copy his private key to be able to SSH into the machine, whereas I couldn’t access the /.ssh folder at all as www-data.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/d8751cdf-c6fd-4e5c-bc5b-804fad0add75)

Now I can cat that file and save it to my local machine and see if it’s passphrase protected or not.
```
# converting the private to a hash format
$ ssh2john id_rsa > hash.txt

# cracking the hash with John
$ john hash.txt -w=/usr/share/wordlists/rockyou.txt
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/6400e260-1f06-4884-9a8f-53423c1d5eb1)

Got access!

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/d4db0cec-fba2-4263-9bc5-573d6cb9716e)

Before escalating to zeus, in my enumeration of the machine stage, I found a directory in /var/ww/html that I couldn’t access that was named a bunch of gibberish.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/82b612db-132a-478c-8bd3-e02e4de29847)

Now I can get in and poke around there. I found an executable so I wanted to see what it did first.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/18282b67-1270-432a-b2ac-6aa6d26a31b6)

I immediately found a password and noticed a variable named $shell.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/ddb81dd4-ab74-49d4-b605-c4751691c7fb)

I cracked the hashed password, but it didn’t work when trying to escalate to root or use as zeus’s password.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/563d83d1-b11a-43df-9a93-3e0b418289e1)

# Privilege escalation:
Next was to enter the commands from the $shell variable.

NOTE: notice the last command is calling the variable $suid_bd so it’s necessary to enter the path instead of the variable.
```
$ uname -a; w; /lib/defended/libc.so.99
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/3626f3a6-e08e-45ea-a60e-56042ad435c2)

Time for the root flag!

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/9f5953d2-7fe1-4d99-8b55-f27818ac284d)
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/a07b2863-7352-4a50-84c8-cb8cffc868e1)


After going to google to find something from that hint, I found:
```
$ grep -irl flag{ /etc/
```

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/d24f0b33-9985-4d70-b232-09fb673b5e5d)

Got the bonus flag and the grep syntax is in there.
