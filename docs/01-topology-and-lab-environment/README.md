# Topology and Lab Environment

This chapter defines the Packet Tracer topology, devices, subnets, and services used throughout the project.

## Topology

The network has three areas: LAN1 for user VLANs and wireless access, LAN2 for DNS and web services, and LAN3 for redundancy. Routed transit networks connect these areas for OSPF, PAT, ACLs, and end-to-end validation.

Addressing is intentionally visible because DHCP scopes, ACL wildcard masks, PAT classification, OSPF advertisements, and HSRP virtual IPs depend on consistent subnet boundaries.

> A topology diagram establishes the intended design and traffic relationships. Command output, client tests, browser checks, and protocol state are used later to validate the actual operational behavior.

| Area | Purpose |
|------|---------|
| LAN1 | User VLANs, wireless access, router-on-a-stick gateways, DHCP, and client-side validation |
| LAN2 | Server network containing DNS and internal web services used for application reachability tests |
| LAN3 | Redundant gateway and switching area used for HSRP, STP, and EtherChannel validation |
| Transit links | Routed connectivity between routers and the OSPF path used to move traffic between network areas |
| Internet/PAT edge | Outside-facing path used to demonstrate address translation and outbound connectivity behavior |

![Complete Lab Topology](../../images/01-overview/01-complete-lab-topology.jpg)

<p><sub><strong>Screenshot 001 - Complete Network Topology:</strong> Packet Tracer topology showing the routed, switched, wireless, server, client, transit, and redundancy segments used throughout the lab.</sub></p>

## Lab Environment

The lab uses Cisco Packet Tracer with ISR4331 routers, Catalyst 2960 switches, a wireless LAN controller, a lightweight AP, two servers, wired PCs, and a wireless laptop. The Packet Tracer file is retained in [configs/ccna-enterprise-network-lab.pkt](../../configs/ccna-enterprise-network-lab.pkt).

| Component | Description |
|-----------|-------------|
| Routers | `SAM-R0` through `SAM-R4`; inter-VLAN routing, OSPF, PAT, HSRP, DHCP, and routed transit links |
| Switches | `SAM-S0`, `SAM-S1`, `SAM-S3` through `SAM-S8`; access VLANs, trunks, Port Security, SSH, STP, and EtherChannel |
| VLAN 10 | `192.168.10.0/24`; gateway `192.168.10.1`; permitted administrative source network |
| VLAN 20 | `192.168.20.0/24`; gateway `192.168.20.1`; isolated from VLAN 10 and denied SSH management |
| R0-R1 transit | `172.31.0.0/16`; R0 `172.31.0.1`, R1 `172.31.0.2` |
| R1-R2 transit | `209.165.200.0/24`; R1 `209.165.200.1`, R2 `209.165.200.2` |
| Server LAN | `172.19.0.0/16`; gateway `172.19.0.1`, DNS `172.19.0.100`, web `172.19.0.200` |
| Redundancy transit | `192.168.13.0/24`; R1 `192.168.13.1`, R3 `192.168.13.2`, R4 `192.168.13.3`, HSRP VIP `192.168.13.222` |
| LAN3 | `10.10.10.0/24`; R3 `10.10.10.1`, R4 `10.10.10.2`, HSRP VIP `10.10.10.222` |
| Wireless | `SamNet` using WPA2-Personal; wireless clients join the VLAN 10 addressing domain |

> Usernames and passwords are shown because this is an isolated Packet Tracer lab. They are not real credentials and must not be reused on production devices.

---

## Project Chapters

| # | Chapter |
|---|---------|
| 0 | [Project Overview](../../README.md) |
| 1 | [Topology and Lab Environment](README.md) |
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
