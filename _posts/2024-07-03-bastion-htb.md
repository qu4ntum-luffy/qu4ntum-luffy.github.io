---
title: "Bastion HTB"
date: 2024-07-02 21:55:00 +0800
categories: [Bastion, Medium, HTB-Walkthrough]
tags: [Bastion]
---


![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*tbBSIIDs8IMC6JiZwCU_jg.png)

SSH, SMBClient and RPC ports are open nothing else…

tried anonymous login but failed. But listing is possible.

with SMBClient -L \\\\10.10.10.134\\ -U Notauser/InvalidUser

Also we can do the same with the smbmap -H 10.10.10.134 -U notauser

![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*8AcP-5ZlpzUnlsqAK3B8RA.png)

We can see, there are read and write permissions enabled for the Backups share.

try connecting this with smbclient,


![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*WASk1dWh58o7UWlaGoAx1Q.png)

There are .vhd files inside this share, Let us mount this share for better access to this files.

sudo mount -t cifs //10.10.10.134/Backup /mnt


![Untitled](https://miro.medium.com/v2/resize:fit:828/format:webp/1*ZQz6e1azuamEZg7G2aF-cA.png)

After accessing the retrieved mount, For reading the .vhd file


![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*j_z7fWPT8Jv7Z-DkD3DFYw.png)

we can either use guestmount or qemu

for utilizing qemu, you need to create a module using modprobe, and qemu with this module can access & emulate a real OS environment using the .vhd file.

https://www.baeldung.com/linux/qemu-from-terminal

$ sudo modprobe nbd


![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*I0deJKaGpvgr8hCnJDHDQA.png)

![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*_2SuBulOHtBwUP3VAj1qYA.png)

nbd0 environment will be generated. But in my case, I have already generated nbd0, I have changed the name to nbd1.

And now we can mount to it. As new local share is available.


![Untitled](https://miro.medium.com/v2/resize:fit:640/format:webp/1*PPOXEnpI_D0NjCR9yfXEzg.png)

After mounting browsing through, we can spot a NTUSER.DAT file.

So, Files are left open. without any needed. Lets search for the SAM and SYSTEM secret files in the backups share.

find Folder-name/ -name SYSTEM & find Folder-name/ -name SAM

![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aPcZJlTDDPjsMKzg8EG_pg.png)

Got SAM and SYSTEM files? copy this files to our kali desktop.

and use samdump2 command.

![Untitled](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*aSMEYEGhZ3fI9vaMPz8LJg.png)

![Untitled](https://miro.medium.com/v2/resize:fit:640/format:webp/1*l2KVGn7KIESPWhtplO4FWA.png)

![Untitled](https://miro.medium.com/v2/resize:fit:720/format:webp/1*fuHcVjFWT5j5hDjdWqji9w.png)

After cracking the password from hashcat,


![Untitled](https://miro.medium.com/v2/resize:fit:720/format:webp/1*fuHcVjFWT5j5hDjdWqji9w.png)

But we can’t access it with the SMB Client,


![Untitled](https://miro.medium.com/v2/resize:fit:720/format:webp/1*fuHcVjFWT5j5hDjdWqji9w.png)

Might have access the other way around, As we know the SSH port is also open for this machine.

ssh L4mpje@10.10.10.134


![Untitled](https://miro.medium.com/v2/resize:fit:640/format:webp/1*mVdrCKs3BdxEdm6bW8yjgA.png)

After looking around we can spot a mremoteng application running on the machine, Which has a credentials exposing vulnerability. So, browsing to Users/L4mpje/AppData/Roaming/mremoteng there’s confcons.xml, Which has the Administrative credentials. Searching for decryption of this password hash mremoteng-python-script


![Untitled](https://miro.medium.com/v2/resize:fit:828/format:webp/1*d5FYiJu8lW9rb2UU6p8TGw.png)


![Untitled](https://miro.medium.com/v2/resize:fit:640/format:webp/1*uSnmftYMhFNa73QLS3GDgQ.png)

ssh login to Administrator.


![Untitled](https://miro.medium.com/v2/resize:fit:828/format:webp/1*5DH87VbkZVKYT1UZDfKFCg.png)