---
title: GOAD LAB Setup on Porxmox
date: 2023-12-07
categories: [Deploy, AD]
tags: [proxmox,AD,KVM,terraform]
---




## What is GOAD?
---
>GOAD is a pentest active directory LAB project. The purpose of this lab is to give pentesters a vulnerable Active directory environment ready to use to practice usual attack techniques. [name=Orange Official Github page]

### Two type of LABs scale
- GOAD : 5 vms, 2 forests, 3 domains (full goad lab)
![Img](https://github.com/Orange-Cyberdefense/GOAD/blob/main/docs/img/GOAD_schema.png)


- GOAD-Light: 3 vms, 1 forest, 2 domains (smaller goad lab for those with a smaller pc)
![image alt](https://github.com/Orange-Cyberdefense/GOAD/blob/main/docs/img/GOAD-Light_schema.png)

### Provider support

- virtualbox
- vmware
- azure
- proxmox

There is a shell script, goad.sh, easy to start:
```shell
// specify lab scale, platform and method
./goad.sh -t install -l [GOAD|GOAD-Light]  -p [virtualbox|vmware|proxmox]  -m [docker|local]
```

However, in the install_provisioning process, there is no provider inventory file of proxmox for GOAD-Light. It's better to follow [MayFly's blog](https://mayfly277.github.io/posts/GOAD-on-proxmox-part2-packer/) step by step, getting all the stuff ready. 




LAB Setup on Proxmox
---
All GOAD LAB installation has three parts, proxmoxs' has no exception:

1. Templating : this will create the template to use (needed only for proxmox)
2. Providing : this will instantiate the virtual machines depending on your provider
3. Provisioning : it is always made with ansible, it will install all the stuff to create the lab


### Pre-request


### Templating, ~~which will get a template image file on proxmox~~
#### Downlaod ISO
Download the Windows Server ISO image from the official website and upload it to the local image storage on Proxmox. Alternatively, for a more efficient process, you can simply set the source to download from the specified URL directly within Proxmox.
![image alt]()

#### Prepare cloudbase-init
> The main goal of cloudbase-init is to provide guest cloud initialization for Windows and other OS, such as disk volume expansion, user creation, password generation, custom PowerShell, CMD and Bash scripts execution, Heat templates, PowerShell remoting setup. And there’s no limitation in the type of supported hypervisors. This service can be used on instances running on Hyper-V, KVM, Xen, ESXi etc. [name=cloudbase-init offical document]

Cloudbase-init has three part, all these can be obtain at `/path/to/GOAD/packer/proxmox/scripts/sysprep`:
1. sysprep
2. config file
3. file execution

```shell
// download cloudbase-init installer from official site 
cd /root/GOAD/packer/proxmox/scripts/sysprep && wget https://cloudbase.it/downloads/CloudbaseInitSetup_Stable_x64.msi

// check cloudbase-init file
~/GOAD/packer/proxmox/scripts/sysprep# ls
cloudbase-init.conf    cloudbase-init.ps1                 cloudbase-init-unattend.conf  sysprep.bat
cloudbase-init-p2.ps1  CloudbaseInitSetup_Stable_x64.msi  cloudbase-init-unattend.xml
```
#### Create Dedicated User and Role for packer
```shell
pveum useradd infra_as_code@pve
pveum passwd infra_as_code@pve
pveum roleadd Packer -privs "VM.Config.Disk VM.Config.CPU VM.Config.Memory Datastore.AllocateTemplate Datastore.Audit Datastore.AllocateSpace Sys.Modify VM.Config.Options VM.Allocate VM.Audit VM.Console VM.Config.CDROM VM.Config.Cloudinit VM.Config.Network VM.PowerMgmt VM.Config.HWType VM.Monitor"
pveum acl modify / -user 'infra_as_code@pve' -role Packer
```
Then you will get a User packer for later use as a result 
![Img]()



#### Packer hcl file
1. config.auto.pkrvars.hcl
2. windows_server2019_proxmox_cloudinit.pkvars.hcl


#### Launch Packer
```shell
packer validate -var-file=windows_server2016_proxmox_cloudinit.pkvars.hcl .
packer build -var-file=windows_server2016_proxmox_cloudinit.pkvars.hcl .
```

Result:
![Img of image template]()

### Providing
#### Prepare Terraform Config File

There are 3 main files for terraform:
0. main.tf -> main file, no need to change
1. variables.tf -> all var used in main.tf can be specified here, including promox API that terraform interact with. 
2. goad.tf -> 

```shell


~/GOAD/ad/GOAD/providers/proxmox/terraform# ls
variables.tf.template    goad.tf    main.tf    terraform.tfstate    ws01.tf.exclude
```
#### VM Create
```shell
// read all config file mentioned above and generate a execuatable binary file, goad.plan
terraform init
terraform plan -out goad.plan
terraform apply "goad.plan"


// if certain target needed specified
terraform plan -target=proxmox_vm_qemu.dc02 -out goad.fix.plan
terraform apply "goad.fix.plan"
```
### Provisioning
#### Pre-request 
```shell
cd /root/GOAD/ansible
ansible-galaxy install -r requirements.yml
```


#### Ansible Provisioning 
When it comes to provisioning, nothing is more handy than ansible does. GOAD has consolidated these Ansible playbook commands into a shell script. Given the infrastructure changes, we need to be adept at modifying these YAML files to align with our network environment.
```shell
#!/bin/bash

ERROR=$(tput setaf 1; echo -n "[!]"; tput sgr0)
OK=$(tput setaf 2; echo -n "[✓]"; tput sgr0)
INFO=$(tput setaf 3; echo -n "[-]"; tput sgr0)

RESTART_COUNT=0
MAX_RETRY=3

#ANSIBLE_COMMAND="ansible-playbook -i ../ad/azure-sevenkingdoms.local/inventory"
echo "[+] Current folder $(pwd)"
echo "[+] Ansible command : $ANSIBLE_COMMAND"

function run_ansible {
    # Check if the maximum number of retries is reached, then exit with an error code
    if [ $RESTART_COUNT -eq $MAX_RETRY ]; then
        echo "$ERROR $MAX_RETRY restarts occurred, exiting..."
        exit 2
    fi

    echo "[+] Restart counter: $RESTART_COUNT"
    let "RESTART_COUNT += 1"

    echo "$OK Running command: $ANSIBLE_COMMAND $1"

    # Run the command with a timeout of 20 minutes to avoid failure when ansible is stuck
    timeout 20m $ANSIBLE_COMMAND $1
    exit_code=$(echo $?)

    if [ $exit_code -eq 4 ]; then # ansible result code 4 = RUN_UNREACHABLE_HOSTS
        echo "$ERROR Error while running: $ANSIBLE_COMMAND $1"
        echo "$ERROR Some hosts were unreachable, we are going to retry"
        run_ansible $1

    elif [ $exit_code -eq 124 ]; then # Command has timed out, relaunch the ansible playbook
        echo "$ERROR Error while running: $ANSIBLE_COMMAND $1"
        echo "$ERROR Command has reached the timeout limit of 20 minutes, we are going to retry"
        run_ansible $1

    elif [ $exit_code -eq 0 ]; then # ansible result code 0 = RUN_OK
        echo "$OK Command successfully executed"
        RESTART_COUNT=0 # Reset the counter for the next playbook
        return 0

    else
        echo "$ERROR Fatal error from ansible with exit code: $exit_code"
        echo "$ERROR We are going to retry"
        run_ansible $1
    fi
}

# We run all the recipes separately to minimize faillure
echo "[+] Running all the playbook to setup the lab"
run_ansible build.yml
run_ansible ad-servers.yml
run_ansible ad-parent_domain.yml
# Wait after the child domain creation before adding servers
run_ansible ad-child_domain.yml
echo "$INFO Waiting 5 minutes for the child domain to be ready"
sleep 5m
run_ansible ad-members.yml
run_ansible ad-trusts.yml
run_ansible ad-data.yml
run_ansible ad-gmsa.yml
run_ansible laps.yml
run_ansible ad-relations.yml
run_ansible adcs.yml
run_ansible ad-acl.yml
run_ansible servers.yml
run_ansible security.yml
run_ansible vulnerabilities.yml
run_ansible reboot.yml
echo "$OK your lab is successfully setup ! have fun ;)"
```

```shell

cd /root/GOAD/ansible
export ANSIBLE_COMMAND="ansible-playbook -i ../ad/GOAD/data/inventory -i ../ad/GOAD/providers/proxmox/inventory"
../scripts/provisionning.sh
```

#### Finish, Snapshot

A snapshot is strongly suggested, because I was trapped by some windows issue and got to rebuild all of the LAB again. I wished I could had took the snapshot tho.
```shell
qm list
 VMID NAME                 STATUS     MEM(MB)    BOOTDISK(GB) PID
 100 Windows-Server-2016  running    4096              50.00 1393
 101 Windows10            stopped    8192              64.00 0
 102 WinServer2019x64-cloudinit-qcow2 stopped    4096  40.00 0
 103 GOAD-DC02            running    3096              40.00 1018040
 104 GOAD-DC01            running    3096              40.00 1018125
 105 GOAD-SRV02           running    4096              40.00 1018045


qm snapshot 103 'snapshot-'$(date '+%Y-%m-%d--%H-%M') --vmstate 1 --description "Provisioning Finished"
qm snapshot 104 'snapshot-'$(date '+%Y-%m-%d--%H-%M') --vmstate 1 --description "Provisioning Finished"
qm snapshot 105 'snapshot-'$(date '+%Y-%m-%d--%H-%M') --vmstate 1 --description "Provisioning Finished"
```



###### tags: `Active Directory` `Proxmox` `pentest`
