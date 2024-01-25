---
title: FortiEMS Setup
date: 2023-01-22
categories: [Deploy, Fortinet]
tags: [fortinet]
img_path: /assets/img/fortinet/
---


## Prerequest

According to official document, the minimum system requirements for FortiClient EMS are:

- Windows Server 2022/2019/2016/2012 R2
- 6 vCPU, 2.0 GHz 64-bit processor
- 8 GB RAM
- 40 GB free hard disk
- Gigabit Ethernet adapter
- Internet access is recommended, for SQL Server's dependencies requirement to be downloaded over the Internet. 



## My Platform
- Hypervisor: Proxmox VE 8.0.4
- VM: Windows Server 2016

## Step1 - Setup EMS Server

### Install FortiEMS
Once windows server is prepared, upload image to destination.
In this case I have `FortiClientEndpointManagementServer_7.2.2.0879_x64.exe` installation file.

While one advantage of using the Windows platform is the simplified installation process with prompts like 'yes' and 'continue,' it's important to note that Windows systems have User Account Control (UAC), requiring administrator privileges to execute the installation procedure.

### Initialize
For first time login, you will be asked to set password for admin account. Then you will be prompted to set listenning IP for EMS, so that you can access EMS portal via listenning IP.

In default, EMS portal use 443 for HTTPS, and using port 8013 for tcp connection between EMS and FortiClient. You can verify connection later by command `netstat -ano | grep 8013` on the endpoint device.




What's important comes after the connection setting is activated EMS license. In this case, we use FortiCloud for EMS licensing by following simple steps:
1. Input Contract Registration Code into FortiCloud Registration Bar.
2. Second filling the FortiCloud account in EMS and accept the license agreement terms.
3. Sync the License Agreement to FortiEMS, they will automatically communicate to each other and exchange the information such as hardware ID.

![Img](Reg.png)


![Img](Reg_success.png)



## Step2 - Connect to FortiGate

As soon as FortiEMS configuration finished, we procceed to `Security Fabric > Fabric Connector` and setup basic EMS connector.

With a pop up panel on the right side of Fortigate, you will be asked to login into FortiEMS. All will be done after you successfully verified EMS credential, then EMS's certificate will be verify on Fortigate. So that whoever's endpoint hold the certificate on FortiClient will be permitted to send traffic throught Fortigate.

![Img](EMS_connector.png)


![Img](EMS_auth.png)


![Img](EMS_conn_success.png)

As the result, we will see Fortigate appear in FortiEMS's Fabric Device Area, indicating a successful connection between FortiEMS and Fortigate. For furthrer inspection, you can utilize command `diagnose endpoint fctems test-connectivity <EMS name>` and `diagnose test application fcnacd 2` to verify connectivity.

![Img](EMS_fabric.png)

If EMS connector works fine, Fortigate will get whatever setting that has been configured on EMS while synchronization. Below image is a demonstration of ZTNA tags that I published on EMS.

![Img](FGT_tag.png)

## Step3 - FortiClient Management

### Setting Zero Trust Tagging rules

The Fortinet Device utilizes tags to identify verified devices, offering a diverse range of information to confirm system details. This empowers users and IT managers to make informed decisions about the policies they wish to apply. For example, they can decide only those devices with latest OS version and Antivirus installed can access the target website.


![Img](ztna_tag_rule.png)


![Img](ztna_tag_AV.png)

Furthermore, the tagging rules support following function:
- User in AD Group
- AntiVirus Software
- Certificate
- EMS Management
- IP Range
- Logged in Domain
- On-Fabric Status
- OS Version
- Vulnerable Devices
- Windows Security
- Common Vulnerabilities and Exposures
- Firewall Threat
- FortiEDR

After applying the tagging rules to endpoint device, which comes after the telemetry communication, the end user should see the tag on their FortiClient Avatar.

![Img](FortiClient_Avatar.png)

### Setting Endpoint Profile

I guess Endpoint profile purpose is to make EMS to communicate with FortiClient, and default setting can be inspected at `Endepoint Profile > System Settings`.

Nevertheless, it seems like EMS allows independent setting from endpoint profiles for various features:
- Remote Access
- ZTNA Destinations
- Web Filter
- Video Filter
- Vulnerability Scan
- Malware Protection
- Sandbox
- Firewall

Users can pick one of above for their needs to customize the profile.

### Setting Endpoint Policy

This is the final setting(?) to do that make EMS communicate with FortiClient via Telemetry Communication. Through including `Endpoint Group` and mounting `Endprofile Profile`, EMS should know how to identify and manage the device information. 

![Img](EMS_EndpointPolicy.png)

### Setting Installer

After all we still need to install a agent on edge device, thus our final step is to prepare setting package from `Manage Deployment` and `FortiClient Installer` section. Beyond procedure then will generate an URL, in which contain many install package  that contain EMS's certificate. Packages executed by target system will automatically deploy FortiClient and establish a authenticated connection, take TLS1.3 for example, to EMS as Telemetry Connection.

![Img](Client_ZTNAconnProxy.png)

![Img](EMS_Endpoint3.png)
