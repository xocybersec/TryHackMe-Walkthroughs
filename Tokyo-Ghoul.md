TryHackMe <br>
Tokyo Ghoul
---

Task 2
---

There are 3 open ports from the `nmap` scan and the OS being used is Ubuntu. <br>
The command I used for nmap:
```
Nmap -A -O <target_machine_ip>
```

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/d981f525-38c9-4333-87c0-a34db9842133)

Task 3
---

When looking at the web page there is a link saying “<i>Can you help his escape?</i>” <br>
That link brings you to another page called “/jasonroom/html” <br>
While inspecting the source code there’s a note that is commented out.. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/c4b9b089-d2c8-4973-95dd-dc55678211b8)

From the nmap scan it is shown that the FTP server accepts anonymous login, so let’s do that. <br>
There was a total of 3 files and two directories to get them from.  <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/82c4b242-1771-4153-8c0e-050116fbc166)
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/9676e828-8f22-40ab-89e6-e40437d16576)
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/dd5f3f68-57d0-4e0b-9c2d-5959f687ab9a)

Upon using `cat` to view “Aogiri_tree.txt” there were a few keywords I made note of since the task questions mentioned a key but that was it. <br>
Next, I had to change the permissions to be able to execute “need_to_talk”.
```
chmod +x need_to_talk
```
I used the `strings` command to see what human readable words were in the code and tried a few of interest as the key. <br>
After some tries, one of them worked and gave us the passphrase for the jpg file. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/b3c11a68-ecf0-424d-8c7c-f958406497c3)

Task 4
---

After using `cat` to view what was in the jpg file it’s morse code and a note saying: <br>
“<i>if you can talk it allright you got my secret directory</i>” <br>
The next step was to decode with CyberChef, which gave us hex, then was base64 encoded, <br>
and after decoding that the secret directory “d1r3c70ry_center” was provided. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/b18809fb-5bd4-4d7c-b9b7-cef6b8acceeb)

On that web page there is repeating text saying to “<i>scan me</i>” <br>
I obliged and fired up `gobuster` and got a result: <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/7c2dee7b-9a1e-4cd7-9504-d22270a7224c)

After visiting that page there were three links to choose from. I decided to view the source code to see where they went. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/a27fc6b5-91cb-4013-9e6d-bfbecc250b9f)

Two options lead to the same place so I decided to try some directory traversal but was unsuccessful. <br>
The page that loaded said not to do that, so there must be a filter in place. <br>
The next step is to try to bypass that filter with some HTML URL encoding. <br>
Filter bypassed and `/etc/passwd` reached! <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/40249aa2-df42-4bd8-8f68-06839072d42e)

The `/bin/bash` shows that that user can access a shell, so we need to crack that hash and access that account. <br>
After saving the hash to a text file and running that through `John` we get the password easily. <br>
```
sudo john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/2670f1e6-9ac9-43e9-8d96-ebc0704b86e2)

Task 5
---

After SSHing in with the found credentials, I wanted to see just what I could run as root with this command: <br>
```
sudo -l
```
Which returned this: <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/3341747d-16d7-4d2b-b031-2c9873964f70)

Python3 and the python program can be ran. <br><br>

In the main directory there is a `user.txt` file that I viewed and got the following: <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/b938c18f-3494-4c19-91d8-ea19a5fac9e2)

Jail.py was in the same directory, but before running the python program, I wanted to see what was in it. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/770cb356-e52f-4170-9e76-ba7e8cb67d00)

A lot of checking going on that will need to be escaped, but how? <br>
I had to turn to my friend Google again to see how this worked. <br>
Luckily, there were some articles about “python jail” that talked about some work arounds. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/c250caf4-de88-41d7-8dff-28b59b33294a)

After getting root access, I had to navigate to `/root` and that was where the last flag was at. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/ff11560d-4e37-4210-8ea8-2ce5afd8d61c)


