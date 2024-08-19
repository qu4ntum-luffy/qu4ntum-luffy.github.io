---
title: "Blackfield Hackthebox: My Walkthrough"
date: 2024-08-16 21:55:00 +0800
categories: [Blackfield, Hard, Windows-AD]
tags: [Hard-Level, Windows-PrivEsc, Windows]
---

Hello Guys, Today I have started solving the AD101 Track from Hackthebox. Since I will take my OSCP Exam soon, I am already done with Offsec labs. Now I’m a little brushing up with this series as it is quite popular and considered the best practice of active directory attack surface, For everyone who is taking the path of OSCP or is Interested in Red Team. Join me on the adventure to find & fetch resources with me together.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*B4PEzvHtWH6_fO0uFJIxKA.gif)
Analyzing Nmap result:
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*Es685Rz7vixffNein9N2ig.png)
Ports (88)kerberos,(389)ldap(signifies it is a DC), 135(rpc) & 5985(web port)(Found a few usernames)
Now on enumerating the port LDAP first,
ldapsearch -h 10.10.10.192 -x -s base namingcontexts
(Got me the domain name as blackfield.local)
Then port 53,
dig axfr @10.10.10.192 blackfield.local (Returned Nothing!!!)
port 445(SMB),
Doing smbclient on the IP,
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*UJGmqF5zCk6wKIo53yBRVQ.png)
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*lsYhaBKhKqPZQqyAa81suw.png)
So, we have read-only access to profiles. Accessing it contains all usernames….
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*QEEp-8Fbx7qHEnp95gij6g.png)
you can either mount the disk. Since it is a large directory.

![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*D0sbJwoNLrZ1QDbe7sRZiQ.png)

Just copy and paste the usernames with ls. or you can use this cmd to simply save it in a users.txt file!!

![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*QePHJasojCIwCAmFfaF5fw.png)

Which I have shamelessly stolen from 0xdf Blackfield write-up.
Now, with all the usernames in place. Let’s just try enumeration with pre-auth asreproasting with GetNPUsers.py.

![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*vsAGMDqgMkY7CBK7f9yLgw.png)


That gave me an asreproasting hash, trying to crack it down with…. hashcat.


![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*LnNqXwj5s5L8G-WoPqBriw.png)
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*3BVBoy3d9KYHSX6RzHPK5w.png)


Outputs password as “#00^BlackKnight” for the user support.
Got our first credentials…
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*djHb5yLa046ezQyEg_7c4Q.gif)
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*5hZmNURhRgT9Gf892xchbg.png)
psexec isn’t working. secretsdump either…
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*RRH9OppjdIjkdq_yZTBVqw.png)
So, tried Bloodhound for a better analysis of the Domain network.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*FQ1rGy3WyUAlaHZ5TgYrZw.gif)
Getting the .json file and analyzing them. with searching for support users on the top left. Either you can to Shortest Path from Owned Principals or At the Node Info. Click on First Degree Object Control(The first next possible privilege from this node).
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*kywZ-GtP1m3WXJcXn6ZggA.png)
Meaning the support user had the privilege of changing the password for the user audit2020. Seeing the login with rpcclient was the only leftover option. Here is a blog suggesting how to do the password change forcibly.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*XHMS_Y1ffqSYrTdzfFFXcA.gif)
Since we already brushed up all other services!! And RDP would be the other login, which we haven’t covered yet.
Extra details on rdp force password reset can be found here.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*h-LGq9oOeXiqH9Ws6EjeuQ.png)
evil winrm didn’t work so, Tried smbclient & it was succesfully.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*YqtW1jVUUOV1cu9Fr8Kd7w.png)
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*d4Oa1DiCOx_yT1P3t7ECQw.png)
unzip it has a lsass.dmp file. The juggernaut website has a better way for cracking this lsass.dmp file
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*DNDvw7yx30MR5sWDqAuA6Q.png)
With this in place…. Finally got another user with hash svc_backup!!!
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*iOGK52St_eHWL_4fgnKipw.png)
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*T0FchgQy9eBMhpad363i5Q.png)
Got the User Flag!!!!
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*g08G8kcIAiEXVSrbq2Et9w.gif)
Priv Esc from User: SVC_backup:
For a better privesc on the machine, we will need the hashes to be dumped… So, I tried downloading the hashes from SAM.hive & SYSTEM.hive but those hashes with secretsdump didn’t work…
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*2AaMedccbvC8uJJL6KQ24g.png)
Since the SeBackupPrivilege is still enabled… Finding another article…https://juggernaut-sec.com/sebackupprivilege/. It seems to be a valuable resource for getting things done!!!
So, Create a diskshadow file with these commands…!!
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*Ib9MwxvwSG6s_HLxEr5mOA.png)
Executing the shadow instructions to create a shadow drive!! with diskshadow.exe(Windows cmd defaults).
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*yfpMQtrFBKWAFAuRsuej2A.png)
So, everything from C:/ has now been saved to Z:/ drive. Let’s try robocopy now!! Earlier it was access denied. Now, Since it was a backup it was allowed now!!
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*Rr6mcQbXT925jFJ5T273FQ.png)
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*2h0z-jNdpt4cCewSDbpP2g.png)
After getting the ntds.dit and SYSTEM file. We are totally ready to get our slimy hashes!!
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*YD34PXMlf1IAtPoK_YcvNg.png)
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*fXzNxgFWbdZjbaK2aBhyDg.png)
Download all these files onto Kali Linux machine!!!
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*ZDZ-NOgddGM5gfT-iKFY4Q.png)
Finally, logging into the Administrator account with the hashes we received!!!
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*cOEA9jpOq00gsOYs9VP53Q.png)
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*kNodU_vRwKWB84O6cMtBPw.png)
And the Root Flag!!!!!!!!!!!!!!!!!!!!!!!!!
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*QP9Fpa5wqFXPEpLo8qgQDA.gif)
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*DUgipa_nhtSkRyi2t09ZGA.png)
Blackfield is a hard difficulty Windows machine featuring Windows and Active Directory misconfigurations. Anonymous / Guest access to an SMB share is used to enumerate users. Once user is found to have Kerberos pre-authentication disabled, which allows us to conduct an ASREPRoasting attack. This allows us to retrieve a hash of the encrypted material contained in the AS-REP, which can be subjected to an offline brute force attack in order to recover the plaintext password. With this user we can access an SMB share containing forensics artefacts, including an lsass process dump. This contains a username and a password for a user with WinRM privileges, who is also a member of the Backup Operators group. The privileges conferred by this privileged group are used to dump the Active Directory database, and retrieve the hash of the primary domain administrator.
why we are doing, what we are doing?
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*ID8ZWakPL0VflsQg0nPuLw.png)
Follow for more!!

