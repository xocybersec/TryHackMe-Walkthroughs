Try Hack Me <br>
Lian_Yu
---

Nmap scan shows some open ports: <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/45b5d0c5-9ea1-4904-a832-e150dc7db990)

Gobuster scan for the directories: <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/36198e1f-d3ac-4426-8303-44c11d9f168e)

I ran gobuster again on that to see if there were more. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/82e6d10e-5b2f-4cca-93a8-793591be3407)

Visiting the */island* page showed this with the highlighted word font color matched to the page background. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/bfcf3c19-9e66-4a91-9269-66f74da88ccc)

Going to the next directory */2100*, there’s a video embedded but the account hosting it was terminated so it wouldn’t play. <br>
I checked the source code after that and found this: <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/ee1e20d3-91ab-48c6-8162-c27b595b8542)

That looks like it could be an file extension, so I ran gobuster once again with <b>.ticket</b> as an extension. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/28fa98fc-26f0-4c16-bf90-26eac487b743)

Yes! This is what is on that page: <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/4f55b133-fc77-44ce-bf7b-5ec3878274d1)

That looked short/scrambled enough to me that it was encoded or ciphered so I ran it through CyberChef and found out it was base58 encoded. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/312ed4b3-12f0-4fc1-b3db-445ec42ce8b8)

I thought that could be a password, so I tried to log in to the FTP server with vigilante:<base58_decoded_pw> <br>
And I’m in! <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/fab7d63c-b144-43e8-829c-e06c3d0f7cd6)

I downloaded the files and checked again for some more. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/7cc9af45-42c2-4954-8111-2a4d4d830640)

*.bash_history* is always a good one, so, after downloading all those I opened it to see the contents. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/bb63937c-e7da-4d49-b0a5-03f6147d3584)

The *.other_user* file was a few paragraphs that didn’t seem to have any worthy information and neither did the other files. <br>
My next idea was to check the jpg file for any hidden info. <br>
```
$ stegseek extract -sf aa.jpg
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/ecd84fcd-3ae9-45bf-b35f-27c5c5e26bad)

After unzipping that file, I found two documents. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/f71a05e4-ecdf-49da-8467-3c089cf83d8f)

The contents of the <b>shado</b> file: (potentially a password or maybe a username) <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/5a70f74f-e476-4631-ad31-791b93776772)

The contents of the <b>passwd.txt</b> file was a note.. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/1820e8e3-6e96-4921-b8c4-4efb885b4716)

I used the `file` tool to check the <b>Leave_me_alone.png</b> file and found out it was a data file. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/277fa6ce-e236-4db6-a711-e90cda132588)

I changed the extension to <b>.dat</b> and used `strings` to see if there was anything but was unsuccessful. <br>
I decided to try to SSH into the machine with the potential password I had and made a wordlist with `cewl` for usernames.  <br>
*NOTE: .other_user did end up helping.* <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/7d7eaa54-81c4-4683-8d3c-8edb30c2dcde)

Here’s what <b>.Important</b> had inside: <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/e569a74a-408d-4b6e-a3eb-bf97f71d1e29)

```
$ find / -type f -iname "secret_mission" 2>/dev/null
```

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/2c2e2caf-e14d-4beb-a385-9e223c61d1ed)

Okay, I wanted to see what I could run as root. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/663351f2-4208-42dc-bee8-4fef911a054e)

Sweet! GTFObins let me know a pro tech trick tip: <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/8b6290fd-7866-4db8-8ae0-3d9546a16ea4)

Fired that off and got root privilege! <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/d7102087-b6a0-4f4d-ae59-86ac3af07d26)

Pwned!





















