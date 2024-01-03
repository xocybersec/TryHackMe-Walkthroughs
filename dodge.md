TryHackMe <br>
Dodge
---

# <b>Reconnaissance/Scanning:</b>

Running an Nmap scan showed me the open ports and other hosts on the server.
```
$ sudo nmap -A -O -sC -sV <machine_IP>
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/2ec0023f-929a-4013-b245-21ad6994de04)

I had to add the hosts to my /etc/hosts file before I could even try to connect to them. <br>
After that the only two hosts I could access were *dev.dodge.thm* and *netops-dev.dodge.thm* <br>
*dev.dodge.thm* was a PHP splash page that didn’t show much except a possible username. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/ceaff661-155f-46e6-aaeb-e9951ae256b8)

*netops-dev.dodge.thm* looked to be a blank page so I viewed the source code to find:

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/d5c112ed-3b25-405a-b04a-11024a931062)

The first script <b>cf.js</b> was a dud but here’s what <b>firewall.js</b> had:

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/a364dc67-08e0-43df-8ca9-7e5eab26239a)

After plugging that in at the end of the *netops-dev.dodge.thm* URL it took me to a web interface for the firewall!

# <b>Vulnerability assessment:</b>

It showed that there was an FTP server up, but it was set to DENY. So, I went ahead and tried to allow traffic.
```
sudo ufw allow 21
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/8dcf9f09-f7e5-4e06-8623-9564e42a181a)

With that enabled I tried to log in as an anonymous user.
```
$ ftp anonymous@<machine_IP>
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/f11e619c-b088-40d5-83ad-fe84f92680ab)

I used mget to transfer all the files I could, however, I couldn’t anything with `user.txt`.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/0445e013-5c43-453d-a7a6-1956f51b7f1c)

Word of advice is to also check the .bash_history file for any information.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/3c295e85-a6e0-4a5d-9c91-da625d8b4f7e)

Those are some files to keep note of after gaining an initial foothold.

The `authorized_keys` file was a great find for me. I found a username to access a SSH session.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/2aeda4d8-04f5-4612-8af9-0185cb4d6800)

```
$ ssh -i id_rsa_backup challenger@<machine_IP>
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/c5d2ed03-0892-4e75-8718-22ee7afe1cbf)

In the home directory there were several other user’s directories to look into.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/3d7c7571-4b43-4895-821e-2fabd549c790)

In <b>ubuntu</b> there was a notes text file, here are the contents:

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/d331bd7f-1e2f-4a0c-9bbf-c53bba5de67b)

Those credentials ended up being a dead end unfortunately.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/a352069e-6dd1-487c-9223-4eb675f32743)

Remember those files from the .bash_history file? Time to find those for inspection.
```
$ find / -type f -iname "setup.php" 2>/dev/null
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/184cb25f-5708-4e28-ac65-4292c33dd374)

Look at all the potential files!

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/b4656ff5-f64e-4d7c-977c-b95ca83ad4ef)

*config.php* showed me that there’s a mysql database and also showed me the credentials.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/9231bff2-7e8f-4389-b82e-0bd857fca882)

Found some more credentials in plaintext in *login.php*

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/80dca98c-3fda-45b0-968c-249ad0e0b42d)

Now, in *posts.php* there was some base64 encoded text.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/c388c009-99d7-4633-b708-3f2c7f3d6c01)

I went ahead and copied that to my machine then decoded it.
```
# saving it to a file
$ echo “<base64 encoded text>” base.txt
# decoding the text file
$ base64 -d base.txt
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/ee3394bf-6365-4d89-9f4f-83c14835cff8)

# <b>Privilege escalation:</b>

Now I have user cobra’s SSH credentials! Time to escalate to that account.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/9862266e-ec17-4333-97ab-eacece937d45)

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/b465f43e-5a8c-421a-9c7c-2a001a4a8b26)

# <b>Exploit:</b>

That’s great news I can run something as root, now it’s time to exploit it! <br>
Visiting the trusty gtfo[.]bins and searching for the `apt` binary I found an exploit.
```
$ sudo apt update -o APT::Update::Pre-Invoke::=/bin/sh
```
After executing that command I had root access, machine pwned!

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/f5e4662c-bb54-4e3f-984f-c1fa32c5020e)

I made a stable shell using:
```
$  python3 -c 'import pty;pty.spawn("/bin/bash")'
```
And grabbed the final flag.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/197d6201-e130-49f1-aa63-92a4991ef151)

# <b>Reporting:</b>

It is strongly recommended to not store passwords, or any credentials, in plaintext. <br>
Using a secure password manager would be a good remediation. <br>
Make sure users have proper permissions assigned per their role. <br>
Web developers should check their code and make sure what doesn’t need to be visible isn’t. <br>
