TryHackMe <br>
Surfer
---
An initial `nmap` scan showed two ports open and a disallowed webpage on the robots.txt file. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/3bfca8c6-13a9-4fa4-a5c4-81d541ff06f3)

Visiting the disallowed page a chat dialogue is shown. <br>
And it sounds like `admin:admin` are the credentials. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/09d1e0fb-bdf4-46b9-b790-87b5935303ea)

I ran a `Gobuster` scan to see if there were any other directories of interest. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/5bd63e4a-cb31-4b23-8a0d-bcc66e065a97)

I also ran a `Nikto` scan to be sure there wasn’t any other low hanging fruit. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/dc504dbe-e4a9-497f-a8d5-c07cbaf8a3ed)

At the main web page, the credentials from earlier did work and I was able to log in as admin. <br>
On the right side of the page there was a file that was located at <i>/internal/admin.php</i> <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/af0a1292-fb65-4a4d-96ca-255a60861029)

When trying to access the file, the message displayed is that it can only be accessed locally. <br>
Okay, there has to be a way to open that file.. At the bottom of the main page there is an `export to pdf` button. <br>
The report that is generated from the <i>export to pdf</i> looks like it could potentially be vulnerable to SSRF (server side request forgery). <br>
Upon intercepting the same report request with `Burpsuite` I tried to have it display the page as the report. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/107eac23-808f-42bf-9866-0385a706d960)

By changing the <i>server-info.php</i> to <i>internal/admin.php</i> and forwarding that request the server produces the file and the flag is displayed. <br>

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/01ec8292-4dbe-4f4f-8fdf-6e85fbb146b6)
<br>
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/a0e90893-e318-4559-9a3d-9e8c64f44771)
