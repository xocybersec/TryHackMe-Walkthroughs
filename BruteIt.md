TryHackMe <br>
Brute It
---
<br>
Running an nmap scan on the server we get the services/versions running on the open ports. <br>

```
nmap -A -O <target_IP>
```
  
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/50dc931d-b1ec-497c-add4-75cf565301ac)

Next, running a gobuster scan will let me know which directories are there. <br>
```
gobuster dir -u http://<target_IP> -w /usr/share/wordlists/dirb/common.txt
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/9416ece0-0730-4707-b03d-38c5d1400d2e)

I decided to run another scan on the `admin` page because there’s usually more in the admin directory. <br>

 ![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/1d1e7a73-9028-4f66-a922-98d99e8bf81d)

After getting that result, I had a feeling it went further than `panel`. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/a7ea0bf6-56e0-489b-9969-b54a991de1a9)

*/id_rsa* is a really good find but we don’t know who’s key that is exactly. <br>
Make no mistake we’re still going to save that and try to crack that with `John`. <br> <br>

Well, the *admin* page was a login page, as suspected but I couldn’t bypass it with basic credentials, so I checked the source code. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/6950f0a7-43c7-449c-8bb1-1355daa1c8b1)

There’s one username for sure and a potential second one. <br>
With that information, time to use `hydra` to try to brute force that login page. <br>
```
hydra -l admin -P /usr/share/wordlists/rockyou.txt -f <target_IP> http-post-form "/admin/index.php:user=^USER^&pass=^PASS^:F=Username or password invalid" -v
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/013fffd3-9201-484f-955a-a51aa0715bb4)

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/dfacd6c0-4aad-4b11-98b1-14f19f16baf3)

After logging in, there’s a flag to get and the name for who the RSA key belongs to. <br>
Let’s get to cracking that RSA key. <br>
```
sudo ssh2john id_rsa > hash.txt
# John hashes the key

sudo john hash.txt -w=/usr/share/wordlists/rockyou.txt
# John attempts to crack the hash using the supplied wordlist
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/da4e771f-87be-4686-a2c4-53583b309838)

After getting a shell as John, I viewed the text file in his home directory for the user flag. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/22aa5798-29d2-49d9-a2cb-b1a97e609b94)

I also checked the cronjobs going but there weren’t any scheduled. <br> 
Another thing to check is `sudo -l` to see what the user can run as root. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/9fd3482c-ff48-4198-919f-cb1758e9f776)

This user can read files that usually require root permissions. <br>
If you’re thinking what I’m thinking, that means we can check the `/etc/shadow` file if the `/etc/passwd` file shows an `x` after the username. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/dc42427c-7ba0-43a9-b8ee-dc0446b811f7)

This shows that we need root permissions and the file itself. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/3341554c-aeec-4364-ae07-635fef477570)

After that the next step is to save the root content from the /etc/shadow file to my own text file and give that to John to crack. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/b7a212a9-a9fd-4642-a340-b6d768881376)

Great success! Time for root access. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/ddd701a1-5bd0-4518-afae-9af76364260f)

Pwned this box!
