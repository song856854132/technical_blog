

## background briefing
::: success
In case I forget where I was working, this note serves as a memo for future review.
:::
 
There are two type of network architectures in openstack:
1. Provider Network
2. Self-Service Network

I found myself confused with the self-service network since I was accustomed to using VMware Esxi Hypervisor, which relies solely on vSwitches. In this setup, the physical NIC serves as an uplink port, connecting a virtual switch to a physical one. And port groups within the virtual switch provide connectivity to both VMs and the ESXi VMkernel. Which is term of Provider Network mentioned above. 
For management link, it must use provider network.


## relative config
According to [Official Document](https://docs.openstack.org/install-guide/launch-instance-networks-provider.html), to use a privider network we have to enable `enable_neutron_provider_networks`. However I didn't aware of this, deployed the openstack without provider network. 
```shell=
(kolla_lastest) chts_admin@chtsopenstack:~$ grep 'enable_neutron' /etc/kolla/config/.globals.yml
#enable_neutron_vpnaas: "no"
#enable_neutron_sriov: "no"
#enable_neutron_dvr: "no"
#enable_neutron_qos: "no"
#enable_neutron_agent_ha: "no"
#enable_neutron_bgp_dragent: "no"
#enable_neutron_provider_networks: "no"
#enable_neutron_segments: "no"
#enable_neutron_sfc: "no"
#enable_neutron_trunk: "no"
#enable_neutron_metering: "no"
#enable_neutron_infoblox_ipam_agent: "no"
#enable_neutron_port_forwarding: "no"
```


```shell=
(kolla_lastest) admin@chtsopenstack:~$ sudo cat /etc/kolla/neutron-server/ml2_conf.ini
[ml2]
type_drivers = flat,vlan,vxlan
tenant_network_types = vxlan
mechanism_drivers = openvswitch,l2population
extension_drivers = port_security

[ml2_type_vlan]
network_vlan_ranges =

[ml2_type_flat]
flat_networks = physnet1

[ml2_type_vxlan]
vni_ranges = 1:1000
```

```shell=
(kolla_lastest) admin@chtsopenstack:~$ sudo cat /etc/kolla/neutron-openvswitch-agent/openvswitch_agent.ini
[agent]
tunnel_types = vxlan
l2_population = true
arp_responder = true

[securitygroup]
firewall_driver = neutron.agent.linux.iptables_firewall.OVSHybridIptablesFirewallDriver

[ovs]
bridge_mappings = physnet1:br-ex
datapath_type = system
ovsdb_connection = tcp:127.0.0.1:6640
ovsdb_timeout = 10
local_ip = 192.168.105.147
```

## command
```shell=
(kolla_lastest) admin@chtsopenstack:~$  openstack network create \
--share --external --provider-physical-network physnet1 \
--provider-network-type flat provider_network
+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | UP                                   |
| availability_zone_hints   |                                      |
| availability_zones        |                                      |
| created_at                | 2023-07-07T03:11:47Z                 |
| description               |                                      |
| dns_domain                | None                                 |
| id                        | 9b7f09be-728c-4ba5-8c60-141195fdff90 |
| ipv4_address_scope        | None                                 |
| ipv6_address_scope        | None                                 |
| is_default                | False                                |
| is_vlan_transparent       | None                                 |
| mtu                       | 1500                                 |
| name                      | provider_network                     |
| port_security_enabled     | True                                 |
| project_id                | e9de2de1e1e7431893b901b04c40dbe8     |
| provider:network_type     | flat                                 |
| provider:physical_network | physnet1                             |
| provider:segmentation_id  | None                                 |
| qos_policy_id             | None                                 |
| revision_number           | 1                                    |
| router:external           | External                             |
| segments                  | None                                 |
| shared                    | True                                 |
| status                    | ACTIVE                               |
| subnets                   |                                      |
| tags                      |                                      |
| tenant_id                 | e9de2de1e1e7431893b901b04c40dbe8     |
| updated_at                | 2023-07-07T03:11:47Z                 |
+---------------------------+--------------------------------------+

(kolla_lastest) admin@chtsopenstack:~$ openstack subnet create \
--network provider_network \
--allocation-pool start=192.168.105.148,end=192.168.105.150 \
--dns-nameserver 168.95.1.1 \
--gateway 192.168.105.254 --subnet-range 192.168.105.0/24 provider_network_mgmt
+----------------------+--------------------------------------+
| Field                | Value                                |
+----------------------+--------------------------------------+
| allocation_pools     | 192.168.105.148-192.168.105.150      |
| cidr                 | 192.168.105.0/24                     |
| created_at           | 2023-07-07T03:23:40Z                 |
| description          |                                      |
| dns_nameservers      | 168.95.1.1                           |
| dns_publish_fixed_ip | None                                 |
| enable_dhcp          | True                                 |
| gateway_ip           | 192.168.105.254                      |
| host_routes          |                                      |
| id                   | a19fc82a-c22e-429f-bfd1-bec4a12311ad |
| ip_version           | 4                                    |
| ipv6_address_mode    | None                                 |
| ipv6_ra_mode         | None                                 |
| name                 | provider_network_mgmt                |
| network_id           | 9b7f09be-728c-4ba5-8c60-141195fdff90 |
| project_id           | e9de2de1e1e7431893b901b04c40dbe8     |
| revision_number      | 0                                    |
| segment_id           | None                                 |
| service_types        |                                      |
| subnetpool_id        | None                                 |
| tags                 |                                      |
| updated_at           | 2023-07-07T03:23:40Z                 |
+----------------------+--------------------------------------+


```