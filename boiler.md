TryHackMe <br>
Boiler
---
<br>

# Reconnaissance/Scanning
Nmap shows a few interesting ports open for me to probe around.
```
$ sudo nmap -A -O -sC -sV -p- <machine_IP>
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/6ef3fb82-db7f-41b8-b258-7d3d1376f1fc)

With the FTP server allowing anonymous login, I accessed the sever and found a text file named `.info.txt`. <br>
Here are the contents, which looks like a type of cipher waiting to be decoded.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/4ba79606-46dd-45bc-a200-ccbbfc292f0e)

Thanks to dcode[.]fr/caesar-cipher I was able to decode it but it wasn’t anything of interest.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/8d01592f-3281-4b99-a494-dfff6664a207)

I also ran a directory scan with Gobuster. Here are the results: 

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/26475cab-556b-4a67-acd6-f17a77d87de2)

# Vulnerability Assessment

First things first, I checked */robots.txt* and got a hit. Yes, it was all a rabbit hole. 

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/0c4962f1-5e7f-4903-8433-7b31f9ea4d6b)

My next approach was to keep enumerating on the */Joomla* directory. 

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/12ff14e6-c476-4c23-8377-19a713475a62)

Out of those, */administrstor* and */_test* weren’t dead ends. I didn’t have admin credentials to log in, so I ran a test with sqlmap in hopes I could find some type of entry point but I didn’t.
*/_test* had an upload form which looked to be vulnerable but I couldn’t upload the right file type.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/d410a275-0e68-4502-942c-117d3d3d3389)

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/763b3c0d-4355-4c78-b8a9-824f97470147)

# Exploitation

When clicking on “New” the URL changed to the above. That looked like a potential way to gain a foothold. <br>
I captured a request with `Burp` and tried to run a request through `sqlmap` again but still no luck. <br>
Next, I checked to see if there were any vulnerabilities via searchsploit. <br>
```
$ searchsploit sar2
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/20ce1f13-56b2-4765-b459-5e2b25e3cf50)

The text file confirms what I was thinking.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/f3840f23-25d9-4bcf-bd73-4fa6b4f71345)

I executed the python script afterwards and was prompted with the ability to enter commands. <br>
Successfully have RCE on that server! (Remote Code Execution)

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/9590bc09-bbd9-40d3-a400-0e0736fd7a2e)

That’s interesting, a text file named *log.txt* in the */_test* directory. Upon viewing, there are plaintext credentials!

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/6dc82d6d-3e62-4275-8d6b-3bfa47d37c34)

Time to get on that system.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/82b9dac6-2958-41ee-83ad-e1f22725e4ad)

In this user’s home directory, there’s a script so I concatenated that and I found some more credentials!

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/8c90bedb-c0e5-47fa-b61b-41e8eb37cf66)

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/f786e4ee-2cac-4270-9aa6-4b4e94a37dd9)

Escalating privileges to user <b>stoner</b>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/74e2b383-b1a4-42c8-8206-9ac028fbf197)

Viewing the contents for the *.secret* file will give the first flag.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/bdb4ad07-8550-4f84-9c8e-ac100c8b8ae3)

To see if there was anything I can run as root I ran the command:
```
$ sudo -l
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/b8f03f26-61c3-44f0-a3f7-a767df0ad5d0)

Well, since that was a dead end, I wanted to see if there were any world files with the SUID bit. <br>
Those files allows the user to run the file with a higher privilege level.
```
$ find / -perm -u=s -type f 2>/dev/null
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/08c10c51-ff7b-474c-81f9-6ea3283ed691)

Upon inspection of what has the SUID bit, I noticed that the binary `find` is listed. <br>
Since that's the case, when using find, you can execute a command using the `-exec` flag. <br>
Using that method I was able to read the *root.txt* file.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/3b3607ea-85dc-4490-b498-eb632f6a4aec)

# Reporting

It is <b>strongly</b> recommended to not store passwords, or any credentials, in plaintext. <br>
Using a secure password manager would be a good remediation. <br>
Also, be vigilant to keep all software/firmware up to date with patches. <br>
Be mindful of which files are kept in "testing" areas and how accessible to the public they are.


