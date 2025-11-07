# The Notebook -- HTB machine Writeup
###### tags: `pentest log`

## Step 1. Info Gathering

### Nmap
![](https://i.imgur.com/5Jzscu4.png)

There are only 3 port opening: 22, 80, and 10010.


### Nitko
![](https://i.imgur.com/s0UwWz9.png)

### Wfuzz 
![](https://i.imgur.com/InK5HQd.png)

Because Nginx is a reverse proxy, we cannot directly access to the server itself. Therefore we can only see 4 directory: admin, login/out and register.

### Burpsuite
After I register an account, I worte a note and using burp proxy for testing http packet, shown as below.
```shell=
POST /85632579-588d-43fa-a17c-0707bac87ed4/notes/add HTTP/1.1
Host: 10.10.10.230
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 18
Origin: http://10.10.10.230
Connection: close
Referer: http://10.10.10.230/85632579-588d-43fa-a17c-0707bac87ed4/notes/add
Cookie: auth=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6Imh0dHA6Ly9sb2NhbGhvc3Q6NzA3MC9wcml2S2V5LmtleSJ9.eyJ1c2VybmFtZSI6IjEyMyIsImVtYWlsIjoiMTIzQGdtYWlsLmNvbSIsImFkbWluX2NhcCI6ZmFsc2V9.HPKYrqKYDish7vknxWpnteLCj7VC-_n0xlRH4JIdhNR2zY9bBndB-1wPHo7jY4DjWzA6opaPSNQ-yeOSoW7TRcUO0RtsgQqrhuuZ1imiz3WVjFbmsyadwwr3xNgRfh0n_9a4OPydBhQIidnysagiMa-GPZ5kmJFNgIJD79i39o0GOqDPwd-fRAP7E40-4ptPh5iF4kHpRT5_DCZNSQion-Jyyx29VwWHwIMNzpKKMsTCMcpHRh-Y7HpEppoQog2lr-qUxvp4wsm8CC7foos5QmTsn65m-wORHGT5NniaW6u1LvWf-_BUeYog2uZgz3YpHg89TKwktMu-3f4DhY8DiexwT2-ufxIYccK8pViPmzf-l-jML1hHjxzLthoUd4Qcl930Ei2UuQhsyNB9ZdVFDGpDeowo7tgLNAW21YtlZBoHM3hUalG8YThk5bDhZ-5Jywq23mc5sEMLjVFgzODN0pbDAxSLvkBZTFq_ftGmJFVdnnM5QewTpI9ua5z_r_w33VT3K1sm8SIRMdjwFVSL3ebp1Jy_uCGinXCyLz3s7EuRY3Q30dUEd-fCMZFfTY1W0hQCbL1o923lz2hhX4pXWxCw9A8DOPCEij9nBz5DrrtiINjd8kIjZ4EMt07PycSo1rpAWd1zK0nDSJeb0-WOBdzB_FKOIJIzkE9-LFsQKSk; uuid=85632579-588d-43fa-a17c-0707bac87ed4
Upgrade-Insecure-Requests: 1

title=abc&note=abc
```
Then we try the point of decrypting cookie authentication on Base64. 
![](https://i.imgur.com/gJJ6GmK.png)

## Step 2. Vulnerablilty Analysis

### Authentication
:::success
**JWT -- Json Web Token**
JWT 是一組字串，透過（.）切分成三個為 Base64 編碼的部分：

    Header：
    {
      "typ": "JWT",
      "alg": "RS256",
      "kid": "http://10.10.14.42:7070/priv_key"
    }
    Payload：
    {
      "username": "aaa",
      "email": "aaa@gmail.com",
      "admin_cap": true
    }
    Signature：
    -----BEGIN RSA PRIVATE KEY-----
    OXOXOXOXOXOXOXOXOXOXOXOXOXOXOXO
    -----END RSA PRIVATE KEY-----
    
![](https://i.imgur.com/lhqMDPU.png)
Formal authenticated mechanism took amount of time on searching its session.

Thus JWT came out as solution for stateless characteristic of RESTful API.
![](https://i.imgur.com/65SyyDN.png)

:::
 

## Step 3. Exploit It

### PHP reverse shell
After succeed to login after faking authentication, we find an file upload interface. This give us a chance to upload an php reverse shell that can embed to web page.

![](https://i.imgur.com/N0joLhO.png)
And then we open netcat to listen the reverse connection.
![](https://i.imgur.com/p7UoaFN.png)

### SSH
Fortunately we found a backup tar file within ssh id_rsa key. Then we open an http server to make my localhost download the backup file. Thus we can remotely ssh connect to this machine by using this public key.

![](https://i.imgur.com/lbNUvA9.png)

## Step 4. Privilege Escalation

By command `sudo -l`, we can see that there is a command ` sudo docker exec -it webapp-dev01*` that has sudo privilege to execute it without entering password.

I am not familiar with this theory, but according to others share their method.
It's common way to reverse shell but using go language, however we need to open two terminal windows.
One for executing go file that we alreasy built on localhost
![](https://i.imgur.com/k0ROA4c.png)
Another for reissue the docker exec by launching /bin/sh command 
![](https://i.imgur.com/EVvFiAp.png)


[CVE-2019-5736-PoC](https://github.com/Frichetten/CVE-2019-5736-PoC/blob/master/main.go)

[Team5 explain Docker exploit](https://teamt5.org/tw/posts/container-escape-101/)