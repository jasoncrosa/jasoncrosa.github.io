---
title: THM Overpass
date: 2021-01-13 22:50:00 +0000
categories: [Security, TryHackMe]
tags: [thm, overpass, writeup]     # TAG names should always be lowercase
---

1. Connect on the URL, we found a website. Quick browsing allow us to see that there is some files available on the download section.
2. Let's start an enumeration with nmap. Only two ports are available (22 & 80)
3. Enumarate the webserver with gobuster. We discover an hidden repository (/admin). Browse to this page show a login page.
4. Let's try to authenticate on this form with brute force. This is not working (too long) and the tip said that we must not brute force.
5. After a network capture, we identify a login.js file which manage the login action
6. This code is not secure because if we have a good answer from the API, the js just create a cookie named SessionToken
7. Use the web console to just create the cookie manually and then refresh the page to be logged in magicaly.
8. On the web page, we found a RSA Private key. Use ssh2john to create the hash and use john to crash the SSH passphrase based on the previous hash.
9. Use the RSA Private key + the passphrase to logon on the box and found the first flag user.txt in the home directory.
10. Then start enumerate the box for all privilege escalation. You can use a script like linpeas or do it manually.
11. We find a cronjob running every minute as root and /etc/hosts file is writable by our user.
12. Let's use this cronjob to create a reverse shell as root user.
13. Reproduce the directory structure of the cronjob call on your attack machine. Create the buildscript.sh file with a reverse shell following the same directory structure and start a web server at the root path to match the bash call done by the cronjob. Don't forget to start your netcat listener to recover the reverse shell.
14. Update /etc/hosts file to make a local resolution of overpass.thm with your own attack machine.
15. After some seconds (1min maximum) we have a root shell available on netcat and we can found the root flag on the home directory.

