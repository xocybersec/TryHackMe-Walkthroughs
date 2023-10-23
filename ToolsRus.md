TryHackMe <br>
ToolsRus
---

Nmap helps show which ports are hosting web services, the types of servers and their versions. <br>
The answers to Q5, Q6, Q8 and Q9 are shown also. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/0847dea6-95e4-48d5-b17e-9d860153a9ca)

To see which directories there are on the server on Port 80 I ran:
```
gobuster dir -u http://<target_IP> -k -w /usr/share/wordlists/dirb/big.txt
```
There are directories that I don’t have access to, however, <i>/guidelines</i> is accessible and the answer to Q1. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/60c5addb-5d6b-4396-85dd-d7e734dba5b0)

When visiting <i>http://<target_IP>/guidelines</i> the name needed for Q2 is shown. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/cd04f5fa-6b14-4345-8696-73816abbb703)

I wonder what could be behind the <i>/protected</i> directory. (Q3 answer)<br>
I was prompted with a box to enter credentials when I visited the page. Since I didn’t have a password for bob, I tried to bruteforce the login. <br>
```
hydra -l bob -P /usr/share/wordlists/rockyou.txt 10.10.99.0 http-get /protected
```
And the password was cracked fairly quick. Q4 asnwer. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/76f5f1e9-429f-4258-b29f-57f1fe56a980)

After logging in, this page is shown. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/32301dc3-828a-48ea-aca1-ccf0f554c459)

Now time to check out the server on Port 1234.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/7dfec0fe-0299-4bed-b10e-25250d9ba47f)

I logged in to the manager app and proceeded to scan that with `Nikto`
```
nikto -h http://<target_IP>:1234/manager/html -id <username>:<password>
``` 
The results show that there are 5 documentation files that were found. Q7 asnwer.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/2b39610c-3de3-48fa-b8cd-77c63daf6182)

Next, I fired up `Metasploit` and searched <i>Tomcat</i> for an exploit and this was the one I used: <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/1979c577-4ecf-4ec9-b373-98e5903d5c44)

Once all the options were set up, I ran the exploit, got the root shell and the flag! <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/219acfcf-5d94-45f0-a99f-0cb945eaa131)



