TryHackMe <br>
Valley
---

Running an nmap scan shows two ports open. SSH and a webpage. <br>
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/f70e0084-d3d4-4e17-b057-7eeb40dd8e1b)

Gobuster scanned for the directories on the web page. <br>
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/607f4792-24c4-4f44-88df-0d56156bc838)

*/pricing* had something of interest. <br>
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/473c8121-df59-41de-9bfe-afb6a0a73736)

Here’s what the text file said. <br>
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/753da32a-2f42-46a5-8d08-2a68814be590)

Looks like we have some initials of names to be on the lookout for and some more potential notes. <br>

The */static* page also stood out to me, so I ran another scan on that page. <br>
Got a lot of hits, time to check them out. Hopefully it’s not a rabbit hole.. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/040ecaba-dc1a-4206-a179-233eab8f0a85)

After checking all the pages, there were two with a breadcrumb for me. The rest were all pictures and there were more pages than the scan showed. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/0097d47d-536f-4a6d-8b47-88ae27a14ff6)

*/dev1243224123123* took me to a login page. Maybe *valleyDev* is a username. <br>
After trying that and a few other default style logins and not getting in, I checked the source code for anything. <br>
In the *Debugger* section of Firefox’s dev tools were some scripts to look at. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/f605a36c-1442-4210-864c-3ae83bbec4c3)

Towards the end of *dev.js* was this awesome nugget: <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/f86cd3e0-9d6b-40cd-ad1d-a26602f317bf)

Here are the contents of the text file after the login. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/c289eeb1-986f-471f-b8d0-f84c00e398f3)

Okay, there’s an FTP server but on which port if not 21? Time to run nmap again with the `-p-` flag this time. The only other port open was <b>37370</b>. <br>
```
$ ftp <username>@<target_IP> -P <port>
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/abfb6957-0b66-4454-a4d3-13916684b6b1)

Found some packet capture files; time to crack Wireshark open and search them. <br>
On the FTP pcap I found a login *anonymous:anonymous* but it didn’t work for me when I tried. <br>
In the HTTP2 pcap, there’s some other credentials. Like I suspected in the beginning, *valleyDev* is a valid username. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/a9c70c91-a3eb-42c8-8c0d-ed993d544d78)

After SSHing into the machine, I check the files in the home directory, if I have `sudo` permissions and the passwd file. <br>
```
$ sudo -l; cat user.txt; cat /etc/passwd
```
Found the first flag, now to find a way to escalate my privileges. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/9e97042f-60ad-4f75-b023-6c0af4e24238)
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/02d5c23e-f66a-43b3-a896-ece9623f04bc)

In the *home* directory there was a file of interest so I used a python server to be able to get the file to my local machine. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/47ad8cce-efcb-42ec-a4f1-d525a36a10cc)

```
(on target machine)
$ python3 -m http.server 1234
# starts python server on port 1234

(on local machine)
$ wget http//:<target_IP>:1234/valleyAuthenticator
# downloads file to local machine
```
After that I used the `file` command to find out the file is an executable. <br>
I wanted to check for name or passwords so I ran the `strings` command and couldn’t read much of anything. <br>
At the end of the output there was a few lines with *UPX* which made me think to use that tool. <br>
```
$ upx -d valleyAuthenticator
# decompressed the file

# Next, I ran the strings command again piping to grep looking for the term username.
$ strings valleyAuthenticator | grep "username"
```
I got a hit and where that was were some hashes. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/7e2526d6-f054-420a-ae21-5b1c969406e4)

I took those to `crackstation.net` to crack them. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/ef8c7f60-7cea-441f-9650-2f30434d3dfe)

Time to try those credentials and SSH in to the machine. <br>
Success! <br> <br>
Now, time to check the usual and check the `/etc/crontab` file (I forgot to do that on the other account). <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/f7a6be3c-4c7a-426f-b1f0-c38a28896973)

A cronjob that’s ran as root, perfect! <br>
When inspecting the file, I tried to edit a netcat reverse shell into it, but I didn’t have permissions to save. <br>
However, `import bas64` was at the top, so that was my next stop. <br>
```
$ locate base64
# shows all the places a file with that in it is at.

(after finding a python file)
$  nano /usr/lib/python3.8/base64.py
# that way I can try to edit this file.
```
I was able to write  to that and upload my reverse shell, now time to wait with my listener..
And now I am root of the server. <br>
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/162479aa-030d-495a-b7fd-c1b1cdc58ad2)


*NOTE: I forgot to check /etc/crontab when logged in as valleyDev, so afterwards I went back and tried to edit the base64.py file as that user and didn’t have the permissions.* <br>
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/5808a645-8c91-4486-9193-30d88fd24a39)

















