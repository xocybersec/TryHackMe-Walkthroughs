TryHackMe <br>
Psycho Break
---
Task 1
---
Initial nmap scan of the machine shows the ports open and the OS of the machine. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/0f5310d3-cdd3-4278-a9d3-6cf2bfed7aeb)

Gobuster shows: <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/eb60106c-f9ef-4ed9-b212-46b6a4612bb9)

---
Task 2 Web
---
The source code of the page shows this comment. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/6ad69b88-d592-4585-8eb9-e62d153b15e7)

This <i>map.html</i> didn’t lead anywhere due to a broken link. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/b3937789-0a62-4217-982d-d52cf6d78eec)

When visiting the <i>/sadistRoom</i> there was a link to click for the locker room key. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/0f944165-a549-4a2b-b5c3-7948ec9542d0)

After copying the key and closing the pop-up window there’s a prompt to enter it before it’s too late. <br>
Now in <i>/lockerRoom</i> there’s some encoded text that is hiding the next key. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/9ce0a13d-1ff8-4d2d-ad6e-809a982770eb)

I used an online decoder testing different ciphers until Alphabetical Substitution worked. (z = a, y = b, etc) <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/ca399b29-46b1-46d0-a6d4-8439136bb949)

When that key was entered, I was given the option to select which room I wanted to view next. <br>
I chose <i>/SafeHeaven</i> and saw some images in a gallery. I had to check the source code while I was there. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/a9a8e4d7-743e-4048-805e-6cfd31e82e73)

I’m not sure If the creator meant to search the images but I’ll try that for now. <br>
I tried `binwalk`, `stegseek`, and `steghide` but couldn’t find anything! <br>
Then I had an idea what the creator meant about “searching”. Time to bruteforce the directories again.. <br>
That worked, here are the results. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/a7e40e7c-9465-4dc4-b151-5e220f1fee53)

On that page there’s a picture that prompted for the actual location to get the key. <br>
I had to go to images[.]google[.]com and find the location from there. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/dcbedff0-9b33-40e4-9652-c7178710ab27)

Back to the map.php page, I then went to <i>/adandonedRoom</i> <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/aaea4d68-5a7a-45c5-9277-f31860ea57eb)

I could only get `ls` to work, every other command showed not permitted. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/45f78dd0-fb7f-4a79-9a3f-e14379ab5d55)

After using:
```
Ls ..
```
I got some more results:

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/bd0f29ac-41a6-4a00-a032-16037af8d7b2)

I thought they might be hashes and `crackstation` proved me right. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/d928ff34-a779-44f7-8614-6de91e0a049d)

<br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/4194587e-dc9f-4076-aae2-01a6a265abd6)

Navigating to the first directory: <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/e3401a87-0b48-4127-84fe-4cb14609f87c)

---
Task 3 Help Mee
---
This is the content from the text file. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/14840e65-4724-4545-9731-15b0300e32e2)

After unzipping the zip file I got: <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/ccf66af8-a38e-44a4-8a96-f41d800da6a9)

The contents from the text file: <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/296fcbd6-d9ec-4f15-8b51-640f342366a5)

I used `binwalk` to examine the Table.jpg file since there should be a <i>key</i> on the <i>table</i>.
```
sudo binwalk -e Table.jpg --run-as=root
```
These were the contents: <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/86bdb248-3b29-4574-9d9c-338f8287a055)

The audio file had morse code when played so I had to find an online morse code decoder. <br>
From that file I got the passphrase to extract the contents from the jpg file. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/b159897d-256e-4f29-9532-e2b754f1bc3c)

Inside of that there were the credentials for signing in to the FTP server. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/b3f3fa68-a040-4ae7-a3b5-c3a104d8ee91)

---
Task 4 Crack it open
---
On the FTP server there were 2 files to download. One program and one file with what looked like potential passwords. <br>
After adding the permission to run the program, it prompted for a keyword to be entered. <br>
I figured it had to be one from the list so there’s a `while` statement to help do that.
```
while read LINE; do ./program "$LINE"; done < test.txt
```
I had to use `strings` on the <i>random.dic</i> file and output that into a different file and then use that because I wasn’t getting any output at first. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/d04e0916-00a7-4708-b290-7db0554dbddb)

Decoded and I have the password. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/6f1f8276-e556-4d6a-b091-15b8397d0dad)

---
Task 5 Go Capture The Flag
---
Now I can SSH into the machine with the new credentials. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/1fc56ae3-c6e4-4b3c-a761-1f88efc2c395)

I checked what I could run as root, which was nothing. <br>
Time to explore and see the other files available. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/cc913a9c-d97c-4ed5-ad39-8a816549ec29)

This was the content from the <i>.readThis.txt</i> file. Decoded, of course. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/da9e889e-9a8e-4304-b3ae-d35198a04938)

Next, was to check for any cronjobs before I went searching for that string. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/5945ca1f-d1d8-4841-8d30-c0b0c4ca1a87)

The string mentioned ended up being a cronjob, what are the odds? <br>
Since that was scheduled to run every two minutes I edited the contents and put in a reverse shell. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/eb5c36a0-37ba-4c5a-9106-9e742eff29a4)

Time to get the flag! <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/1d000794-5ddc-4853-a23e-f65f3d6114e2)
