TryHackMe <br>
Jack-of-All-Trades
---

Nmap showed an interesting scan. The SSH port is swapped with the http server port. <br>
To make the page viewable I had to go into the `about:config` of Firefox and change the proxy to manual with the desired port for the webpage in the Network Settings. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/87f944dd-5e2a-4e02-8ca9-cd2571fbb10e)

Checking the source code of this page shows another page to visit and a base64 encoded string. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/4a8796bd-91f4-4088-9c37-690889f11b85)

I decoded the string and got a message with a password. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/63076f9e-f132-4ed7-94fa-cb1228f10248)

Visiting the */recovery.php* page we get a login form, however, the password I decoded didn’t work. <br>
I checked the source code to find another encoded string, this time with base32. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/2d553b9b-ebf9-4945-94ca-be3176dae873)

I decoded that and ended up with a hex output. Decoded that and got another message that looked like cipher text. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/acd9d1dc-2414-4ba6-8c30-4b4dd322fcc2)

I used a cipher decoder and it ended up being a ROT13 cipher. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/dc890220-9269-4d78-b363-ee1192587c7c)

After no success logging in, I visited the homepage again and poked around then thought to see if there was anything hidden in the images. <br>
I downloaded the three that were available, ran `steghide` on each and got a text file from one. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/3eaf9b9b-d524-4847-82d1-be7fb285a233)
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/b8fee5d9-4978-447d-a88f-744231a3c9e3)

The next one that had something was <b>header.jpg</b>. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/1a4bfccb-f966-43ad-b693-86f38e59c603)

Now I can try these credentials and log in again. Success! <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/7b275c78-1658-47ce-86e9-7346048565a1)

I ran a basic command in the URL bar, and it worked. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/e00dc919-5474-4f35-a5c5-45819d1108bb)
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/848acbdf-e775-4367-910f-27cb8156d130)

After poking around for a while I found a password list in the home directory. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/fc07d470-db5a-4efc-88aa-c3c2e4483c29)

I copied that, and since it’s a password list, maybe that’ll work for brute forcing the SSH password for jack. <br>
I fired up hydra to give it a shot and found the password fairly quickly. <br>
```
$ hydra -l jack -P hash.txt <target_IP> -s 80 ssh
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/e7c19baa-87ce-40bd-b7fa-91377986b3b7)

I’m in! <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/634f5e32-861d-497b-a415-4679ab0f9e37)

There is an image in jack’s home directory which I downloaded with SCP (secure copy) <br>
Then when viewing it I got the user flag. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/e8ac6568-fd81-4060-98ea-9f9feb0595c5)

Next up is to get the root flag. `sudo -l` showed I couldn’t run any commands as root; I checked a few more directories for anything but had no luck there either. <br>
Another trick I tried was to see which files with a SUID could be ran as root. <br>
```
$ find / -perm -u=s -type f 2>/dev/null
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/730e8a75-4f7e-40ef-bf33-3a1551790434)

If I can execute `strings` as root then I should be able to view the root flag.. <br>
```
$ strings /root/root.txt
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/bab50205-9331-4b81-ae2c-cacc5bebbf00)

Pwned!













