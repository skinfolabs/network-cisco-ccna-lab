# Cisco Enterprise Network Design and Security Lab

![Category](https://img.shields.io/badge/Category-Networking%20%7C%20Cybersecurity-blue)
![Platform](https://img.shields.io/badge/Platform-Cisco%20Packet%20Tracer-lightgrey)
![Focus](https://img.shields.io/badge/Focus-Routing%20%7C%20Switching%20%7C%20Access%20Control-blueviolet)
![Status](https://img.shields.io/badge/Status-Lab%20Validated-brightgreen)

> Project by Samuel Kim. All rights reserved. See [LICENSE](LICENSE).

## Project Overview

This project implements a multi-segment Cisco enterprise network in Packet Tracer. The environment combines routed transit networks, two user VLANs, a server LAN, a wireless client path, management-plane controls, dynamic routing, access control, NAT/PAT, redundancy, switching resiliency, and centralized logging.

The configuration progresses from device naming and management access to VLAN segmentation, router-on-a-stick inter-VLAN routing, DHCP, DNS, WPA2-Personal wireless access, OSPF, SSH, ACL enforcement, PAT, HSRP, EtherChannel, STP, and centralized Syslog. Each service is introduced in dependency order so later security and redundancy controls operate on an established addressing and routing foundation.

The lab demonstrates segmentation, controlled administrative access, unused-port shutdown, sticky Port Security, source-based SSH restrictions, inter-VLAN isolation, encrypted management, gateway redundancy, loop prevention, link aggregation, and event collection. The documented results distinguish configuration evidence from end-to-end validation and retain the limitations of the laboratory tests.

## Network Topology

The network is organized into three operational areas. LAN1 contains the routed user VLANs and wireless infrastructure, LAN2 contains the DNS and internal web services, and LAN3 introduces the redundant gateway and switching path. Routed transit networks connect these areas and provide the foundation for OSPF routing, PAT, inter-VLAN access control, and end-to-end service validation.

Addressing is intentionally visible because DHCP scopes, ACL wildcard masks, PAT source classification, OSPF network advertisements, and HSRP virtual addresses all depend on consistent subnet boundaries. The topology gives the reader the map first, then the later chapters prove that routing, segmentation, security controls, and redundancy work as expected.

> A topology diagram establishes the intended design and traffic relationships. Command output, client tests, browser checks, and protocol state are used later to validate the actual operational behavior.

| Area | Purpose |
|------|---------|
| LAN1 | User VLANs, wireless access, router-on-a-stick gateways, DHCP, and client-side validation |
| LAN2 | Server network containing DNS and internal web services used for application reachability tests |
| LAN3 | Redundant gateway and switching area used for HSRP, STP, and EtherChannel validation |
| Transit links | Routed connectivity between routers and the OSPF path used to move traffic between network areas |
| Internet/PAT edge | Outside-facing path used to demonstrate address translation and outbound connectivity behavior |

![Complete Lab Topology](images/01-overview/01-complete-lab-topology.jpg)

<p><sub><strong>Screenshot 001 - Complete Network Topology:</strong> Packet Tracer topology showing the routed, switched, wireless, server, client, transit, and redundancy segments used throughout the lab.</sub></p>

## Objectives

- Build a routed and switched Cisco topology containing user, server, wireless, and transit networks.
- Segment client traffic with VLANs, 802.1Q trunks, router subinterfaces, and ACLs.
- Automate client addressing through router-based DHCP and publish an internal web service through DNS.
- Replace the initial Telnet workflow with SSH version 2 and source-restricted management access.
- Demonstrate OSPF routing, PAT, HSRP gateway roles, EtherChannel, STP, Port Security, and Syslog.
- Validate permitted and denied behavior using leases, pings, SSH sessions, browser tests, protocol state, and logs.

## Project Chapters

| Chapter | Description |
|---------|-------------|
| [Device Identity and Management Foundation](docs/01-device-identity-management/README.md) | Hostnames, local access, banners, console/VTY baseline, and device setup |
| [VLAN Segmentation and Trunk Hardening](docs/02-vlan-segmentation-trunking/README.md) | VLAN creation, access ports, trunk hardening, and trunk validation |
| [DHCP and Router-on-a-Stick Routing](docs/03-dhcp-router-on-a-stick/README.md) | Router subinterfaces, DHCP pools, switch trunk path, and client leases |
| [Server, DNS, and Wireless Services](docs/04-server-dns-wireless/README.md) | Static servers, DNS publishing, WLAN profile, WPA2 access, and wireless path validation |
| [Access-Layer Port Security](docs/05-port-security/README.md) | Unused-port shutdown, sticky MAC learning, violation mode, and validation limits |
| [OSPF Dynamic Routing](docs/06-ospf-routing/README.md) | Routed transit links, OSPF advertisements, adjacency validation, and LAN3 expansion |
| [SSH Management and Source ACLs](docs/07-ssh-management-acls/README.md) | SSH version 2 configuration, management access, and source-based ACL restriction |
| [Inter-VLAN Access Control](docs/08-inter-vlan-access-control/README.md) | Inter-VLAN isolation policy and validation of blocked and preserved reachability |
| [PAT and Internal Web Validation](docs/09-pat-web-validation/README.md) | PAT configuration on SAM-R2 and client DNS/HTTP validation |
| [HSRP Gateway Redundancy](docs/10-hsrp-redundancy/README.md) | Redundant gateway topology, HSRP active/standby roles, and validation limits |
| [STP and LACP EtherChannel](docs/11-stp-etherchannel/README.md) | STP root control, redundant switching, and LACP EtherChannel configuration |
| [Centralized Syslog Monitoring](docs/12-syslog-monitoring/README.md) | Centralized Syslog destination and event collection validation |
| [Source-Restricted Switch Management](docs/13-switch-management-acl/README.md) | Switch SVI management access and VLAN-based SSH allow/deny validation |
| [Testing, Results, and Recommendations](docs/14-testing-results-summary/README.md) | Confirmed results, skills demonstrated, limitations, and production recommendations |

## Lab Environment Summary

The lab uses Cisco Packet Tracer with ISR4331 routers, Catalyst 2960 switches, a wireless LAN controller, a lightweight access point, two servers, wired PCs, and a wireless laptop. The supplied Packet Tracer file is retained in [configs/ccna-enterprise-network-lab.pkt](configs/ccna-enterprise-network-lab.pkt).

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

> The usernames and passwords are intentionally shown openly because this is an isolated Packet Tracer lab and the visible values make the configuration reproducible for learning and review. They are not real credentials and must not be reused on production devices.

## Tools and Technologies

- Cisco Packet Tracer 8.x
- Cisco IOS CLI
- ISR4331 routers and Catalyst 2960 switches
- VLANs, 802.1Q, DTP controls, router-on-a-stick, and DHCP
- OSPF, ACLs, PAT, HSRP, STP, and LACP EtherChannel
- SSH version 2, Port Security, WPA2-Personal, DNS, HTTP, and Syslog

## Key Outcomes

- Built a routed and switched Cisco topology with user, server, wireless, transit, and redundant LAN segments.
- Configured VLAN segmentation, hardened trunks, router-on-a-stick gateways, and router-based DHCP.
- Published an internal web service through DNS and validated wired and wireless client access.
- Configured OSPF routing, SSH management, source-based management ACLs, inter-VLAN ACLs, and PAT.
- Added Port Security, HSRP, STP root selection, LACP EtherChannel, and centralized Syslog.
- Documented evidence limits where a configuration exists but full operational behavior was not captured.

## Skills Demonstrated

- Cisco IOS device initialization and management-plane configuration
- VLAN design, access-port assignment, 802.1Q trunks, and router-on-a-stick
- DHCP, DNS, wireless access, OSPF, ACLs, PAT, HSRP, STP, and EtherChannel
- SSH hardening, source-restricted management, Port Security, and Syslog monitoring
- Evidence-based validation and production-risk analysis
