---
title: "Soulmate Writeup"
date: 2026-01-16 14:23:32
categories: [Writeup]
tags: [Writeup,HTB]
---
 The soulmate is categorized as an easy machine in Hack the box which is best for beginners who are excited to sharpen the Pentesting skills.
 
 ![Hack](/assets/soulmate/1.png)
 
---
This is my very first Hack The Box write-up, and I’m writing this mainly for beginners who are starting their penetration testing journey — just like me.

In this walkthrough, I’ll be covering the **Soulmate** machine. Soulmate is rated as an **easy** box on Hack The Box, making it perfect for practicing enumeration, basic exploitation, and privilege escalation techniques.

If you’re new to CTFs or HTB, don’t worry — I’ll explain each step clearly and simply as i can.

Let’s get started without any further delay.

----
## Reconnaissance

- We were given an ip `10.129.23.28`. [NOTE: this IP changes every time you start the machine.]
## Enumeration

- Let's continue by using **Nmap** to do port scan to see which ports are alive before doing deep scan.
- After finding the open ports lets do an script and version detection scan.

 ![Hack](/assets/soulmate/image-99.png)

- From the scan we noticed that ip resolve into `soulmate.htb` ,so lets add it to `/etc/hosts`. so it can resolve the dns, when the ip request is made in the  browser.
- Then i started to Enumerate the browser manually found nothing much except it uses php files.
- After some directory enumeration i didn't find anything.
- But i found something interesting when done virtual host scan on the url.

 ![Hack](/assets/soulmate/image-100.png)
 
- Added the `ftp.soulmate.htb` to `/etc/hosts` to resolve the domain.

 ![Hack](/assets/soulmate/image-112.png)

- Done some `page source` enumeration and found the version of webpage.
- Found the version of the `CrushFTP` which is `Version: 11.W.657 (released 2025_03_08_07_52)`.
- And after some research i found that this version is vulnerable to [CVE-2025-31161]( https://github.com/Immersive-Labs-Sec/CVE-2025-31161 )
- By running the python program i found, the exploit will create an admin privilege user account that will be valid for one login.
- In simple terms the exploit creates an temporary access to admin panel of the crush ftp server.
- By running the python program its shows the syntax on how to use the program.

![Hack](/assets/soulmate/image-110.png)

- Lets run the exploit.

![Hack](/assets/soulmate/image-113.png)

- The exploit created the new user with admin privileges.
- Now we can access the page

![Hack](/assets/soulmate/image-114.png)

- Now we navigate to `Admin > User Manager` , we found that user **ben** have the website files, which means this is where we need to upload our reverse shell to gain access.
- We also need to write it in **php** to bypass the security measures.
- Lets change the password for ben and login to his account and upload the files.Since the account we created is temporary and not stable.
- I've generated the php reverse shell payload using the [revshell generator](https://www.revshells.com/)and upload it to the website files folder.
- And started the listener `nc -nvlp 8888`.

![Hack](/assets/soulmate/image-117.png)

- Since Ben account contains the assets the website required to load and run the website. When we add the revshell here we can execute it in the soulmate website and get the revshell.
## Exploitation

- Now lets execute the revshell from the soulmate website by trying to access the revshell.

![Hack](/assets/soulmate/image-118.png)

- we have already started listener.

![Hack](/assets/soulmate/image-119.png)

- Then i've used the `python3 -c 'import pty; pty.spawn("/bin/bash")'` cmd to gain an stable shell.
## Privilege Escalation

- After some enumeration i found that there is an **Erlang service** running on the ben user. But we need to ssh into ben first.
- So i enumerated the erlang service files and guess what? i have found some credentials.

![Hack](/assets/soulmate/image-120.png)

- With the creds found i ssh'd into the `ben` and get the **user flag**.
 
![Hack](/assets/soulmate/image-121.png)

- we know that erlang service is running on localhost in ben user account.so we ssh into the erlang service from ben user.

![Hack](/assets/soulmate/image-122.png)

- After some research we found that inside erlang server. we can os commands like `os:cmd("id").` and we found that we are already root from the cmd.
-  lets enumerate to find the **root flag**. using the erlang server cmds to navigate.

![Hack](/assets/soulmate/image-124.png)

And with that we have found both **user and root flag** and completed the soulmate machine.

This box was a great beginner-friendly experience and helped me understand the importance of proper enumeration, exploiting real-world vulnerabilities, and thinking logically during privilege escalation. From virtual host discovery to abusing CrushFTP and finally escalating privileges through Erlang, every step taught me something new.

If you’re just starting out in **Hack The Box**, I highly recommend trying this machine — it covers many core concepts you’ll see again in future boxes.

Thank you for taking the time to read my first write-up. I hope it helped you in some way.

