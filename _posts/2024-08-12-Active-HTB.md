---

title: "Active HTB"
date: 2024-01-11 20:17:41 +0800
categories: [Active-HTB, Easy-Level]
tags: [Active, Easy-Windows, HTB-Walkthrough]

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