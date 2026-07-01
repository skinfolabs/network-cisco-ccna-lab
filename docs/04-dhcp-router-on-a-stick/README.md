# DHCP and Router-on-a-Stick Routing

SAM-R0 acts as the default gateway and DHCP server for VLAN 10 and VLAN 20. Two 802.1Q subinterfaces provide one Layer 3 address per VLAN, while separate DHCP pools assign the matching gateway and DNS server.

## Technical Context

The switch port toward SAM-R0 is a trunk so tagged frames from both user VLANs reach the correct router subinterface.

> Router-on-a-stick is efficient for a lab, but all inter-VLAN traffic shares one physical router link. Larger designs often use multilayer switching.

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

---

## Detailed Walkthrough

### Step 01 - Configure the DHCP pools and gateway subinterfaces

VLAN 10 uses `192.168.10.0/24` with gateway `192.168.10.1`, while VLAN 20 uses `192.168.20.0/24` with gateway `192.168.20.1`. Both pools distribute `172.19.0.100` as the DNS server.

> DHCP options must match the routed interface for the client VLAN. A lease can be assigned successfully while still producing unusable connectivity if the gateway or DNS option is incorrect.

#### SAM-R0

SAM-R0 creates one tagged gateway and one DHCP pool per user VLAN. Gateway addresses are excluded from leasing.

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

---

### Step 02 - Connect the router through the switch trunk

SAM-S3 FastEthernet0/24 carries tagged VLAN traffic to SAM-R0.

> The physical router interface is shared, but each subinterface processes only frames with its configured 802.1Q tag.

#### SAM-S3

SAM-S3 FastEthernet0/24 is the static trunk toward SAM-R0.

```cisco
configure terminal
interface FastEthernet0/24
 switchport mode trunk
 switchport trunk native vlan 99
 switchport nonegotiate
end
write memory
```

---

### Step 03 - Validate client address allocation

PC0, PC1, and PC5 receive VLAN 10 leases; PC2, PC3, and PC4 receive VLAN 20 leases. Each lease shows the expected `/24` mask, gateway, and DNS server.

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

---

## Validation and Summary

Validation confirms that router subinterfaces, DHCP pools, and client leases deliver the expected addresses, gateways, and DNS settings for later ACL, DNS, and web tests.

---

## Project Chapters

| # | Chapter |
|---|---------|
| 0 | [Project Overview](../../README.md) |
| 1 | [Topology and Lab Environment](../01-topology-and-lab-environment/README.md) |
| 2 | [Device Identity and Management Foundation](../02-device-identity-management/README.md) |
| 3 | [VLAN Segmentation and Trunk Hardening](../03-vlan-segmentation-trunking/README.md) |
| 4 | [DHCP and Router-on-a-Stick Routing](../04-dhcp-router-on-a-stick/README.md) |
| 5 | [Server, DNS, and Wireless Services](../05-server-dns-wireless/README.md) |
| 6 | [Access-Layer Port Security](../06-port-security/README.md) |
| 7 | [OSPF Dynamic Routing](../07-ospf-routing/README.md) |
| 8 | [SSH Management and Source ACLs](../08-ssh-management-acls/README.md) |
| 9 | [Inter-VLAN Access Control](../09-inter-vlan-access-control/README.md) |
| 10 | [PAT and Internal Web Validation](../10-pat-web-validation/README.md) |
| 11 | [HSRP Gateway Redundancy](../11-hsrp-redundancy/README.md) |
| 12 | [STP and LACP EtherChannel](../12-stp-etherchannel/README.md) |
| 13 | [Centralized Syslog Monitoring](../13-syslog-monitoring/README.md) |
| 14 | [Source-Restricted Switch Management](../14-switch-management-acl/README.md) |
| 15 | [Final Summary](../15-final-summary/README.md) |
