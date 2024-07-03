---
title: "Bastion HTB"
date: 2024-07-02 21:27:11 +0800
categories: [Bastion HTB]
tags: [Bastion]
---

![alt text](images/1.png)

SSH, SMBClient and RPC ports are open nothing else…

tried anonymous login but failed. But listing is possible. 

with SMBClient -L \\\\10.10.10.134\\ -U Notauser/InvalidUser

Also we can do the same with the smbmap -H 10.10.10.134 -U notauser
![alt text](images/3.png)

We can see, there are read and write permissions enabled for the Backups share.

try connecting this with smbclient, 

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/efc222bf-a41d-420a-a741-6e9264ec1eac/66d69863-98fd-4005-b05e-c0879cb6da59/Untitled.png)

There are .vhd files inside this share, Let us mount this share for better access to this files.

sudo mount -t cifs //10.10.10.134/Backup /mnt 

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/efc222bf-a41d-420a-a741-6e9264ec1eac/d1f4ea5f-e08a-4f39-8b7f-e53afc24ab2a/Untitled.png)

After accessing the retrieved mount, For reading the .vhd file

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/efc222bf-a41d-420a-a741-6e9264ec1eac/c7f4a81e-c4ef-4ada-9e16-2928964c0999/Untitled.png)

we can either use guestmount or qemu 

for utilizing qemu, you need to create a module using modprobe, and qemu with this module can access & emulate a real OS environment using the .vhd file.

https://www.baeldung.com/linux/qemu-from-terminal

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/efc222bf-a41d-420a-a741-6e9264ec1eac/654a39d8-55a7-4d67-b212-f8989133241a/Untitled.png)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/efc222bf-a41d-420a-a741-6e9264ec1eac/04f4689f-a91e-4fcc-9709-4263974bf08f/Untitled.png)

nbd0 environment will be generated. But in my case, I have already generated nbd0, I have changed the name to nbd1.

And now we can mount to it. As new local share is available.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/efc222bf-a41d-420a-a741-6e9264ec1eac/b3a212e9-f857-427c-b6c5-685c78fdcf1c/Untitled.png)

After mounting browsing through, we can spot a NTUSER.DAT file.

So, Files are left open. without any needed. Lets search for the SAM and SYSTEM secret files in the backups share.

find Folder-name/ -name *SYSTEM* & find Folder-name/ -name *SAM*

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/efc222bf-a41d-420a-a741-6e9264ec1eac/1ae4993b-0c5c-4f2f-9d04-a9070bfbdf9a/Untitled.png)

Got SAM and SYSTEM files? copy this files to our kali desktop.

and use samdump2 command.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/efc222bf-a41d-420a-a741-6e9264ec1eac/be77f683-4645-488f-94be-73fb4f56058f/Untitled.png)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/efc222bf-a41d-420a-a741-6e9264ec1eac/f14af560-c0e8-404c-a3d1-460e4a127cc2/Untitled.png)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/efc222bf-a41d-420a-a741-6e9264ec1eac/3164128c-4b57-46a0-8067-9499552ca128/Untitled.png)

After cracking the password from hashcat,

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/efc222bf-a41d-420a-a741-6e9264ec1eac/81c452ac-6f49-4629-8157-09c435c8bec0/Untitled.png)

But we can’t access it with the SMB Client,

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/efc222bf-a41d-420a-a741-6e9264ec1eac/8975bd91-040b-4b32-aa0e-6f38de54f17e/Untitled.png)

Might have access other way around, As we know the ssh port is also open for this machine. 

ssh L4mpje@10.10.10.134

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/efc222bf-a41d-420a-a741-6e9264ec1eac/251319c2-03a6-44ef-a744-8ce397cee9de/Untitled.png)

.

After looking around we can spot a mremoteng application running on the machine, Which has a credentials exposing vulnerability. So, browsing to Users/L4mpje/AppData/Roaming/mremoteng there’s  confcons.xml, Which has the Administrative credentials. Searching for decryption of this password hash [mremoteng-python-script](https://github.com/kmahyyg/mremoteng-decrypt/blob/master/mremoteng_decrypt.py)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/efc222bf-a41d-420a-a741-6e9264ec1eac/434775ad-6c35-4ee1-bec2-2d9226c4a6c4/Untitled.png)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/efc222bf-a41d-420a-a741-6e9264ec1eac/a0172928-4a2f-4ec0-8880-1ddde4bad666/Untitled.png)

ssh login to Administrator.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/efc222bf-a41d-420a-a741-6e9264ec1eac/34cf0bfc-f715-46a2-b756-dca45b6b33e1/Untitled.png)
# Bastion
