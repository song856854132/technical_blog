---
title: Openstack Deployment with Kolla-Ansible
date: 2023-11-29
categories: [Deploy, Openstack]
tags: [Openstack,KVM]
img_path: /assets/img/kvm/
---

## Openstack Intro
> OpenStack is a open standard cloud computing platform. It is mostly deployed as infrastructure-as-a-service (IaaS) in both public and private clouds where virtual servers and other resources are made available to users. - [Wikipedia](https://zh.wikipedia.org/zh-tw/OpenStack) 
 
As a virtualize platform, it supports multiple hypervisors such as KVM, Xen, VMware, and Hyper-V. Among them, Openstack with KVM stands out as one of the most popular implementations.

- Openstack Component in Nuts (Graph)
![Img](https://docs.openstack.org/contributors/_images/map-of-OpenStack-projects.png)

- Architecture of Openstack and KVM
![Img](https://static.packt-cdn.com/products/9781784399054/graphics/3990_12_01.jpg)

## Deployment
### Kolla-Ansible
One of the most painful thing of starting to use OpenStack is the inconvenience of the deployment process, which often leads to unknown failures during installation. Kolla-ansible replace the inflexible deployment process with container, which organized by Ansible playbook. 

By defining the associated variables in `/etc/kolla/global.yml`, these templating configuration options will then convert into Kolla’s Ansible playbooks for later container deployment.

Container plays a crucial role because it's lighted-weight, flexible and portable characteristic greatly enhance the efficiency of the deployment process.

Once the deployment is completed using Kolla-ansible, the OpenStack service container will execute tasks similar to the original charms deployment method: using [libvirt](https://en.wikipedia.org/wiki/Libvirt) API to interact with KVM hypervisor.

![Img](https://upload.wikimedia.org/wikipedia/commons/thumb/d/d0/Libvirt_support.svg/300px-Libvirt_support.svg.png)

### Step by Step Guide
Official Guide(https://docs.openstack.org/kolla-ansible/latest/user/quickstart.html) is reliable document if you've choose latest version. How funny was that I chose to deploy YOGA for most updated functionality, but went wrong over and over again.

Here is the brief summarize of the guide context.

Step 1: Install Dependencies

Step 2: Clone Kolla-Ansible Repository

Step 3: Copy the Configuration File to Destination

> all-in-one, ansible inventory file, show 6 types of hosts to deploy and all serve on localhost.

```shell
[control]
localhost       ansible_connection=local

[network]
localhost       ansible_connection=local

[compute]
localhost       ansible_connection=local

[storage]
localhost       ansible_connection=local

[monitoring]
localhost       ansible_connection=local

[deployment]
localhost       ansible_connection=local
```
> globals.yml main configuration
Step 4: Edit the Configuration Files

```shell
kolla_base_distro: "ubuntu"
network_interface: "eth0"
neutron_external_interface: "eth1"
kolla_internal_vip_address: "10.1.0.250"
```
Step 5: Bootstrap Servers
Step 7: Deploy OpenStack
Step 8: Post-Deployment Setup



### Result of Kolla-Ansible Deployment
```shell
~$ sudo docker ps -a
CONTAINER ID   IMAGE                                                                   COMMAND                  CREATED       STATUS                 PORTS     NAMES
1ae523539d56   quay.io/openstack.kolla/horizon:master-ubuntu-jammy                     "dumb-init --single-…"   7 days ago   Up 7 days (healthy)             horizon
c30f90bd97f6   quay.io/openstack.kolla/heat-engine:master-ubuntu-jammy                 "dumb-init --single-…"   7 days ago   Up 7 days (healthy)             heat_engine
83a5425f4f7c   quay.io/openstack.kolla/heat-api-cfn:master-ubuntu-jammy                "dumb-init --single-…"   7 days ago   Up 7 days (healthy)             heat_api_cfn
922cfb0ade52   quay.io/openstack.kolla/heat-api:master-ubuntu-jammy                    "dumb-init --single-…"   7 days ago   Up 7 days (healthy)             heat_api
8df3c8193bb5   quay.io/openstack.kolla/neutron-metadata-agent:master-ubuntu-jammy      "dumb-init --single-…"   7 days ago   Up 7 days (healthy)             neutron_metadata_agent
a9bd1226c31d   quay.io/openstack.kolla/neutron-l3-agent:master-ubuntu-jammy            "dumb-init --single-…"   7 days ago   Up 7 days (healthy)             neutron_l3_agent
b9cefdbd4580   quay.io/openstack.kolla/neutron-dhcp-agent:master-ubuntu-jammy          "dumb-init --single-…"   7 days ago   Up 7 days (healthy)             neutron_dhcp_agent
e92abf1e638c   quay.io/openstack.kolla/neutron-openvswitch-agent:master-ubuntu-jammy   "dumb-init --single-…"   7 days ago   Up 7 days (healthy)             neutron_openvswitch_agent
a2b51983fa51   quay.io/openstack.kolla/neutron-server:master-ubuntu-jammy              "dumb-init --single-…"   7 days ago   Up 7 days (healthy)             neutron_server
f7db7a7410e1   quay.io/openstack.kolla/openvswitch-vswitchd:master-ubuntu-jammy        "dumb-init --single-…"   7 days ago   Up 7 days (healthy)             openvswitch_vswitchd
fa7ad45e7269   quay.io/openstack.kolla/openvswitch-db-server:master-ubuntu-jammy       "dumb-init --single-…"   7 days ago   Up 7 days (healthy)             openvswitch_db
84a1b84beac2   quay.io/openstack.kolla/nova-compute:master-ubuntu-jammy                "dumb-init --single-…"   7 days ago   Up 7 days (healthy)             nova_compute
75699d4058da   quay.io/openstack.kolla/nova-libvirt:master-ubuntu-jammy                "dumb-init --single-…"   7 days ago   Up 7 days (healthy)             nova_libvirt
f1070570eeae   quay.io/openstack.kolla/nova-ssh:master-ubuntu-jammy                    "dumb-init --single-…"   7 days ago   Up 7 days (healthy)             nova_ssh
c5b4601a0f76   quay.io/openstack.kolla/nova-novncproxy:master-ubuntu-jammy             "dumb-init --single-…"   7 days ago   Up 7 days (healthy)             nova_novncproxy
48cb9f48a1d7   quay.io/openstack.kolla/nova-conductor:master-ubuntu-jammy              "dumb-init --single-…"   7 days ago   Up 7 days (healthy)             nova_conductor
b3d011f128d9   quay.io/openstack.kolla/nova-api:master-ubuntu-jammy                    "dumb-init --single-…"   7 days ago   Up 7 days (healthy)             nova_api
7c617ad17d6e   quay.io/openstack.kolla/nova-scheduler:master-ubuntu-jammy              "dumb-init --single-…"   7 days ago   Up 7 days (healthy)             nova_scheduler
73040875217b   quay.io/openstack.kolla/placement-api:master-ubuntu-jammy               "dumb-init --single-…"   7 days ago   Up 7 days (healthy)             placement_api
59cac6707023   quay.io/openstack.kolla/glance-api:master-ubuntu-jammy                  "dumb-init --single-…"   7 days ago   Up 7 days (healthy)             glance_api
ee3ac2e0c2fc   quay.io/openstack.kolla/keystone:master-ubuntu-jammy                    "dumb-init --single-…"   7 days ago   Up 7 days (healthy)             keystone
f0d2fe5e75f9   quay.io/openstack.kolla/keystone-fernet:master-ubuntu-jammy             "dumb-init --single-…"   7 days ago   Up 7 days (healthy)             keystone_fernet
b9153c1cd1b2   quay.io/openstack.kolla/keystone-ssh:master-ubuntu-jammy                "dumb-init --single-…"   7 days ago   Up 7 days (healthy)             keystone_ssh
1f49340675a6   quay.io/openstack.kolla/rabbitmq:master-ubuntu-jammy                    "dumb-init --single-…"   7 days ago   Up 7 days (healthy)             rabbitmq
2af1b258b048   quay.io/openstack.kolla/memcached:master-ubuntu-jammy                   "dumb-init --single-…"   7 days ago   Up 7 days (healthy)             memcached
54ef9ec23bfc   quay.io/openstack.kolla/mariadb-server:master-ubuntu-jammy              "dumb-init -- kolla_…"   7 days ago   Up 7 days (healthy)             mariadb
52e75ac7af0a   quay.io/openstack.kolla/cron:master-ubuntu-jammy                        "dumb-init --single-…"   7 days ago   Up 7 days                       cron
aeb85d93d58f   quay.io/openstack.kolla/kolla-toolbox:master-ubuntu-jammy               "dumb-init --single-…"   7 days ago   Up 7 days                       kolla_toolbox
0405af7aed05   quay.io/openstack.kolla/fluentd:master-ubuntu-jammy                     "dumb-init --single-…"   7 days ago   Up 7 days                       fluentd
```

### Post Deployment
Kolla-Ansible also give a convinient shell script, `init-runonce`, to create an sample ready-to-go environment.