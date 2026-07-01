# OSPF Dynamic Routing

OSPF area 0 connects the user VLANs, transit links, server LAN, and later LAN3 networks. Routers advertise connected prefixes and form FULL adjacencies over transit segments.

## Technical Context

The initial three-router design expands through `192.168.13.0/24` to SAM-R3, which also advertises `10.10.10.0/24` for the later LAN3 and HSRP path.

> OSPF router IDs can appear as an address from another interface, so a neighbor ID does not have to match the connected-link address. Production OSPF should also use explicit passive interfaces and authentication where supported.

**Implemented controls:**

- Addressed the routed transit interfaces.
- Advertised all documented networks in OSPF area 0.
- Verified FULL adjacency and server-LAN reachability.

## Key Technical Terms

| Term | Meaning in this chapter |
|------|-------------------------|
| OSPF | A dynamic routing protocol that allows routers to learn network paths from each other. |
| Area 0 | The OSPF backbone area used by the lab to keep routing design simple. |
| Adjacency | A neighbor relationship between routers that exchange OSPF information. |
| Wildcard mask | The inverse-style mask used in Cisco OSPF network statements to match interfaces. |
| Passive interface | An interface advertised into OSPF without forming neighbor relationships on that segment. |

---

## Detailed Walkthrough

### Step 01 - Address the core transit links

The `172.31.0.0/16` and `209.165.200.0/24` transit networks connect SAM-R0, SAM-R1, and SAM-R2. Each side receives an address before OSPF is enabled.

> Dynamic routing cannot repair an incorrect connected-link mask. Both ends must agree on the subnet before a neighbor relationship can form.

![Routed Core Topology](../../images/05-ospf-routing/00-routed-core-topology.png)

<p><sub><strong>Screenshot 020 - Routed Core Topology:</strong> Transit networks between SAM-R0, SAM-R1, and SAM-R2.</sub></p>

#### SAM-R0

SAM-R0 addresses the transit link to SAM-R1 and advertises both user VLANs and the transit network in OSPF area 0.

```cisco
configure terminal
interface GigabitEthernet0/0/1
 ip address 172.31.0.1 255.255.0.0
 no shutdown
 exit
router ospf 1
 network 192.168.10.0 0.0.0.255 area 0
 network 192.168.20.0 0.0.0.255 area 0
 network 172.31.0.0 0.0.255.255 area 0
 passive-interface GigabitEthernet0/0/0
end
write memory
```

#### SAM-R1

SAM-R1 connects the two routed transit networks and advertises both in OSPF area 0.

```cisco
configure terminal
interface GigabitEthernet0/0/0
 ip address 172.31.0.2 255.255.0.0
 no shutdown
 exit
interface GigabitEthernet0/0/1
 ip address 209.165.200.1 255.255.255.0
 no shutdown
 exit
router ospf 1
 network 172.31.0.0 0.0.255.255 area 0
 network 209.165.200.0 0.0.0.255 area 0
end
write memory
```

#### SAM-R2

SAM-R2 addresses the SAM-R1 transit link and the server LAN. `172.19.0.1/16` becomes the gateway for DNS, web, and Syslog services.

```cisco
configure terminal
interface GigabitEthernet0/0/0
 ip address 209.165.200.2 255.255.255.0
 no shutdown
 exit
interface GigabitEthernet0/0/1
 ip address 172.19.0.1 255.255.0.0
 no shutdown
 exit
router ospf 1
 network 209.165.200.0 0.0.0.255 area 0
 network 172.19.0.0 0.0.255.255 area 0
 passive-interface GigabitEthernet0/0/1
end
write memory
```

After the network statements are added, routers report FULL OSPF adjacency and the server LAN becomes reachable.

---

### Step 02 - Advertise the initial networks and validate adjacency

SAM-R0 advertises user VLANs and the R0-R1 transit, SAM-R1 advertises both transit networks, and SAM-R2 advertises the R1-R2 transit plus server LAN. FULL adjacency confirms neighbor formation, while the DNS-server ping confirms reachability after an initial timeout.

> With router subinterfaces, production OSPF should make user-facing subinterfaces passive explicitly or use `passive-interface default` with selected transit links enabled.

![DNS Server Ping](../../images/05-ospf-routing/01-dns-server-ping.png)

<p><sub><strong>Screenshot 021 - DNS Server Ping:</strong> Initial DNS-server ping receives three of four replies, demonstrating reachability after the first timeout.</sub></p>

---

### Step 03 - Extend OSPF to SAM-R3 and LAN3

SAM-R1 and SAM-R3 receive `192.168.13.0/24` addresses, and SAM-R3 receives `10.10.10.1/24` for LAN3. Both networks are advertised in area 0.

> Error output can be useful when the corrected command follows immediately, because it preserves the troubleshooting path and final syntax.

![SAM-R1 to SAM-R3 Expansion](../../images/05-ospf-routing/02-sam-r1-r3-expansion-topology.png)

<p><sub><strong>Screenshot 022 - SAM-R1 to SAM-R3 Expansion:</strong> New 192.168.13.0/24 link and LAN3 path added to the routed topology.</sub></p>

#### SAM-R1

SAM-R1 adds the `192.168.13.0/24` link toward SAM-R3 and advertises it through OSPF.

```cisco
configure terminal
interface GigabitEthernet0/0/2
 ip address 192.168.13.1 255.255.255.0
 no shutdown
 exit
router ospf 1
 network 192.168.13.0 0.0.0.255 area 0
end
write memory
```

#### SAM-R3

SAM-R3 connects the new transit segment to LAN3 and advertises both networks in area 0.

```cisco
configure terminal
interface GigabitEthernet0/0/0
 ip address 192.168.13.2 255.255.255.0
 no shutdown
 exit
interface GigabitEthernet0/0/1
 ip address 10.10.10.1 255.255.255.0
 no shutdown
 exit
router ospf 1
 network 192.168.13.0 0.0.0.255 area 0
 network 10.10.10.0 0.0.0.255 area 0
 passive-interface GigabitEthernet0/0/1
end
write memory
```

---

## Validation and Summary

Validation confirms interface addressing, OSPF advertisements, adjacency state, server-LAN reachability, and the later LAN3 expansion.

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
