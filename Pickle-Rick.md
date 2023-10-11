TryHackMe <br>
Pickle Rick
---

I had to start with some enumeration to see what I was working with and here are the `Nmap` scan results: <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/6e0f6b24-91cb-41ab-9df0-60ec3bcbcd7d)

I also used `Gobuster` to find directories. Here are those results: <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/207655ac-fa8d-4835-a8d5-969563e9fa67)

On the main web page, I looked at the source code and found a comment towards bottom. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/da64de79-3f48-4f75-bdd7-d439b75ecba2)

The `robots.txt` showed this: <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/90cbd43a-4ecc-4bce-a8e2-bc1d5aac66c9)

I potentially gained some credentials. <br>
I then tried using them to SSH but was unsuccessful. They'll stay in my notes for later, just in case. <br>
Next, I proceded to run a `Nikto` scan and found a login page. Looks promising! <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/9719a85a-ced9-4686-ac50-18abc21a8e92)

The credentials from earlier were able to get me in and I was taken to a <i>Command Panel</i> page. <br>
I looked at the source code of it as well and found another comment of interest that was base64 encoded. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/d73802d9-a962-4a98-be62-466587a37240)

Using CyberChef to decode the base64 string the result is: (I chuckled) <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/0553488f-8ea0-486f-b723-bac08a4b397d)

Back to the drawing board it is. <br>
At the Command Panel page, I first ran the command: <br>
```
sudo -l
```
And found I could run any command without needing a password. Bad security practice but great for me! <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/7c71a145-fa1f-4b06-b175-8fa502e1a574)

Next, I used the command to show what directory I was in and what files were there. <br>
```
pwd; ls -la
```

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/e4c37574-3662-471b-9ca4-9d4973a4c2af)

I tried to read the files on the Command Panel webpage, but the commands used were all filtered, and I couldn’t open them. <br>
After that I thought of two ways to get the files open. <br>
I edited the source code to open a file when clicking on a tab instead of the <i>denied.php</i> file. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/81d4e391-1f64-47cc-8a26-480dd1c8b428)
<br>
The second way is easier, I used the following command from my terminal: <br>
 ```
wget http://target_machine_IP/Sup3rS3cretPickl3Ingred.txt
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/0e5ae2be-a529-4f92-9dbf-b334556c9921)
<br>
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/b12a60b3-5e9c-4ad2-b5a7-2eab65e255eb)

With that clue, I tried to break out of the directory I was in [vaw/www/html] and explore. <br>
This was the command I used to do that. <br>
<i>NOTE: the semi colon is to separate commands on a single line</i> <br>
```
cd ../../../; ls -la
```
Two directories immediately stood out to me. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/fd0145a1-45df-4edb-8439-1e91ab16b8ed)

I explored `/home` first. <br>
```
cd /home; ls -la
```

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/9a5bc3a1-8b33-40e6-9167-2c6bebad96a1)

After seeing “<i>rick</i>" I had to poke around in there. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/2fc8eef6-0dcf-4960-8c60-848ac0501790)

In there the second ingredient was waiting to be read. <br>

```
less /home/rick/"second ingredients"
```

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/64b6bc5b-71d8-48ca-9282-b7431d3c528a)

Now for the last flag, it was time to look at the root directory as the first stop before looking elsewhere. <br>
I used `sudo` that way I could get access. <br>
```
sudo ls /root
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/bc159a28-4cfc-4458-b612-816c6a225a50)

Looks like the last  flag! <br>
```
sudo less /root/3rd.txt
```

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/5fddc902-3b5b-4033-a273-606e770a3f1c)
