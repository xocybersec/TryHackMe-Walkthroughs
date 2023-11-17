TryHackMe <br>
b3dr0ck
---

I started off with an nmap scan as usual to see the ports/ services running. <br>
 
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/b0b17731-366c-4024-8276-e1f345ca6f77)
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/72c9919c-9dde-465c-b17d-b16172703e47)

Following the link highlighted led me to this page: <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/b5f2ba2c-f2da-45ce-9af3-109a1c40c7a6)

Using `socat` to view what was on port 9009
```
socat TCP:<target_IP>:9009 –
```
I tried to see if a flag was available, which there wasn’t, but instead I received a helpful prompt for what I could retrieve. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/493d3cb2-6747-47ad-ae69-d61d25191201)

Once the key and certificate were output, I saved each into their own text file. <br>
I tried some other commands and the only one that worked was `help`. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/02a6e28e-18bd-40e5-ac05-4ec000d0116c)

After using that command and logging in I tried a few more commands and was given an output for what the service would do. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/3ec78638-e0ad-4095-8bd0-85efd0d7b828)

I got a password hint, that made me wonder if that was either a hash or maybe something to decode. <br>
Next, I tried the login command and was told to use SSH. <br>
Okay, I checked on crackstation[.]net to see if it was a hash and it wasn’t.  <br>
Then, checked CyberChef to see if it was encoded, it wasn’t. After that I decided to see if that was the password and not a hint. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/6cd07010-5bcb-4b2d-9a08-891b4eafd63a)

Awesome! <br>
I checked the cronjobs but nothing was there. Next was to see what I could run as root. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/a39db9f2-208e-443e-abb7-10b27dce4869)

Time for the first flag as well. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/f937d923-83ac-4651-9286-75df40bfe8f3)

Since I can run `certutil` as root I’ll make a ney key/certificate pair for Fred and save them to my machine. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/af91e01f-d3d1-4e4e-876b-ca8df3839140)

```
sudo certutil fred "Fred Flintstone"
```
After that I’ll use `socat` again to log in to port 9009 and get Fred’s SSH password. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/f462625a-9a0c-45a3-920b-59ac8a3ebc50)

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/c8e9bbd8-638e-4852-8dc1-e5b412c48f77)

What can Fred run as root? <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/8f14ae89-c908-4188-9bc7-1317e7e3b897)

Thanks to the hints from TryHackMe about it being multi encoded and needing to decode then to check crackstation with the result I got the root password. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/b9e2f3ee-4943-4311-aa01-da616d4074a5)

Thanks to crackstation it was able to crack that hash. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/eca290e3-e48e-46f7-9294-062ce8fc6654)

Time to `su root` and get the flag. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/cd17f146-5f4b-4cab-a048-3abc22bd6375)

Pwned this box! <br>







