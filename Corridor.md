TryHackMe <br>
Corridor
---
This CTF starts with a web page with a background of a corridor and the doors in the image lead to other web pages.
I started off by examining the source code to see what the URLs show or if there’s anything else I can modify/ view.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/0a4b89fc-28a5-42eb-ae00-6d836555dfd6)

<i>(Sidenote: TryHackMe did leave a clue mentioning that the URL endpoints look like hashes)</i><br>

Now with that in mind, I went to `Cyberchef` to see if it could give me a clue to which kind of hash they are or if there were any other info I could find out. <br>
That tool mentioned it could be MD5 hashed, so I went to `Crackstation` and entered the first hash. It returned a ‘1’. <br>
I repeated the prior steps for the second URL endpoint, that hash returned ‘2’. As you would imagine that repeated in ascending order. 

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/df7dd1d0-8670-4316-8af0-7c5c058e0852)

In the source code I also tried manipulating the `cords` values but I couldn’t get anything to happen. <br>
My thought process on what to try next was to hash the value of ‘0’ in MD5 and swap one of the URL endpoints to the new hash. <br>
After doing that and requesting to go to that web page I was greeted with the flag.

![image](https://github.com/xocybersec/TryHackMe-Walkthroughs/assets/91302698/2f29283f-43c1-40bd-9114-6cbbb441ccd4)
