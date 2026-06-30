# DHCP and Router-on-a-Stick Routing

SAM-R0 acts as both the default gateway and DHCP server for VLAN 10 and VLAN 20. Two 802.1Q subinterfaces give the router one Layer 3 address in each VLAN, while separate DHCP pools assign clients the matching gateway and DNS server.

The switch port toward SAM-R0 is configured as a trunk so tagged frames from both user VLANs can reach the correct router subinterface.

> Router-on-a-stick is efficient for a lab and small environment, but all inter-VLAN traffic shares one physical router link. Larger designs commonly use multilayer switching for greater throughput and redundancy.

**Implemented controls:**

- Created DHCP pools for both user VLANs.
- Excluded the router interface addresses from dynamic allocation.
- Configured 802.1Q gateway subinterfaces and validated six client leases.

## Key Technical Terms

| Term | Meaning in this chapter |
|------|-------------------------|
| Router-on-a-stick | A design where one router interface carries multiple VLANs through subinterfaces and 802.1Q tags. |
| Subinterface | A logical router interface such as `g0/0/0.10` that acts as the default gateway for one VLAN. |
| DHCP pool | The address range and options a router gives to clients automatically. |
| Excluded address | An IP address kept out of DHCP leasing so infrastructure addresses are not accidentally assigned to clients. |
| Default gateway | The router address clients use to reach networks outside their own subnet. |

### Configure the DHCP pools and gateway subinterfaces

VLAN 10 uses `192.168.10.0/24` with gateway `192.168.10.1`, while VLAN 20 uses `192.168.20.0/24` with gateway `192.168.20.1`. Both pools distribute `172.19.0.100` as the DNS server.

> DHCP options must match the routed interface for the client VLAN. A lease can be assigned successfully while still producing unusable connectivity if the gateway or DNS option is incorrect.

#### SAM-R0

SAM-R0 creates one tagged gateway and one DHCP pool for each user VLAN. The gateway addresses are excluded so they cannot be leased to clients.

```cisco
configure terminal
interface GigabitEthernet0/0/0
 no shutdown
 exit
interface GigabitEthernet0/0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 exit
interface GigabitEthernet0/0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 exit
ip dhcp pool VLAN10
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 172.19.0.100
 exit
ip dhcp excluded-address 192.168.10.1
ip dhcp pool VLAN20
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
 dns-server 172.19.0.100
 exit
ip dhcp excluded-address 192.168.20.1
end
write memory
```

### Connect the router through the switch trunk

SAM-S3 FastEthernet0/24 carries the tagged VLAN traffic to SAM-R0. The consolidated command captures show the trunk, subinterface, scope, gateway, DNS, and excluded-address settings together.

> The physical router interface remains shared, but each subinterface processes only frames carrying its configured 802.1Q VLAN tag.

#### SAM-S3

SAM-S3 FastEthernet0/24 is the static trunk toward SAM-R0, allowing both tagged user VLANs to reach their matching router subinterfaces.

```cisco
configure terminal
interface FastEthernet0/24
 switchport mode trunk
 switchport trunk native vlan 99
 switchport nonegotiate
end
write memory
```

### Validate client address allocation

PC0, PC1, and PC5 receive VLAN 10 addresses; PC2, PC3, and PC4 receive VLAN 20 addresses. Every lease shows the expected `/24` mask, local gateway, and DNS server.

> A DHCP lease validates scope delivery. Routing and policy behavior are verified separately with the later ping, SSH, and browser tests.

![PC0 DHCP Lease](../../images/02-dhcp-router-on-a-stick/01-pc0-dhcp-lease.png)

<p><sub><strong>Screenshot 002 - PC0 DHCP Lease:</strong> PC0 receives 192.168.10.2 with the VLAN 10 gateway and laboratory DNS server.</sub></p>

![PC1 DHCP Lease](../../images/02-dhcp-router-on-a-stick/02-pc1-dhcp-lease.png)

<p><sub><strong>Screenshot 003 - PC1 DHCP Lease:</strong> PC1 receives 192.168.10.3 from the VLAN 10 pool.</sub></p>

![PC2 DHCP Lease](../../images/02-dhcp-router-on-a-stick/03-pc2-dhcp-lease.png)

<p><sub><strong>Screenshot 004 - PC2 DHCP Lease:</strong> PC2 receives 192.168.20.2 from the VLAN 20 pool.</sub></p>

![PC3 DHCP Lease](../../images/02-dhcp-router-on-a-stick/04-pc3-dhcp-lease.png)

<p><sub><strong>Screenshot 005 - PC3 DHCP Lease:</strong> PC3 receives 192.168.20.3 from the VLAN 20 pool.</sub></p>

![PC5 DHCP Lease](../../images/02-dhcp-router-on-a-stick/05-pc5-dhcp-lease.png)

<p><sub><strong>Screenshot 006 - PC5 DHCP Lease:</strong> PC5 receives 192.168.10.4 from the VLAN 10 pool.</sub></p>

![PC4 DHCP Lease](../../images/02-dhcp-router-on-a-stick/06-pc4-dhcp-lease.png)

<p><sub><strong>Screenshot 007 - PC4 DHCP Lease:</strong> PC4 receives 192.168.20.4 from the VLAN 20 pool.</sub></p>

---------

---

## Project Chapters

| Chapter | Description |
|---------|-------------|
| [Project Overview](../../README.md) | Main project overview, topology, environment, objectives, and outcomes |
| [Device Identity and Management Foundation](../01-device-identity-management/README.md) | Hostnames, local access, banners, console/VTY baseline, and device setup |
| [VLAN Segmentation and Trunk Hardening](../02-vlan-segmentation-trunking/README.md) | VLAN creation, access ports, trunk hardening, and trunk validation |
| [DHCP and Router-on-a-Stick Routing](../03-dhcp-router-on-a-stick/README.md) | Router subinterfaces, DHCP pools, switch trunk path, and client leases |
| [Server, DNS, and Wireless Services](../04-server-dns-wireless/README.md) | Static servers, DNS publishing, WLAN profile, WPA2 access, and wireless path validation |
| [Access-Layer Port Security](../05-port-security/README.md) | Unused-port shutdown, sticky MAC learning, violation mode, and validation limits |
| [OSPF Dynamic Routing](../06-ospf-routing/README.md) | Routed transit links, OSPF advertisements, adjacency validation, and LAN3 expansion |
| [SSH Management and Source ACLs](../07-ssh-management-acls/README.md) | SSH version 2 configuration, management access, and source-based ACL restriction |
| [Inter-VLAN Access Control](../08-inter-vlan-access-control/README.md) | Inter-VLAN isolation policy and validation of blocked and preserved reachability |
| [PAT and Internal Web Validation](../09-pat-web-validation/README.md) | PAT configuration on SAM-R2 and client DNS/HTTP validation |
| [HSRP Gateway Redundancy](../10-hsrp-redundancy/README.md) | Redundant gateway topology, HSRP active/standby roles, and validation limits |
| [STP and LACP EtherChannel](../11-stp-etherchannel/README.md) | STP root control, redundant switching, and LACP EtherChannel configuration |
| [Centralized Syslog Monitoring](../12-syslog-monitoring/README.md) | Centralized Syslog destination and event collection validation |
| [Source-Restricted Switch Management](../13-switch-management-acl/README.md) | Switch SVI management access and VLAN-based SSH allow/deny validation |
| [Testing, Results, and Recommendations](../14-testing-results-summary/README.md) | Confirmed results, skills demonstrated, limitations, and production recommendations |
