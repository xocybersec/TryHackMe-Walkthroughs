TryHackMe Agent T room

I started off spinning up Nmap to probe and see what ports are open. Port 80 shows that there’s a webpage. Let’s visit it.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/d9da5ef7-8c11-43fd-943e-74cc68158c45)

When browsing to the webpage the admin dashboard is shown without logging in.
Upon clicking the ‘admin dashboard’ button, an “/index.html can not be found error” is displayed.
You can try to intercept the request and look at the headers with Burpsuite, however, I used the dev tools built in to the browser. This is what I found that could be leveraged for further potential.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/4827535f-8bd8-4482-920b-c8c172938e88)

Using searchsploit to find if there is any type of vulnerability regarding PHP/8.1.0-dev and look what was returned.
```
$ searchsploit -t -w PHP/8.1.0-dev
```
![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/10aee2a0-b69e-4d07-91bd-0447254625f6)

After visiting the exploit-db webpage to learn about the RCE, there is a payload that you can download that is a python script. I used the reverse shell python script from the Github that was linked on exploit-db.
For the reverse shell I set up a listener with `netcat`. After that, I executed the python script and got root access to the server. All that's left is to find and capture the flag.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/dc105cb6-b26e-4f6f-b90d-a32fa7661970)

Using the `ls` command, that lists the files that are in the directory, we see the 'flag.txt' file.

Next, we use the `cat` command to display the contents of the file.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/9eea22f2-332b-45ec-942f-a4525c9813ec)
