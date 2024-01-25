---
title: HTB Write Up - Cap
date: 2023-11-09 12:00:00
categories: [Pentest, HTB]
tags: [HTB]
img_path: /assets/img/htb/
---


###### tags: `pentest log`

:::info
**FTP**
The File Transfer Protocol (FTP), using a client–server structure to 

:::

![](https://i.imgur.com/JBZMwoI.png)

The target we play today, Cap, is quiet easy one and noob friendly. From HTB dashboard, we can acknowledge its IP address : 10.10.10.245.

## Step 1. Info Gathering
So first step we might want to begin with info gathering.
![](https://i.imgur.com/GmwVygu.png)

The target offers 3 kind of service: ftp, ssh and http. 
Now, let's go check about http service, what's more information about this machine that website tell us.

![](https://i.imgur.com/yfnhcsr.png)

This is a network flow monitoring website, offering pcap files to be downloaded. As we first entered the website, we already logged in as "nathan" ID, which must be the administrator. 
![](https://i.imgur.com/pRkzaJT.png)

Next, we downloaded the pcap file and opened it with wireshark.  


## Step 2. Search for Vulnerability

![](https://i.imgur.com/UKMVhOe.png)

Occasionally, we got his password while using ftp. 
Then, it's a good chance for us to try it out

It succeeded, and we got a user flag from ftp server.     Then we tried ssh and it worked as well. 


## Step 3. Exploit it
![](https://i.imgur.com/waeGUXg.jpg)

![](https://i.imgur.com/hdiGDvv.png)

![](https://i.imgur.com/SEfmGch.jpg)

Finally, by privilege escalation, we pwn the machine and found the root flag.

