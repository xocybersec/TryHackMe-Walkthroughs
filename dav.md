TryHackMe <br>
Dav
---

# Scanning

Nmap:
```
sudo -A -O -sC -sV <machine_IP>
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/8f4f4c49-ed86-4cc8-bac6-2a4c462f8ae5)

I scanned for directories after checking the webpage since it was the default Apache server page. <br>
Gobuster:
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/21a4dfe3-270d-49e4-a3e5-5b8de608cabd)

When visiting that page, a login page popped up. Naturally, I tried all the basic credential combos but no luck.. <br>
# Vulnerability Assessment/Exploitation
I looked up the default credentials and found two sets. One was `jigsaw:jigsaw` which didn’t work. <br>
The other that I found was `wampp:xampp` and that worked! <br>
I was using a tool called `cadaver` when logging in.
```
cadaver http://<machine_IP>/webdav
```

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/4b114e1f-6c56-43df-8533-73a8766412bd)

Found an interesting file that looks to have a hash, so I’ll try to crack that with `John`. <br>
After trying numerous ways to crack this without luck I decided to try another method to get in. <br>
I uploaded a reverse shell to the `/webdav` directory using cadaver and was able to get in that way instead.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/373de866-6ebe-44df-8b48-77df10a52ef8)

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/c02a74c7-c315-4e39-9fa3-7afb1a98438c)

I checked the home directory to find the user flag and grabbed that from the user **Merlin**. <br>
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/963566f7-5302-4bde-aaa1-e998667edbea)

Before trying to escalate privileges, I wanted to see if I could run anything as root/see what kind of cronjobs there were. <br>
No luck on the cronjobs, *however*, what I could run as root was more than enough!

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/50ddadfd-d8e6-46c9-b8e1-d5ab25717e84)

Time to grab that root flag and pwn the box! <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/9af973fe-d027-4930-96e8-5a17ed44ece5)

# Reporting
It is strongly encouraged to change/remove the default credentials for any type of software/app where a login field is present. <br>
Also, sanitization needs to be utilized so that if an attacker were to gain a foothold, things that were uploaded or entered were checked first.















