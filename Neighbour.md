TryHackMe <br>
Neighbour

---
We start off by navigating to the url http://ip_of_machine and viewing the source code of the home/ login page. <br> 
Towards the bottom of the code we see that there are guest credentials. Let’s log in.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/b5fa6322-7e58-4a53-aa5e-f24f5c208d93)

Once we successfully log in, the url reflects what user we are logged in as. Maybe we can use that as our attack vector and see if the off limits account is accessible.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/f230853d-959c-41b7-b341-6f21c37876a3)

By changing 'user=guest' to 'user=admin' we are able to see the admin account and capture the flag.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/80baac1b-dc88-4bf5-b773-1156f83a372a)

That attack vector is called an IDOR, Insecure Direct Object Reference and is a type of access control vulnerability.
It can occur when a web server receives user-supplied input to retrieve objects (files, data, documents) and it is not validated on the server-side to confirm the requested object belongs to the user requesting it.
