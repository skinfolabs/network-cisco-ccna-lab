# Cisco Enterprise Network Design and Security Lab

![Category](https://img.shields.io/badge/Category-Networking%20%7C%20Cybersecurity-blue)
![Platform](https://img.shields.io/badge/Platform-Cisco%20Packet%20Tracer-lightgrey)
![Focus](https://img.shields.io/badge/Focus-Routing%20%7C%20Switching%20%7C%20Access%20Control-blueviolet)
![Status](https://img.shields.io/badge/Status-Lab%20Validated-brightgreen)

> Project by Samuel Kim. All rights reserved. See [LICENSE](LICENSE).

## Overview

This project implements a multi-segment Cisco enterprise network in Packet Tracer. The environment combines routed transit networks, two user VLANs, a server LAN, a wireless client path, and a redundant LAN3 design into one connected laboratory topology.

The configuration progresses from device naming and management access to VLAN segmentation, router-on-a-stick inter-VLAN routing, DHCP, DNS, WPA2-Personal wireless access, OSPF, SSH, ACL enforcement, PAT, HSRP, EtherChannel, STP, and centralized Syslog. Each service is introduced in dependency order so later security and redundancy controls operate on an established addressing and routing foundation.

From a networking and cybersecurity perspective, the lab demonstrates segmentation, controlled administrative access, unused-port shutdown, sticky Port Security, source-based SSH restrictions, inter-VLAN isolation, encrypted management, gateway redundancy, loop prevention, link aggregation, and event collection. The documented results distinguish configuration evidence from end-to-end validation and retain the limitations of the original laboratory tests.

## Objectives

- Build a routed and switched Cisco topology containing user, server, wireless, and transit networks.
- Segment client traffic with VLANs, 802.1Q trunks, router subinterfaces, and ACLs.
- Automate client addressing through router-based DHCP and publish an internal web service through DNS.
- Replace the initial Telnet workflow with SSH version 2 and source-restricted management access.
- Demonstrate OSPF routing, PAT, HSRP gateway roles, EtherChannel, STP, Port Security, and Syslog.
- Validate permitted and denied behavior using leases, pings, SSH sessions, browser tests, protocol state, and logs.

## Project Roadmap

| Section | What I configured |
|---------|-------------------|
| Network foundation | Device identities, management controls, addressing, and the complete topology |
| Segmentation and services | VLANs, trunks, DHCP, DNS, server addressing, and wireless access |
| Routing and access control | OSPF, SSH, management ACLs, inter-VLAN isolation, and PAT |
| Resilience and monitoring | HSRP, STP, EtherChannel, and centralized Syslog |

## Lab Environment

The lab uses Cisco Packet Tracer with ISR4331 routers, Catalyst 2960 switches, a wireless LAN controller, a lightweight access point, two servers, wired PCs, and a wireless laptop. The supplied Packet Tracer file is retained in [`configs/ccna-enterprise-network-lab.pkt`](configs/ccna-enterprise-network-lab.pkt).

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

> The usernames, passwords, IP addresses, and hostnames shown here are intentional Packet Tracer laboratory values. They must not be reused as production credentials or addressing without a separate security and capacity design.

## Tools and Technologies

- Cisco Packet Tracer 8.x
- Cisco IOS CLI
- ISR4331 routers and Catalyst 2960 switches
- VLANs, 802.1Q, DTP controls, router-on-a-stick, and DHCP
- OSPF, ACLs, PAT, HSRP, STP, and LACP EtherChannel
- SSH version 2, Port Security, WPA2-Personal, DNS, HTTP, and Syslog

## Implementation Walkthrough

The walkthrough follows the dependency order of the working network. Device identities and management controls are established first, Layer 2 and addressing services are added next, dynamic routing connects the sites, security policy limits access, and redundancy and monitoring complete the design. Commands are summarized in the relevant steps and collected in [`docs/commands.md`](docs/commands.md).

---------

## Network Topology and Addressing Plan

The topology is organized into three operational areas. LAN1 contains the routed user VLANs and wireless controller, LAN2 contains DNS and web services, and LAN3 introduces a redundant gateway and switching path. Routed transit networks connect the areas and provide the foundation for OSPF.

Addressing is deliberately visible throughout the project because DHCP scopes, ACL wildcard masks, PAT classification, OSPF advertisements, and HSRP virtual addresses all depend on consistent subnet boundaries.

> A topology diagram establishes intended relationships, not operational state. Later command output and client tests are used to validate the individual routing, access-control, and service paths.

**Implemented controls:**

- Defined separate user, server, transit, wireless, and redundancy segments.
- Assigned explicit gateway, server, and HSRP addresses.
- Identified the traffic paths later controlled by ACLs and PAT.

### Identify the complete network layout

The initial diagram records the router chain, user switches, server LAN, wireless infrastructure, and the future LAN3 area. It provides the reference used to interpret every interface, route, policy, and validation result that follows.

> The topology grows during the HSRP stage, but the original baseline remains useful for understanding which links and devices existed before redundancy was introduced.

![Complete Lab Topology](images/01-network-topology/01-complete-lab-topology.png)

<p><sub><strong>Screenshot 001 - Complete Lab Topology:</strong> Original Packet Tracer topology showing the routed, switched, wireless, server, and client segments used in the project.</sub></p>

---------

## Device Identity and Management Foundation

Each network device receives a predictable hostname and a common baseline configuration. The baseline includes console behavior, an enable secret, an authorization banner, line access, disabled domain lookup, and synchronized logging.

Telnet appears in the initial source configuration as a learning step. It is later replaced with SSH-only VTY access because Telnet exposes credentials and commands in clear text.

> `service password-encryption` provides reversible Cisco Type 7 obfuscation for some stored passwords; it is not a strong password-protection mechanism. Production administration should use strong secrets, AAA, SSH, and centralized identity controls.

**Implemented controls:**

- Assigned `SAM-*` hostnames to routers and switches.
- Applied console, VTY, enable-secret, banner, and logging settings.
- Disabled DNS lookup delays caused by unrecognized CLI input.

### Assign device hostnames

Hostnames identify the device in prompts, logs, SSH keys, and troubleshooting output. The naming pattern separates routers (`SAM-R*`) from switches (`SAM-S*`) and makes multi-device command captures readable.

> Consistent names reduce configuration mistakes when the same command sequence is repeated across many devices.

![Switch Hostname Assignment](images/02-device-foundation/01-switch-hostname-assignment.png)

<p><sub><strong>Screenshot 002 - Switch Hostname Assignment:</strong> Hostname changes applied across the initial Cisco switch set.</sub></p>

![Router Hostname Assignment](images/02-device-foundation/02-router-hostname-assignment.png)

<p><sub><strong>Screenshot 003 - Router Hostname Assignment:</strong> Hostname changes applied to SAM-R0 through SAM-R3.</sub></p>

### Configure initial local access controls

Console and VTY lines receive the laboratory password, privileged access receives an enable secret, and the MOTD banner warns against unauthorized use. `exec-timeout 0 30` closes an idle line after 30 seconds; it does not repair an invalid command.

> The initial `transport input telnet` setting is retained as historical lab evidence. The SSH chapter later replaces it with encrypted management, which is the appropriate operational state.

```cisco
line console 0
 exec-timeout 0 30
 password Samabcd
 login
line vty 0 4
 transport input telnet
 password Samabcd
 login
enable secret Sam1234
service password-encryption
```

![Console Password and Timeout](images/02-device-foundation/03-console-password-and-timeout.png)

<p><sub><strong>Screenshot 004 - Console Password and Timeout:</strong> Console access protected with the lab password and a 30-second idle timeout.</sub></p>

![Initial Telnet VTY Access](images/02-device-foundation/04-initial-telnet-vty-access.png)

<p><sub><strong>Screenshot 005 - Initial Telnet VTY Access:</strong> Initial VTY line configuration using the laboratory Telnet password before the later SSH migration.</sub></p>

![Enable Secret Configuration](images/02-device-foundation/05-enable-secret-configuration.png)

<p><sub><strong>Screenshot 006 - Enable Secret Configuration:</strong> Privileged EXEC access protected with an enable secret.</sub></p>

![Password Obfuscation Command](images/02-device-foundation/06-password-obfuscation-command.png)

<p><sub><strong>Screenshot 007 - Password Obfuscation Command:</strong> Cisco service password-encryption enabled for stored line passwords.</sub></p>

![Authorized Access Banner](images/02-device-foundation/07-authorized-access-banner.png)

<p><sub><strong>Screenshot 008 - Authorized Access Banner:</strong> MOTD banner identifying authorized use and device ownership.</sub></p>

![Synchronous Console Logging](images/02-device-foundation/08-logging-synchronous.png)

<p><sub><strong>Screenshot 009 - Synchronous Console Logging:</strong> Console logging synchronized so system messages do not split command input.</sub></p>

![Domain Lookup Disabled](images/02-device-foundation/09-disable-domain-lookup.png)

<p><sub><strong>Screenshot 010 - Domain Lookup Disabled:</strong> DNS lookup disabled for unrecognized CLI input to prevent command-line delays.</sub></p>

### Apply the baseline to the routers

The same baseline is applied to SAM-R0 through SAM-R3. These captures preserve each device-specific implementation rather than presenting one example as proof for every router.

> Repeated configuration should remain consistent, but each device still needs individual verification because a missing line on one router can interrupt remote administration or logging.

![SAM-R0 Base Configuration](images/02-device-foundation/10-sam-r0-base-configuration.png)

<p><sub><strong>Screenshot 011 - SAM-R0 Base Configuration:</strong> Full baseline console, banner, password, and VTY configuration captured for SAM-R0.</sub></p>

![SAM-R1 Base Configuration](images/02-device-foundation/11-sam-r1-base-configuration.png)

<p><sub><strong>Screenshot 012 - SAM-R1 Base Configuration:</strong> Full baseline console, banner, password, and VTY configuration captured for SAM-R1.</sub></p>

![SAM-R3 Base Configuration](images/02-device-foundation/12-sam-r3-base-configuration.png)

<p><sub><strong>Screenshot 013 - SAM-R3 Base Configuration:</strong> Full baseline console, banner, password, and VTY configuration captured for SAM-R3.</sub></p>

![SAM-R2 Base Configuration](images/02-device-foundation/13-sam-r2-base-configuration.png)

<p><sub><strong>Screenshot 014 - SAM-R2 Base Configuration:</strong> Full baseline console, banner, password, and VTY configuration captured for SAM-R2.</sub></p>

### Apply the baseline to the switches

SAM-S0, SAM-S1, and SAM-S3 through SAM-S7 receive the same management foundation. The resulting device prompts and saved configurations establish the starting state for VLAN, trunk, Port Security, and EtherChannel work.

> Switch management security is independent of data-plane forwarding. A switch can forward frames while still having incomplete or insecure administrative access.

![SAM-S0 Base Configuration](images/02-device-foundation/14-sam-s0-base-configuration.png)

<p><sub><strong>Screenshot 015 - SAM-S0 Base Configuration:</strong> Baseline management configuration captured for SAM-S0.</sub></p>

![SAM-S1 Base Configuration](images/02-device-foundation/15-sam-s1-base-configuration.png)

<p><sub><strong>Screenshot 016 - SAM-S1 Base Configuration:</strong> Baseline management configuration captured for SAM-S1.</sub></p>

![SAM-S3 Base Configuration](images/02-device-foundation/16-sam-s3-base-configuration.png)

<p><sub><strong>Screenshot 017 - SAM-S3 Base Configuration:</strong> Baseline management configuration captured for SAM-S3.</sub></p>

![SAM-S4 Base Configuration](images/02-device-foundation/17-sam-s4-base-configuration.png)

<p><sub><strong>Screenshot 018 - SAM-S4 Base Configuration:</strong> Baseline management configuration captured for SAM-S4.</sub></p>

![SAM-S6 Base Configuration](images/02-device-foundation/18-sam-s6-base-configuration.png)

<p><sub><strong>Screenshot 019 - SAM-S6 Base Configuration:</strong> Baseline management configuration captured for SAM-S6.</sub></p>

![SAM-S5 Base Configuration](images/02-device-foundation/19-sam-s5-base-configuration.png)

<p><sub><strong>Screenshot 020 - SAM-S5 Base Configuration:</strong> Baseline management configuration captured for SAM-S5.</sub></p>

![SAM-S7 Base Configuration](images/02-device-foundation/20-sam-s7-base-configuration.png)

<p><sub><strong>Screenshot 021 - SAM-S7 Base Configuration:</strong> Baseline management configuration captured for SAM-S7.</sub></p>

---------

## VLAN Segmentation and Trunk Hardening

VLAN 10 and VLAN 20 create separate Layer 2 broadcast domains for the user devices. Access ports place endpoints in the correct VLAN, while 802.1Q trunks carry both networks between switches and toward the router-on-a-stick gateway.

VLAN 99 is assigned as the native VLAN on the trunks and DTP negotiation is disabled. This removes reliance on the default native VLAN and prevents links from dynamically negotiating trunk mode.

> VLANs provide segmentation but do not independently enforce routed security. Inter-VLAN traffic is controlled later by router ACLs.

**Implemented controls:**

- Created and named VLAN 10 and VLAN 20.
- Assigned endpoint-facing interfaces as access ports.
- Configured static 802.1Q trunks with native VLAN 99 and `nonegotiate`.

### Create the user VLANs

The VLAN databases on SAM-S0, SAM-S1, and SAM-S3 are populated with VLAN 10 (`YELLOW`) and VLAN 20 (`BLUE`). Matching VLAN IDs are required on every switch that must transport those frames.

> A VLAN name is operational documentation; the numeric VLAN ID is the value carried in the 802.1Q tag.

![SAM-S3 VLAN Database](images/03-vlan-segmentation/01-sam-s3-vlan-database.png)

<p><sub><strong>Screenshot 022 - SAM-S3 VLAN Database:</strong> VLAN 10 named YELLOW and VLAN 20 named BLUE on SAM-S3.</sub></p>

![SAM-S1 VLAN Database](images/03-vlan-segmentation/02-sam-s1-vlan-database.png)

<p><sub><strong>Screenshot 023 - SAM-S1 VLAN Database:</strong> VLAN 10 and VLAN 20 created and named on SAM-S1.</sub></p>

![SAM-S0 VLAN Database](images/03-vlan-segmentation/03-sam-s0-vlan-database.png)

<p><sub><strong>Screenshot 024 - SAM-S0 VLAN Database:</strong> VLAN 10 and VLAN 20 created and named on SAM-S0.</sub></p>

### Assign access ports to VLAN 10 and VLAN 20

Endpoint-facing FastEthernet ports are forced into access mode and assigned to their intended user VLAN. This prevents attached clients from negotiating a trunk and defines which broadcast domain each client joins.

> An incorrect access VLAN can place a client in the wrong IP subnet, bypass the intended ACL source classification, or prevent DHCP from reaching the correct pool.

![SAM-S0 VLAN 20 Access Port](images/03-vlan-segmentation/04-sam-s0-vlan20-access-port.png)

<p><sub><strong>Screenshot 025 - SAM-S0 VLAN 20 Access Port:</strong> FastEthernet0/5 assigned as an access port in VLAN 20.</sub></p>

![SAM-S0 VLAN 10 Access Range](images/03-vlan-segmentation/05-sam-s0-vlan10-access-range.png)

<p><sub><strong>Screenshot 026 - SAM-S0 VLAN 10 Access Range:</strong> FastEthernet0/1-2 assigned to VLAN 10.</sub></p>

![SAM-S1 VLAN 10 Access Port](images/03-vlan-segmentation/06-sam-s1-vlan10-access-port.png)

<p><sub><strong>Screenshot 027 - SAM-S1 VLAN 10 Access Port:</strong> FastEthernet0/5 assigned as an access port in VLAN 10.</sub></p>

![SAM-S1 VLAN 20 Access Range](images/03-vlan-segmentation/07-sam-s1-vlan20-access-range.png)

<p><sub><strong>Screenshot 028 - SAM-S1 VLAN 20 Access Range:</strong> SAM-S1 access interfaces assigned to VLAN 20.</sub></p>

### Configure and validate hardened trunks

The switch uplinks are placed in trunk mode, VLAN 99 is selected as the native VLAN, and DTP is disabled. `show interfaces trunk` then confirms 802.1Q status and native VLAN consistency across the participating switches.

> A native-VLAN mismatch can leak untagged traffic into the wrong VLAN and produce difficult Layer 2 troubleshooting symptoms.

```cisco
switchport mode trunk
switchport trunk native vlan 99
switchport nonegotiate
```

![SAM-S1 Native VLAN 99](images/03-vlan-segmentation/08-sam-s1-native-vlan99.png)

<p><sub><strong>Screenshot 029 - SAM-S1 Native VLAN 99:</strong> VLAN 99 created and named Native on SAM-S1.</sub></p>

![SAM-S1 Secured Trunk](images/03-vlan-segmentation/09-sam-s1-secured-trunk.png)

<p><sub><strong>Screenshot 030 - SAM-S1 Secured Trunk:</strong> SAM-S1 trunk configured with native VLAN 99 and DTP disabled.</sub></p>

![SAM-S0 Secured Trunk](images/03-vlan-segmentation/10-sam-s0-secured-trunk.png)

<p><sub><strong>Screenshot 031 - SAM-S0 Secured Trunk:</strong> SAM-S0 trunk configured with native VLAN 99 and DTP disabled.</sub></p>

![SAM-S0 Trunk Validation](images/03-vlan-segmentation/11-sam-s0-trunk-validation.png)

<p><sub><strong>Screenshot 032 - SAM-S0 Trunk Validation:</strong> SAM-S0 trunk table showing 802.1Q operation and native VLAN 99.</sub></p>

![SAM-S1 Trunk Validation](images/03-vlan-segmentation/12-sam-s1-trunk-validation.png)

<p><sub><strong>Screenshot 033 - SAM-S1 Trunk Validation:</strong> SAM-S1 trunk table showing both links using native VLAN 99.</sub></p>

![SAM-S3 Trunk Validation](images/03-vlan-segmentation/13-sam-s3-trunk-validation.png)

<p><sub><strong>Screenshot 034 - SAM-S3 Trunk Validation:</strong> SAM-S3 trunk table showing Fa0/24 and uplinks operating with native VLAN 99.</sub></p>

---------

## DHCP and Router-on-a-Stick Routing

SAM-R0 acts as both the default gateway and DHCP server for VLAN 10 and VLAN 20. Two 802.1Q subinterfaces give the router one Layer 3 address in each VLAN, while separate DHCP pools assign clients the matching gateway and DNS server.

The switch port toward SAM-R0 is configured as a trunk so tagged frames from both user VLANs can reach the correct router subinterface.

> Router-on-a-stick is efficient for a lab and small environment, but all inter-VLAN traffic shares one physical router link. Larger designs commonly use multilayer switching for greater throughput and redundancy.

**Implemented controls:**

- Created DHCP pools for both user VLANs.
- Excluded the router interface addresses from dynamic allocation.
- Configured 802.1Q gateway subinterfaces and validated six client leases.

### Configure the DHCP pools and gateway subinterfaces

VLAN 10 uses `192.168.10.0/24` with gateway `192.168.10.1`, while VLAN 20 uses `192.168.20.0/24` with gateway `192.168.20.1`. Both pools distribute `172.19.0.100` as the DNS server.

> DHCP options must match the routed interface for the client VLAN. A lease can be assigned successfully while still producing unusable connectivity if the gateway or DNS option is incorrect.

![DHCP Pool Declaration](images/04-dhcp-router-on-a-stick/01-dhcp-pool-declaration.png)

<p><sub><strong>Screenshot 035 - DHCP Pool Declaration:</strong> Router-based DHCP pools declared for VLAN 10 and VLAN 20.</sub></p>

![VLAN 20 Pool and Options](images/04-dhcp-router-on-a-stick/02-vlan20-pool-and-options.png)

<p><sub><strong>Screenshot 036 - VLAN 20 Pool and Options:</strong> VLAN 20 scope combined with gateway and DNS option planning.</sub></p>

![VLAN 10 DHCP Options](images/04-dhcp-router-on-a-stick/03-vlan10-dhcp-options.png)

<p><sub><strong>Screenshot 037 - VLAN 10 DHCP Options:</strong> VLAN 10 network, gateway, DNS server, and excluded gateway address configured.</sub></p>

![VLAN 20 DHCP Options](images/04-dhcp-router-on-a-stick/04-vlan20-dhcp-options.png)

<p><sub><strong>Screenshot 038 - VLAN 20 DHCP Options:</strong> VLAN 20 network, gateway, DNS server, and excluded gateway address configured.</sub></p>

![VLAN 20 Router Subinterface](images/04-dhcp-router-on-a-stick/05-vlan20-router-subinterface.png)

<p><sub><strong>Screenshot 039 - VLAN 20 Router Subinterface:</strong> 802.1Q subinterface GigabitEthernet0/0/0.20 configured with the VLAN 20 gateway.</sub></p>

![VLAN 10 Router Subinterface](images/04-dhcp-router-on-a-stick/06-vlan10-router-subinterface.png)

<p><sub><strong>Screenshot 040 - VLAN 10 Router Subinterface:</strong> 802.1Q subinterface GigabitEthernet0/0/0.10 configured with the VLAN 10 gateway.</sub></p>

### Connect the router through the switch trunk

SAM-S3 FastEthernet0/24 carries the tagged VLAN traffic to SAM-R0. The consolidated command captures show the trunk, subinterface, scope, gateway, DNS, and excluded-address settings together.

> The physical router interface remains shared, but each subinterface processes only frames carrying its configured 802.1Q VLAN tag.

![SAM-S3 Router Trunk](images/04-dhcp-router-on-a-stick/07-sam-s3-router-trunk.png)

<p><sub><strong>Screenshot 041 - SAM-S3 Router Trunk:</strong> FastEthernet0/24 configured as the trunk toward the router-on-a-stick gateway.</sub></p>

![VLAN 20 DHCP Configuration](images/04-dhcp-router-on-a-stick/08-vlan20-dhcp-complete-config.png)

<p><sub><strong>Screenshot 042 - VLAN 20 DHCP Configuration:</strong> Complete VLAN 20 subinterface and DHCP scope sequence.</sub></p>

![VLAN 10 DHCP Configuration](images/04-dhcp-router-on-a-stick/09-vlan10-dhcp-complete-config.png)

<p><sub><strong>Screenshot 043 - VLAN 10 DHCP Configuration:</strong> Complete VLAN 10 subinterface and DHCP scope sequence.</sub></p>

### Validate client address allocation

PC0, PC1, and PC5 receive VLAN 10 addresses; PC2, PC3, and PC4 receive VLAN 20 addresses. Every lease shows the expected `/24` mask, local gateway, and DNS server.

> A DHCP lease validates scope delivery. Routing and policy behavior are verified separately with the later ping, SSH, and browser tests.

![PC0 DHCP Lease](images/04-dhcp-router-on-a-stick/10-pc0-dhcp-lease.png)

<p><sub><strong>Screenshot 044 - PC0 DHCP Lease:</strong> PC0 receives 192.168.10.2 with the VLAN 10 gateway and laboratory DNS server.</sub></p>

![PC1 DHCP Lease](images/04-dhcp-router-on-a-stick/11-pc1-dhcp-lease.png)

<p><sub><strong>Screenshot 045 - PC1 DHCP Lease:</strong> PC1 receives 192.168.10.3 from the VLAN 10 pool.</sub></p>

![PC2 DHCP Lease](images/04-dhcp-router-on-a-stick/12-pc2-dhcp-lease.png)

<p><sub><strong>Screenshot 046 - PC2 DHCP Lease:</strong> PC2 receives 192.168.20.2 from the VLAN 20 pool.</sub></p>

![PC3 DHCP Lease](images/04-dhcp-router-on-a-stick/13-pc3-dhcp-lease.png)

<p><sub><strong>Screenshot 047 - PC3 DHCP Lease:</strong> PC3 receives 192.168.20.3 from the VLAN 20 pool.</sub></p>

![PC5 DHCP Lease](images/04-dhcp-router-on-a-stick/14-pc5-dhcp-lease.png)

<p><sub><strong>Screenshot 048 - PC5 DHCP Lease:</strong> PC5 receives 192.168.10.4 from the VLAN 10 pool.</sub></p>

![PC4 DHCP Lease](images/04-dhcp-router-on-a-stick/15-pc4-dhcp-lease.png)

<p><sub><strong>Screenshot 049 - PC4 DHCP Lease:</strong> PC4 receives 192.168.20.4 from the VLAN 20 pool.</sub></p>

---------

## Server, DNS, and Wireless Services

LAN2 hosts the DNS and web servers on static addresses so clients and network services can depend on stable endpoints. DNS maps `www.sam.com` to the web server, while the wireless controller and lightweight access point extend VLAN 10 connectivity to a laptop.

The wireless workflow uses WPA2-Personal in the isolated lab and confirms that the laptop receives the same gateway and DNS information as wired VLAN 10 clients.

> A production wireless network should normally use WPA2/WPA3-Enterprise with centralized authentication, separate infrastructure management, and stronger key lifecycle controls than a shared PSK.

**Implemented controls:**

- Assigned static addressing to DNS and web services.
- Published the web server through an internal DNS A record.
- Built and validated the `SamNet` wireless client path.

### Configure static server addressing

The DNS server uses `172.19.0.100/16` and the web server uses `172.19.0.200/16`; both use `172.19.0.1` as their gateway. The web server points to the DNS server so the same resolver can be used for service testing.

> Static service addresses prevent a DNS record or logging destination from becoming stale after a lease change.

![DNS Server Static Address](images/05-server-dns-wireless/01-dns-server-static-address.png)

<p><sub><strong>Screenshot 050 - DNS Server Static Address:</strong> DNS server configured as 172.19.0.100/16 with gateway 172.19.0.1.</sub></p>

![Web Server Static Address](images/05-server-dns-wireless/02-web-server-static-address.png)

<p><sub><strong>Screenshot 051 - Web Server Static Address:</strong> Web server configured as 172.19.0.200/16 and pointed to the laboratory DNS server.</sub></p>

### Publish the internal web name

Packet Tracer DNS is enabled and an A record maps `www.sam.com` directly to `172.19.0.200`. An A record stores an IPv4 address; it is not a CNAME alias.

> DNS resolution proves name-to-address mapping. The browser tests later confirm that the HTTP service is reachable at the resolved address.

![Web DNS A Record](images/05-server-dns-wireless/03-web-dns-a-record.png)

<p><sub><strong>Screenshot 052 - Web DNS A Record:</strong> A record maps www.sam.com to 172.19.0.200.</sub></p>

### Configure the WLAN profile and obtain a lease

The `SamNet` WLAN uses WPA2-Personal and places the wireless client in the VLAN 10 addressing domain. The laptop receives `192.168.10.5`, gateway `192.168.10.1`, and DNS server `172.19.0.100`.

> The controller profile defines wireless authentication and client forwarding; the wired trunk and DHCP configuration still determine whether the client can reach the routed network.

![SamNet WLAN Profile](images/05-server-dns-wireless/04-samnet-wlan-profile.png)

<p><sub><strong>Screenshot 053 - SamNet WLAN Profile:</strong> Wireless profile configured for SamNet with WPA2-Personal and the management setting shown in Packet Tracer.</sub></p>

![Wireless Client DHCP Lease](images/05-server-dns-wireless/05-wireless-client-dhcp-lease.png)

<p><sub><strong>Screenshot 054 - Wireless Client DHCP Lease:</strong> Laptop receives 192.168.10.5, the VLAN 10 gateway, and DNS settings over WLAN.</sub></p>

### Build and validate the wireless client path

FlexConnect options are enabled, a compatible wireless module is installed in the laptop, and the client discovers and joins SamNet through the lightweight access point. The access-point infrastructure address is excluded from the DHCP pool, and the switch links toward the access point and controller are configured as trunks.

> The visible pre-shared key is retained as laboratory data only. A real key must not be committed to a public repository.

![FlexConnect Settings](images/05-server-dns-wireless/06-flexconnect-settings.png)

<p><sub><strong>Screenshot 055 - FlexConnect Settings:</strong> Local switching and local authentication enabled for the lightweight access-point workflow.</sub></p>

![Laptop Wireless Module](images/05-server-dns-wireless/07-laptop-wireless-module.png)

<p><sub><strong>Screenshot 056 - Laptop Wireless Module:</strong> Compatible wireless module installed in the Packet Tracer laptop.</sub></p>

![Wireless Client and Access Point](images/05-server-dns-wireless/08-wireless-client-and-ap.png)

<p><sub><strong>Screenshot 057 - Wireless Client and Access Point:</strong> Laptop associated through the lightweight access point.</sub></p>

![SamNet Discovery](images/05-server-dns-wireless/09-samnet-discovery.png)

<p><sub><strong>Screenshot 058 - SamNet Discovery:</strong> Laptop detects the SamNet WLAN with WPA2-PSK security.</sub></p>

![SamNet WPA2 Authentication](images/05-server-dns-wireless/10-samnet-wpa2-authentication.png)

<p><sub><strong>Screenshot 059 - SamNet WPA2 Authentication:</strong> Laboratory pre-shared key entered to join the wireless network.</sub></p>

![Access Point Address Reservation](images/05-server-dns-wireless/11-access-point-address-reservation.png)

<p><sub><strong>Screenshot 060 - Access Point Address Reservation:</strong> 192.168.10.129 excluded from the DHCP pool for infrastructure use.</sub></p>

![Access Point Switch Trunk](images/05-server-dns-wireless/12-access-point-switch-trunk.png)

<p><sub><strong>Screenshot 061 - Access Point Switch Trunk:</strong> SAM-S0 port toward the access point configured as a trunk with native VLAN 99.</sub></p>

![Controller Switch Trunk](images/05-server-dns-wireless/13-controller-switch-trunk.png)

<p><sub><strong>Screenshot 062 - Controller Switch Trunk:</strong> SAM-S1 port toward the wireless controller configured as a trunk with native VLAN 99.</sub></p>

---------

## Access-Layer Port Security

SAM-S4 represents the server-room access switch. Unused interfaces are administratively disabled, and active access ports use sticky MAC learning with a one-address limit and restrict violation mode.

Restrict mode drops frames from unauthorized MAC addresses and increments the violation counter without shutting down the entire interface. This makes the violation observable while preserving access for the learned endpoint.

> Port Security limits attachment at one switch port; it does not authenticate a user and should complement 802.1X, physical security, and monitoring in production.

**Implemented controls:**

- Shut down unused access and uplink interfaces.
- Enabled one-address sticky Port Security on the used access ports.
- Recorded the configured restrict action and violation count.

### Disable unused interfaces and enable sticky learning

Unused SAM-S4 ports are shut down before the three active interfaces are placed in access mode and protected. Sticky learning stores the first observed source MAC as the permitted address for that port.

> An overly broad interface range can disable legitimate uplinks. Interface selections must be checked against the physical topology before applying a shutdown command.

![Unused Switch Ports Disabled](images/06-port-security/01-disable-unused-switch-ports.png)

<p><sub><strong>Screenshot 063 - Unused Switch Ports Disabled:</strong> Unused SAM-S4 interfaces shut down to reduce unauthorized physical access.</sub></p>

![Sticky Port Security](images/06-port-security/02-sticky-port-security.png)

<p><sub><strong>Screenshot 064 - Sticky Port Security:</strong> Access ports limited to one learned MAC address with restrict violation mode.</sub></p>

### Review Port Security state and limitations

`show port-security` reports the protected interfaces, one learned address per interface, restrict mode, and a violation count of eight on one port. The retained ping uses `172.168.0.1`, which is outside the documented topology, so that timeout is not treated as proof of enforcement.

> The violation counter is the relevant control evidence. A generic timeout can also result from an incorrect address, missing route, or unavailable endpoint.

![Port Security Status](images/06-port-security/03-port-security-status.png)

<p><sub><strong>Screenshot 065 - Port Security Status:</strong> show port-security output lists secured interfaces and restrict actions.</sub></p>

![Initial Port Security Ping Test](images/06-port-security/04-initial-port-security-ping.png)

<p><sub><strong>Screenshot 066 - Initial Port Security Ping Test:</strong> The original test targets 172.168.0.1; this address is outside the documented topology and is retained as limited lab evidence.</sub></p>

![Port Security Violation Counter](images/06-port-security/05-port-security-violation-counter.png)

<p><sub><strong>Screenshot 067 - Port Security Violation Counter:</strong> SecurityViolation count records eight events on one protected access port.</sub></p>

![Server Room Access Topology](images/06-port-security/06-server-room-access-topology.png)

<p><sub><strong>Screenshot 068 - Server Room Access Topology:</strong> Laptop and server connections shown around the protected SAM-S4 access switch.</sub></p>

![Restrict Violation Mode](images/06-port-security/07-restrict-violation-mode.png)

<p><sub><strong>Screenshot 069 - Restrict Violation Mode:</strong> Port-security restrict mode limits unauthorized frames without err-disabling the interface.</sub></p>

![Routed Core Topology](images/06-port-security/08-routed-core-topology.png)

<p><sub><strong>Screenshot 070 - Routed Core Topology:</strong> Transit networks between SAM-R0, SAM-R1, and SAM-R2 before OSPF activation.</sub></p>

---------

## OSPF Dynamic Routing

OSPF area 0 connects the user VLANs, routed transit links, server LAN, and later LAN3 networks. The routers advertise their directly connected prefixes and form FULL neighbor adjacencies over the transit segments.

The initial three-router design is expanded through the `192.168.13.0/24` link to SAM-R3, which also advertises `10.10.10.0/24`. This prepares the network for HSRP and the redundant LAN3 path.

> OSPF router IDs can appear as an address from another interface, so a neighbor ID does not have to match the connected-link address. Production OSPF should also use explicit passive interfaces and authentication where supported.

**Implemented controls:**

- Addressed the routed transit interfaces.
- Advertised all documented networks in OSPF area 0.
- Verified FULL adjacency and server-LAN reachability.

### Address the core transit links

The `172.31.0.0/16` and `209.165.200.0/24` networks connect SAM-R0, SAM-R1, and SAM-R2. Each side receives a unique address before OSPF is enabled.

> Dynamic routing cannot repair an incorrect connected-link mask. Both ends must agree on the subnet before a neighbor relationship can form.

![SAM-R1 209.165.200.0 Link](images/07-ospf-routing/01-sam-r1-west-link.png)

<p><sub><strong>Screenshot 071 - SAM-R1 209.165.200.0 Link:</strong> SAM-R1 GigabitEthernet0/0/1 configured as 209.165.200.1/24.</sub></p>

![SAM-R0 Transit Link](images/07-ospf-routing/02-sam-r0-transit-link.png)

<p><sub><strong>Screenshot 072 - SAM-R0 Transit Link:</strong> SAM-R0 GigabitEthernet0/0/1 configured as 172.31.0.1/16.</sub></p>

![SAM-R2 209.165.200.0 Link](images/07-ospf-routing/03-sam-r2-west-link.png)

<p><sub><strong>Screenshot 073 - SAM-R2 209.165.200.0 Link:</strong> SAM-R2 GigabitEthernet0/0/0 configured as 209.165.200.2/24.</sub></p>

![SAM-R1 172.31.0.0 Link](images/07-ospf-routing/04-sam-r1-east-link.png)

<p><sub><strong>Screenshot 074 - SAM-R1 172.31.0.0 Link:</strong> SAM-R1 GigabitEthernet0/0/0 configured as 172.31.0.2/16.</sub></p>

### Advertise the initial networks and validate adjacency

SAM-R0 advertises both user VLANs and the R0-R1 transit, SAM-R1 advertises both transit networks, and SAM-R2 advertises the R1-R2 transit and server LAN. FULL adjacency messages confirm neighbor formation, while the DNS-server ping confirms routed reachability after an initial timeout.

> The source uses `passive-interface g0/0/0` on a router with subinterfaces. In production, the exact user-facing subinterfaces should be made passive explicitly, or `passive-interface default` should be combined with selected `no passive-interface` transit links.

![SAM-R0 OSPF Networks](images/07-ospf-routing/05-sam-r0-ospf-networks.png)

<p><sub><strong>Screenshot 075 - SAM-R0 OSPF Networks:</strong> VLAN and transit networks advertised in OSPF area 0 with a passive-interface command.</sub></p>

![SAM-R1 OSPF Adjacency](images/07-ospf-routing/06-sam-r1-ospf-adjacency.png)

<p><sub><strong>Screenshot 076 - SAM-R1 OSPF Adjacency:</strong> SAM-R1 reaches FULL adjacency and advertises the 209.165.200.0/24 network.</sub></p>

![SAM-R2 OSPF Adjacency](images/07-ospf-routing/07-sam-r2-ospf-adjacency.png)

<p><sub><strong>Screenshot 077 - SAM-R2 OSPF Adjacency:</strong> SAM-R2 reaches FULL adjacency and advertises the server LAN.</sub></p>

![DNS Server Ping](images/07-ospf-routing/08-dns-server-ping.png)

<p><sub><strong>Screenshot 078 - DNS Server Ping:</strong> Initial DNS-server ping receives three of four replies, demonstrating reachability after the first timeout.</sub></p>

### Extend OSPF to SAM-R3 and LAN3

SAM-R1 and SAM-R3 receive addresses in `192.168.13.0/24`, and SAM-R3 also receives `10.10.10.1/24` for LAN3. Both networks are then advertised in area 0; the incomplete first OSPF entry is retained as an intermediate command attempt followed by the completed syntax.

> Error output is useful evidence when the corrected command is shown immediately afterward. It demonstrates the final accepted syntax without hiding the troubleshooting path.

![SAM-R1 to SAM-R3 Expansion](images/07-ospf-routing/09-sam-r1-r3-expansion-topology.png)

<p><sub><strong>Screenshot 079 - SAM-R1 to SAM-R3 Expansion:</strong> New 192.168.13.0/24 link and LAN3 path added to the routed topology.</sub></p>

![SAM-R1 R3-Facing Interface](images/07-ospf-routing/10-sam-r1-r3-interface.png)

<p><sub><strong>Screenshot 080 - SAM-R1 R3-Facing Interface:</strong> SAM-R1 GigabitEthernet0/0/2 configured as 192.168.13.1/24.</sub></p>

![SAM-R3 Interface Addressing](images/07-ospf-routing/11-sam-r3-interface-addressing.png)

<p><sub><strong>Screenshot 081 - SAM-R3 Interface Addressing:</strong> SAM-R3 configured with 192.168.13.2/24 and 10.10.10.1/24.</sub></p>

![SAM-R1 New OSPF Network](images/07-ospf-routing/12-sam-r1-new-ospf-network.png)

<p><sub><strong>Screenshot 082 - SAM-R1 New OSPF Network:</strong> 192.168.13.0/24 added to OSPF after an incomplete first entry.</sub></p>

![SAM-R3 OSPF Configuration](images/07-ospf-routing/13-sam-r3-ospf-configuration.png)

<p><sub><strong>Screenshot 083 - SAM-R3 OSPF Configuration:</strong> SAM-R3 advertises 192.168.13.0/24 and 10.10.10.0/24 in area 0.</sub></p>

---------

## SSH Management and Source ACLs

The management plane is migrated from Telnet to SSH version 2. RSA keys and a local user enable encrypted login, while the VTY lines accept only SSH and authenticate against the local username database.

An extended ACL on SAM-R0 then permits SSH sourced from VLAN 10 and denies SSH sourced from VLAN 20 while allowing other IP traffic. This separates management authorization from general network connectivity.

> The original devices use 1024-bit RSA keys, and two later devices use 2024-bit keys. These values are preserved as laboratory evidence; current production baselines should use a supported key size of at least 2048 bits and centralized AAA.

**Implemented controls:**

- Enabled SSH version 2 and local authentication.
- Replaced Telnet transport with SSH-only VTY access.
- Restricted SSH by source VLAN through ACL 101.

### Enable SSH and validate encrypted sessions

The devices generate RSA keys, create the local user `Sam`, and configure VTY lines with `transport input ssh` and `login local`. Successful client-to-router and router-to-router sessions show the authorization banner and target prompt.

> `ip domain-name SSH` satisfies the Packet Tracer key-generation prerequisite but is not a production DNS domain design. Real devices should use an organization-controlled domain name.

![RSA Key Generation](images/08-ssh-management/01-rsa-key-generation.png)

<p><sub><strong>Screenshot 084 - RSA Key Generation:</strong> A 1024-bit RSA key is generated in the original SSH laboratory configuration.</sub></p>

![SSH-Only VTY Lines](images/08-ssh-management/02-ssh-only-vty-lines.png)

<p><sub><strong>Screenshot 085 - SSH-Only VTY Lines:</strong> VTY transport changed to SSH with local authentication and SSH version 2.</sub></p>

![Router-to-Router SSH](images/08-ssh-management/03-router-to-router-ssh.png)

<p><sub><strong>Screenshot 086 - Router-to-Router SSH:</strong> SAM-R1 successfully opens an SSH session to SAM-R2 at 172.19.0.1.</sub></p>

![Client-to-Router SSH](images/08-ssh-management/04-client-to-router-ssh.png)

<p><sub><strong>Screenshot 087 - Client-to-Router SSH:</strong> A Packet Tracer client successfully opens an SSH session to SAM-R1 at 209.165.200.1.</sub></p>

### Apply SSH to the complete device set

The SSH sequence is repeated across SAM-R0 through SAM-R3 and SAM-S0, SAM-S1, SAM-S3 through SAM-S7. Individual captures preserve the device-specific implementation and show that Telnet is replaced by SSH-only VTY transport.

> Local accounts are practical for a lab but difficult to rotate consistently at scale. Centralized TACACS+ or RADIUS is preferable for production accountability and revocation.

![SAM-R0 SSH Configuration](images/08-ssh-management/05-sam-r0-ssh-config.png)

<p><sub><strong>Screenshot 088 - SAM-R0 SSH Configuration:</strong> RSA key, local user, SSH-only VTY lines, and SSH version 2 configured on SAM-R0.</sub></p>

![SAM-R1 SSH Configuration](images/08-ssh-management/06-sam-r1-ssh-config.png)

<p><sub><strong>Screenshot 089 - SAM-R1 SSH Configuration:</strong> RSA key, local user, SSH-only VTY lines, and SSH version 2 configured on SAM-R1.</sub></p>

![SAM-R3 SSH Configuration](images/08-ssh-management/07-sam-r3-ssh-config.png)

<p><sub><strong>Screenshot 090 - SAM-R3 SSH Configuration:</strong> RSA key, local user, SSH-only VTY lines, and SSH version 2 configured on SAM-R3.</sub></p>

![SAM-R2 SSH Configuration](images/08-ssh-management/08-sam-r2-ssh-config.png)

<p><sub><strong>Screenshot 091 - SAM-R2 SSH Configuration:</strong> RSA key, local user, SSH-only VTY lines, and SSH version 2 configured on SAM-R2.</sub></p>

![SAM-S1 SSH Configuration](images/08-ssh-management/09-sam-s1-ssh-config.png)

<p><sub><strong>Screenshot 092 - SAM-S1 SSH Configuration:</strong> SSH configuration applied to SAM-S1.</sub></p>

![SAM-S0 SSH Configuration](images/08-ssh-management/10-sam-s0-ssh-config.png)

<p><sub><strong>Screenshot 093 - SAM-S0 SSH Configuration:</strong> SSH configuration applied to SAM-S0.</sub></p>

![SAM-S4 SSH Configuration](images/08-ssh-management/11-sam-s4-ssh-config.png)

<p><sub><strong>Screenshot 094 - SAM-S4 SSH Configuration:</strong> SSH configuration applied to SAM-S4.</sub></p>

![SAM-S3 SSH Configuration](images/08-ssh-management/12-sam-s3-ssh-config.png)

<p><sub><strong>Screenshot 095 - SAM-S3 SSH Configuration:</strong> SSH configuration applied to SAM-S3.</sub></p>

![SAM-S6 SSH Configuration](images/08-ssh-management/13-sam-s6-ssh-config.png)

<p><sub><strong>Screenshot 096 - SAM-S6 SSH Configuration:</strong> SSH configuration applied to SAM-S6.</sub></p>

![SAM-S5 SSH Configuration](images/08-ssh-management/14-sam-s5-ssh-config.png)

<p><sub><strong>Screenshot 097 - SAM-S5 SSH Configuration:</strong> SSH configuration applied to SAM-S5.</sub></p>

![SAM-S7 SSH Configuration](images/08-ssh-management/15-sam-s7-ssh-config.png)

<p><sub><strong>Screenshot 098 - SAM-S7 SSH Configuration:</strong> SSH configuration applied to SAM-S7.</sub></p>

### Restrict SSH by source subnet

ACL 101 denies TCP destination port 22 from `192.168.20.0/24`, permits it from `192.168.10.0/24`, and then permits all remaining IP traffic. The ACL is applied inbound on both router subinterfaces so the source policy is evaluated as traffic enters SAM-R0.

> The final `permit ip any any` means this ACL is an SSH-management filter, not a general firewall policy. PC0 succeeds from VLAN 10, while PC2 and PC3 time out from VLAN 20.

![SSH Source ACL 101](images/08-ssh-management/16-ssh-source-acl101.png)

<p><sub><strong>Screenshot 099 - SSH Source ACL 101:</strong> Extended ACL 101 denies VLAN 20 SSH, permits VLAN 10 SSH, and allows other IP traffic.</sub></p>

![ACL 101 Final Permit](images/08-ssh-management/17-acl101-final-permit.png)

<p><sub><strong>Screenshot 100 - ACL 101 Final Permit:</strong> Explicit permit ip any any preserves non-SSH traffic after the targeted SSH rules.</sub></p>

![PC2 SSH Denied](images/08-ssh-management/18-pc2-ssh-denied.png)

<p><sub><strong>Screenshot 101 - PC2 SSH Denied:</strong> VLAN 20 client cannot open SSH to the VLAN 10 router interface.</sub></p>

![PC3 SSH Denied](images/08-ssh-management/19-pc3-ssh-denied.png)

<p><sub><strong>Screenshot 102 - PC3 SSH Denied:</strong> Second VLAN 20 client receives a timeout when testing SSH access.</sub></p>

![PC0 SSH Allowed](images/08-ssh-management/20-pc0-ssh-allowed.png)

<p><sub><strong>Screenshot 103 - PC0 SSH Allowed:</strong> VLAN 10 client successfully authenticates to SAM-R0 over SSH.</sub></p>

![SSH ACL Client Topology](images/08-ssh-management/21-ssh-acl-client-topology.png)

<p><sub><strong>Screenshot 104 - SSH ACL Client Topology:</strong> VLAN 10 and VLAN 20 clients used for allowed and denied management tests.</sub></p>

---------

## Inter-VLAN Access Control

Two standard ACLs isolate VLAN 10 and VLAN 20 from each other while preserving access to other routed networks. Because a standard ACL matches only source IPv4 addresses, each list is placed outbound on the destination VLAN subinterface.

The tests show reciprocal blocking between the two user VLANs and continued reachability to local gateways and the LAN3 router address.

> Standard ACL placement matters: placing the same source-only rule too close to the source can unintentionally block that network from every destination.

**Implemented controls:**

- Blocked VLAN 10 sources from exiting toward VLAN 20.
- Blocked VLAN 20 sources from exiting toward VLAN 10.
- Preserved access to destinations outside the isolated VLAN pair.

### Map the segmentation test topology

The topology identifies the clients in both VLANs and the SAM-R0 subinterfaces where the outbound filters are applied.

> The ACLs operate at Layer 3; devices in the same VLAN still communicate through Layer 2 switching unless another control is introduced.

![Inter-VLAN ACL Topology](images/09-inter-vlan-acls/01-inter-vlan-acl-topology.png)

<p><sub><strong>Screenshot 105 - Inter-VLAN ACL Topology:</strong> Client VLANs and routed gateway used for the segmentation test.</sub></p>

### Apply reciprocal ACLs and validate behavior

ACL 20 denies the VLAN 10 source range before traffic leaves toward VLAN 20, and ACL 10 denies the VLAN 20 source range before traffic leaves toward VLAN 10. Destination-unreachable responses confirm the blocked paths, while successful pings confirm that unrelated routes remain available.

> The gateway-generated unreachable response demonstrates a routed policy decision more clearly than a silent timeout because it identifies the filtering router as the responding device.

![Standard Inter-VLAN ACLs](images/09-inter-vlan-acls/02-standard-inter-vlan-acls.png)

<p><sub><strong>Screenshot 106 - Standard Inter-VLAN ACLs:</strong> Reciprocal standard ACLs block traffic between VLAN 10 and VLAN 20 while permitting other sources.</sub></p>

![VLAN 10 to VLAN 20 Blocked](images/09-inter-vlan-acls/03-vlan10-to-vlan20-blocked.png)

<p><sub><strong>Screenshot 107 - VLAN 10 to VLAN 20 Blocked:</strong> PC0 receives destination-unreachable responses when pinging a VLAN 20 host.</sub></p>

![VLAN 20 to VLAN 10 Blocked](images/09-inter-vlan-acls/04-vlan20-to-vlan10-blocked.png)

<p><sub><strong>Screenshot 108 - VLAN 20 to VLAN 10 Blocked:</strong> PC2 cannot ping a VLAN 10 host.</sub></p>

![LAN3 Reachability Preserved](images/09-inter-vlan-acls/05-lan3-reachability-preserved.png)

<p><sub><strong>Screenshot 109 - LAN3 Reachability Preserved:</strong> A permitted ping reaches 10.10.10.1 outside the blocked VLAN pair.</sub></p>

![Gateway Reachability Preserved](images/09-inter-vlan-acls/06-gateway-reachability-preserved.png)

<p><sub><strong>Screenshot 110 - Gateway Reachability Preserved:</strong> VLAN 10 client retains reachability to its local default gateway.</sub></p>

---------

## PAT and Internal Web Validation

SAM-R2 demonstrates Port Address Translation between the `172.31.0.0/16` source network and the `172.19.0.0/16` server-facing segment. The router classifies matching inside sources and overloads them on the server-facing interface address.

The translation table proves the PAT behavior for `172.31.0.1`. Separate browser tests confirm that wired and wireless clients can resolve `www.sam.com` and reach the web server, but those `192.168.10.0/24` and `192.168.20.0/24` tests do not prove PAT because they do not match ACL 1.

> NAT is not an access-control substitute. Routing and ACLs decide whether a path is allowed; PAT only changes the addressing for traffic that matches its classification rule.

**Implemented controls:**

- Classified `172.31.0.0/16` for source translation.
- Configured PAT overload on SAM-R2.
- Validated the translation table and HTTP service reachability separately.

### Configure PAT on SAM-R2

GigabitEthernet0/0/0 is marked as the NAT inside interface and GigabitEthernet0/0/1 as outside for this laboratory direction. ACL 1 selects `172.31.0.0/16`, and the translation table maps the source to `172.19.0.1` while preserving unique ICMP identifiers.

> This is an internal source-NAT demonstration rather than an Internet edge deployment. The interface labels describe the configured translation direction, not a public/private trust classification.

![PAT Lab Topology](images/10-pat-and-web-validation/01-pat-lab-topology.png)

<p><sub><strong>Screenshot 111 - PAT Lab Topology:</strong> R2 sits between the 209.165.200.0/24 transit and 172.19.0.0/16 server LAN.</sub></p>

![SAM-R2 PAT Configuration](images/10-pat-and-web-validation/02-sam-r2-pat-configuration.png)

<p><sub><strong>Screenshot 112 - SAM-R2 PAT Configuration:</strong> R2 classifies 172.31.0.0/16 as the translated source and overloads on its server-facing interface.</sub></p>

![PAT Translation Table](images/10-pat-and-web-validation/03-pat-translation-table.png)

<p><sub><strong>Screenshot 113 - PAT Translation Table:</strong> show ip nat translations records ICMP translations from 172.31.0.1 to 172.19.0.1.</sub></p>

### Validate DNS and HTTP from every client

The wireless laptop and PCs in both user VLANs open `http://www.sam.com` and receive the Packet Tracer web page. These results validate DNS resolution, OSPF routing, ACL allowances for HTTP, and the web service itself.

> A browser result confirms application reachability. The PAT translation table remains the separate evidence for address translation.

![Wireless Web Validation](images/10-pat-and-web-validation/04-wireless-web-validation.png)

<p><sub><strong>Screenshot 114 - Wireless Web Validation:</strong> Wireless laptop resolves www.sam.com and loads the Packet Tracer web service.</sub></p>

![PC0 Web Validation](images/10-pat-and-web-validation/05-pc0-web-validation.png)

<p><sub><strong>Screenshot 115 - PC0 Web Validation:</strong> PC0 successfully opens www.sam.com.</sub></p>

![PC1 Web Validation](images/10-pat-and-web-validation/06-pc1-web-validation.png)

<p><sub><strong>Screenshot 116 - PC1 Web Validation:</strong> PC1 successfully opens www.sam.com.</sub></p>

![PC2 Web Validation](images/10-pat-and-web-validation/07-pc2-web-validation.png)

<p><sub><strong>Screenshot 117 - PC2 Web Validation:</strong> PC2 successfully opens www.sam.com.</sub></p>

![PC3 Web Validation](images/10-pat-and-web-validation/08-pc3-web-validation.png)

<p><sub><strong>Screenshot 118 - PC3 Web Validation:</strong> PC3 successfully opens www.sam.com.</sub></p>

![PC4 Web Validation](images/10-pat-and-web-validation/09-pc4-web-validation.png)

<p><sub><strong>Screenshot 119 - PC4 Web Validation:</strong> PC4 successfully opens www.sam.com.</sub></p>

![PC5 Web Validation](images/10-pat-and-web-validation/10-pc5-web-validation.png)

<p><sub><strong>Screenshot 120 - PC5 Web Validation:</strong> PC5 successfully opens www.sam.com.</sub></p>

---------

## HSRP Gateway Redundancy

SAM-R4 and SAM-S8 extend the topology so SAM-R3 and SAM-R4 can share virtual gateways on `192.168.13.0/24` and `10.10.10.0/24`. OSPF advertises the additional router, and HSRP priorities make SAM-R3 active while SAM-R4 remains standby.

Clients use the virtual IP rather than either router's physical address. Preemption allows the preferred higher-priority router to reclaim the active role after it returns.

> The evidence validates HSRP roles and normal-path connectivity, but it does not show SAM-R3 being disabled and SAM-R4 becoming active. The project therefore documents configured redundancy without claiming a completed failover test.

**Implemented controls:**

- Added and secured SAM-R4 and SAM-S8.
- Advertised the redundant path through OSPF.
- Configured two HSRP groups with active and standby priorities.

### Add the redundant router and switch

The expanded topology introduces SAM-R4 as an alternate gateway and SAM-S8 as an additional switching component. Both receive the same management baseline and SSH-only administration used elsewhere in the lab.

> HSRP provides router gateway redundancy. It does not make a switch a standby device; Layer 2 path selection is handled separately by STP and EtherChannel.

![Expanded HSRP Topology](images/11-hsrp-redundancy/01-expanded-hsrp-topology.png)

<p><sub><strong>Screenshot 121 - Expanded HSRP Topology:</strong> Router4 and Switch8 added to provide an alternate routed path for the two shared LANs.</sub></p>

![SAM-R4 Base Configuration](images/11-hsrp-redundancy/02-sam-r4-base-configuration.png)

<p><sub><strong>Screenshot 122 - SAM-R4 Base Configuration:</strong> Baseline device configuration and corrected hostname command for the standby router.</sub></p>

![SAM-R4 SSH Configuration](images/11-hsrp-redundancy/03-sam-r4-ssh-configuration.png)

<p><sub><strong>Screenshot 123 - SAM-R4 SSH Configuration:</strong> SAM-R4 configured with a local user, SSH-only VTY lines, SSH version 2, and a 2024-bit lab key.</sub></p>

![SAM-S8 Base Configuration](images/11-hsrp-redundancy/04-sam-s8-base-configuration.png)

<p><sub><strong>Screenshot 124 - SAM-S8 Base Configuration:</strong> Baseline configuration applied to the additional switch.</sub></p>

![SAM-S8 SSH Configuration](images/11-hsrp-redundancy/05-sam-s8-ssh-configuration.png)

<p><sub><strong>Screenshot 125 - SAM-S8 SSH Configuration:</strong> SAM-S8 configured with SSH version 2 and a local administrator account.</sub></p>

### Address SAM-R4 and advertise its networks

SAM-R4 receives `10.10.10.2/24` on LAN3 and `192.168.13.3/24` on the transit network. OSPF area 0 advertises both networks, and a successful ping confirms reachability to the new router address.

> Routing must be operational before HSRP can provide useful end-to-end redundancy; a virtual gateway cannot compensate for a missing upstream route.

![SAM-R4 LAN3 Interface](images/11-hsrp-redundancy/06-sam-r4-lan3-interface.png)

<p><sub><strong>Screenshot 126 - SAM-R4 LAN3 Interface:</strong> SAM-R4 GigabitEthernet0/0/0 configured as 10.10.10.2/24.</sub></p>

![SAM-R4 Transit Interface](images/11-hsrp-redundancy/07-sam-r4-transit-interface.png)

<p><sub><strong>Screenshot 127 - SAM-R4 Transit Interface:</strong> SAM-R4 GigabitEthernet0/0/1 configured as 192.168.13.3/24.</sub></p>

![SAM-R4 OSPF Configuration](images/11-hsrp-redundancy/08-sam-r4-ospf-configuration.png)

<p><sub><strong>Screenshot 128 - SAM-R4 OSPF Configuration:</strong> Both SAM-R4 networks advertised in OSPF area 0.</sub></p>

![SAM-R4 Ping Validation](images/11-hsrp-redundancy/09-sam-r4-ping-validation.png)

<p><sub><strong>Screenshot 129 - SAM-R4 Ping Validation:</strong> PC0 successfully pings the new SAM-R4 transit address.</sub></p>

### Configure the HSRP virtual gateways

HSRP group 1 uses virtual IP `192.168.13.222`, and group 2 uses `10.10.10.222`. SAM-R3 has priority 150 and SAM-R4 priority 100, with preemption enabled on both routers.

> Production HSRP commonly tracks uplink or routing health. Without tracking, an active router can retain the gateway role even when a different upstream dependency fails.

![HSRP Router Pair](images/11-hsrp-redundancy/10-hsrp-router-pair.png)

<p><sub><strong>Screenshot 130 - HSRP Router Pair:</strong> SAM-R3 and SAM-R4 share virtual gateway addresses across both connected networks.</sub></p>

![SAM-R3 HSRP Group 1](images/11-hsrp-redundancy/11-sam-r3-hsrp-group1.png)

<p><sub><strong>Screenshot 131 - SAM-R3 HSRP Group 1:</strong> SAM-R3 configured as preferred active router for virtual IP 192.168.13.222.</sub></p>

![SAM-R3 HSRP Group 2](images/11-hsrp-redundancy/12-sam-r3-hsrp-group2.png)

<p><sub><strong>Screenshot 132 - SAM-R3 HSRP Group 2:</strong> SAM-R3 configured as preferred active router for virtual IP 10.10.10.222.</sub></p>

![SAM-R4 HSRP Group 1](images/11-hsrp-redundancy/13-sam-r4-hsrp-group1.png)

<p><sub><strong>Screenshot 133 - SAM-R4 HSRP Group 1:</strong> SAM-R4 configured with lower priority for the 192.168.13.222 virtual gateway.</sub></p>

![SAM-R4 HSRP Group 2](images/11-hsrp-redundancy/14-sam-r4-hsrp-group2.png)

<p><sub><strong>Screenshot 134 - SAM-R4 HSRP Group 2:</strong> SAM-R4 configured with lower priority for the 10.10.10.222 virtual gateway.</sub></p>

### Validate active and standby state

PC6 uses the LAN3 virtual IP as its default gateway. `show standby brief` identifies SAM-R3 as active and SAM-R4 as standby for both groups, while ping and traceroute demonstrate normal connectivity through the active path.

> These checks prove role election and current reachability. A complete failover validation would additionally shut down the active path and capture SAM-R4 taking the active role.

![PC6 Virtual Gateway](images/11-hsrp-redundancy/15-pc6-virtual-gateway.png)

<p><sub><strong>Screenshot 135 - PC6 Virtual Gateway:</strong> PC6 uses 10.10.10.222 as its HSRP default gateway.</sub></p>

![SAM-R4 Standby Status](images/11-hsrp-redundancy/16-sam-r4-standby-status.png)

<p><sub><strong>Screenshot 136 - SAM-R4 Standby Status:</strong> show standby brief identifies SAM-R4 as standby for both HSRP groups.</sub></p>

![SAM-R3 Active Status](images/11-hsrp-redundancy/17-sam-r3-active-status.png)

<p><sub><strong>Screenshot 137 - SAM-R3 Active Status:</strong> show standby brief identifies SAM-R3 as active for both HSRP groups.</sub></p>

![PC6 Routed Path Validation](images/11-hsrp-redundancy/18-pc6-routed-path-validation.png)

<p><sub><strong>Screenshot 138 - PC6 Routed Path Validation:</strong> PC6 ping and traceroute test normal routed connectivity through the active gateway.</sub></p>

![PC4 Routed Path Validation](images/11-hsrp-redundancy/19-pc4-routed-path-validation.png)

<p><sub><strong>Screenshot 139 - PC4 Routed Path Validation:</strong> A second client validates normal connectivity across the HSRP-enabled topology.</sub></p>

---------

## STP and LACP EtherChannel

LAN3 receives redundant switching links between SAM-S5 and SAM-S7. LACP combines three physical FastEthernet links into one logical EtherChannel, while STP designates SAM-S7 as the root bridge for VLAN 1.

The design increases aggregate link capacity and gives STP a deterministic root for the Layer 2 topology. SAM-S6 and the additional trunks complete the alternate switching path.

> The screenshots prove configuration but do not include `show etherchannel summary` or `show spanning-tree`. Operational bundling and root election are therefore documented as configured, not independently validated.

**Implemented controls:**

- Configured trunk links around LAN3.
- Created LACP channel-group 1 on SAM-S5 and SAM-S7.
- Set SAM-S7 as the VLAN 1 STP root primary.

### Configure the redundant Layer 2 path

SAM-S5 and SAM-S7 place FastEthernet0/21-22 and FastEthernet0/24 in channel-group 1 using LACP active mode. SAM-S7 is then configured as the preferred VLAN 1 root bridge.

> EtherChannel member interfaces must use matching speed, duplex, trunk, and VLAN settings. A mismatch can suspend a member or prevent the bundle from forming.

![LAN3 Redundant Switching Topology](images/12-stp-etherchannel/01-lan3-redundant-switching-topology.png)

<p><sub><strong>Screenshot 140 - LAN3 Redundant Switching Topology:</strong> Switch7 is designated as STP root and linked to Switch5 through a multi-link EtherChannel.</sub></p>

![SAM-S6 Trunk Port](images/12-stp-etherchannel/02-sam-s6-trunk-port.png)

<p><sub><strong>Screenshot 141 - SAM-S6 Trunk Port:</strong> SAM-S6 uplink interfaces placed in trunk mode.</sub></p>

![SAM-S5 LACP Channel](images/12-stp-etherchannel/03-sam-s5-lacp-channel.png)

<p><sub><strong>Screenshot 142 - SAM-S5 LACP Channel:</strong> SAM-S5 member links assigned to channel-group 1 in active mode.</sub></p>

![SAM-S7 LACP and STP Root](images/12-stp-etherchannel/04-sam-s7-lacp-and-stp-root.png)

<p><sub><strong>Screenshot 143 - SAM-S7 LACP and STP Root:</strong> SAM-S7 builds the matching LACP channel and is configured as VLAN 1 root primary.</sub></p>

---------

## Centralized Syslog Monitoring

The Packet Tracer web server also hosts the Syslog service for this laboratory. SAM-R2 enables millisecond log timestamps and sends messages to `172.19.0.200`, allowing configuration events to be viewed from one console.

The server receives messages from `172.19.0.1`, confirming the remote logging path. The displayed 1993 date reveals that the simulated devices are not synchronized to a reliable time source.

> Central logs lose investigative value when clocks disagree. Production devices and collectors should use authenticated NTP where available, consistent time zones, protected transport, and a dedicated retention policy.

**Implemented controls:**

- Enabled detailed timestamps on SAM-R2 logs.
- Defined the remote Syslog host.
- Verified event arrival on the Packet Tracer service.

### Send SAM-R2 events to the Syslog server

SAM-R2 confirms reachability to the server, enables `service timestamps log datetime msec`, and configures `logging host 172.19.0.200`. The Syslog service then lists the received configuration events.

> Co-locating web and logging services is acceptable in this isolated lab. Production monitoring should use a dedicated, hardened collector separated from the application server.

![SAM-R2 Syslog Client](images/13-syslog-monitoring/01-sam-r2-syslog-client.png)

<p><sub><strong>Screenshot 144 - SAM-R2 Syslog Client:</strong> Router logging timestamps enabled and remote logging directed to 172.19.0.200.</sub></p>

![Syslog Server Events](images/13-syslog-monitoring/02-syslog-server-events.png)

<p><sub><strong>Screenshot 145 - Syslog Server Events:</strong> Packet Tracer Syslog service receives SAM-R2 configuration events with the unsynchronized lab date.</sub></p>

---------

## Source-Restricted Switch Management

SAM-S4 receives a management SVI at `172.19.0.254/16` on VLAN 1 and uses `172.19.0.1` as its default gateway. The earlier SSH ACL on SAM-R0 permits management traffic from VLAN 10 and denies it from VLAN 20.

This is a source-subnet restriction, not a claim that the switch management SVI belongs to VLAN 10. The screenshots show the actual SVI as VLAN 1.

> Production management should use a dedicated management VLAN or out-of-band network rather than VLAN 1, with AAA, least-privilege ACLs, logging, and restricted administrator workstations.

**Implemented controls:**

- Assigned a routed management address to SAM-S4.
- Configured the switch default gateway.
- Validated successful VLAN 10 SSH and denied VLAN 20 SSH.

### Configure the management SVI and gateway

SAM-S4 uses interface VLAN 1 with address `172.19.0.254/16` and gateway `172.19.0.1`. These settings make the Layer 2 switch reachable from remote routed networks.

> A Layer 2 switch needs an SVI address for management and an IP default gateway to return traffic to administrators outside the local subnet.

![SAM-S4 Management SVI](images/14-switch-management-acl/01-sam-s4-management-svi.png)

<p><sub><strong>Screenshot 146 - SAM-S4 Management SVI:</strong> VLAN 1 SVI configured as 172.19.0.254/16 for switch management.</sub></p>

![SAM-S4 Default Gateway](images/14-switch-management-acl/02-sam-s4-default-gateway.png)

<p><sub><strong>Screenshot 147 - SAM-S4 Default Gateway:</strong> Switch default gateway configured as 172.19.0.1.</sub></p>

### Validate allowed and denied SSH sources

PC0 in VLAN 10 successfully authenticates to `172.19.0.254`, while PC2 in VLAN 20 receives a timeout. The paired tests demonstrate that the source-based SSH restriction affects the intended management connection.

> Allowed and denied tests together provide stronger evidence than the ACL configuration alone because they show the observed client behavior on both sides of the policy.

![VLAN 10 Switch SSH Allowed](images/14-switch-management-acl/03-vlan10-switch-ssh-allowed.png)

<p><sub><strong>Screenshot 148 - VLAN 10 Switch SSH Allowed:</strong> VLAN 10 client successfully opens an SSH session to 172.19.0.254.</sub></p>

![VLAN 20 Switch SSH Denied](images/14-switch-management-acl/04-vlan20-switch-ssh-denied.png)

<p><sub><strong>Screenshot 149 - VLAN 20 Switch SSH Denied:</strong> VLAN 20 client receives a timeout when attempting the same management connection.</sub></p>

## Testing and Verification

- Confirmed six wired DHCP leases and one wireless DHCP lease with the expected gateway and DNS values.
- Confirmed OSPF FULL adjacencies and routed access to the DNS/server LAN.
- Confirmed SSH succeeds from permitted VLAN 10 sources and is denied from VLAN 20.
- Confirmed reciprocal inter-VLAN ACL blocking while unrelated routed destinations remain reachable.
- Confirmed a PAT translation entry for traffic sourced from `172.31.0.0/16`.
- Confirmed all wired and wireless clients can resolve and open `www.sam.com`.
- Confirmed HSRP active and standby roles under normal operation; failover itself was not captured.
- Confirmed Syslog event delivery; the laboratory clock remains unsynchronized.
- Recorded Port Security violation state; the original ping to `172.168.0.1` is retained but not treated as validation.

## Results

The completed lab demonstrates how core CCNA routing and switching features work together in one environment. VLANs and router subinterfaces create the user segments, DHCP and DNS provide client services, OSPF carries routes between the sites, ACLs constrain management and inter-VLAN traffic, and PAT demonstrates source translation toward the server segment.

The later stages add encrypted management, access-port protection, HSRP gateway roles, STP root selection, LACP EtherChannel, and centralized event collection. Evidence limitations and production distinctions are documented in [`docs/notes.md`](docs/notes.md) instead of overstating what a configuration screen proves.

## Skills Demonstrated

- Cisco IOS device initialization and management-plane configuration
- VLAN creation, access-port assignment, 802.1Q trunks, and router-on-a-stick
- Router-based DHCP, static server addressing, DNS, and wireless integration
- OSPF single-area routing and route-expansion troubleshooting
- Standard and extended ACL design and validation
- SSH version 2, source-restricted management, and Port Security
- PAT, HSRP, STP root selection, and LACP EtherChannel
- Syslog configuration, evidence interpretation, and production-risk analysis

## Repository Structure

```text
cisco-packet-tracer-enterprise-network-lab/
|-- README.md
|-- LICENSE
|-- IMAGE_MANIFEST.md
|-- .gitattributes
|-- docs/
|   |-- commands.md
|   `-- notes.md
|-- configs/
|   `-- ccna-enterprise-network-lab.pkt
`-- images/
    |-- 01-network-topology/
    |-- 02-device-foundation/
    |-- 03-vlan-segmentation/
    |-- 04-dhcp-router-on-a-stick/
    |-- 05-server-dns-wireless/
    |-- 06-port-security/
    |-- 07-ospf-routing/
    |-- 08-ssh-management/
    |-- 09-inter-vlan-acls/
    |-- 10-pat-and-web-validation/
    |-- 11-hsrp-redundancy/
    |-- 12-stp-etherchannel/
    |-- 13-syslog-monitoring/
    |-- 14-switch-management-acl/
    `-- 15-source-context/
```

## Notes

- Credentials shown in screenshots are isolated Packet Tracer laboratory values and must not be reused.
- The Packet Tracer file is retained unchanged as the original configuration artifact.
- HSRP failover, STP root state, and EtherChannel operational state require additional live validation commands to be considered fully proven.
- The `/16` laboratory LANs, WPA2-Personal WLAN, VLAN 1 management SVI, Telnet history, and combined web/Syslog server should be redesigned before production use.
- The complete screenshot set is indexed in [IMAGE_MANIFEST.md](IMAGE_MANIFEST.md).

### Retained source context

The final contextual artwork from the source is preserved to keep the complete visual inventory, but it is not used as technical evidence.

![Cisco Source Artwork](images/15-source-context/01-cisco-source-artwork.png)

<p><sub><strong>Screenshot 150 - Cisco Source Artwork:</strong> Contextual Cisco artwork retained from the source document.</sub></p>
