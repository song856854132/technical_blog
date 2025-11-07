# Armageddon -- HTB machine Writeup
###### tags: `pentest log`

## Step 1. Info Gathering

### Nmap
![](https://i.imgur.com/1KHGQCV.png)

![](https://i.imgur.com/dMInkoG.png)

### Nitko
![](https://i.imgur.com/E08XNpN.png)
![](https://i.imgur.com/nFm17eO.png)

from Nikto, we can assume that vulnerablity is hiding from **php** or **drupal**. Especially OSVDB point out the Drupa at the end of scanning.
### wfuzz 
![](https://i.imgur.com/avASs4v.png)
Here we can collect some useful payload, sudh as "includes", "script" and "sites".
![](https://i.imgur.com/H59kMsQ.png)
![](https://i.imgur.com/9OTO0zn.png)

### Burpsuite 
![](https://i.imgur.com/Mz1M8Er.png)
It's a login page, thus we try random "123" to test the message sent to backend by using burpsuite. 
![](https://i.imgur.com/Smv7c45.png)
Result of data it sent was paintext. It gives a hing that maybe we can use Hydra or John to break through the login system. 
## Step 2. Vulnerablilty Analysis

### Searchsploit
![](https://i.imgur.com/mAJmEmu.png)

### PHP User Database
:::success

**password_hash(PHP 5 >= 5.5.0, PHP 7, PHP 8)**

e.x. echo password_hash("rasmuslerdorf", PASSWORD_DEFAULT);
$2y$10$.vGA1O9wmRjrwAVXD98HNOgsNpDczlqm3Jq7KnEd1rVAGv3Fykk1a

:::


## Step 3. Exploit It

We use metasploit's drupal exploit module to access the remote host.
### Metasploit
![](https://i.imgur.com/GyDbGtW.png)
![](https://i.imgur.com/7NQmpoK.png)
![](https://i.imgur.com/otWNkqx.png)
Then fill out the LHOST and RHOST.
![](https://i.imgur.com/MJE2A75.png)



![](https://i.imgur.com/CQIsXPz.png)

![](https://i.imgur.com/9TGWVOg.jpg)

### John the Ripper
![](https://i.imgur.com/6g6s6rH.png)
Now we get the password for website login page and ssh(hopefully)
![](https://i.imgur.com/PrDGvgZ.png)


## Step 4. Privilege Escalation
![](https://i.imgur.com/KXtATjz.png)

Just Google it. Searching for what can we use to escalate the privilege. 
![](https://i.imgur.com/p6DAghZ.png)

But it didn't success. Waiting for debugging.
![](https://i.imgur.com/12S4LtS.png)
