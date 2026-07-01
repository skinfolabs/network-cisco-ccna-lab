# STP and LACP EtherChannel

LAN3 receives redundant switching links between SAM-S5 and SAM-S7. LACP bundles multiple physical links into one EtherChannel, while STP controls the forwarding path when redundancy exists.

## Technical Context

The design increases aggregate capacity, reduces Layer 2 loop risk, and gives the topology a predictable root bridge. SAM-S6 and additional trunks complete the alternate LAN3 switching path.

> LACP and STP are complementary. LACP makes several physical links behave as one bundle, and STP decides the forwarding structure for the remaining Layer 2 topology so redundant links do not create a switching loop.

**Implemented controls:**

- Configured trunk links around LAN3.
- Created LACP channel-group 1 on SAM-S5 and SAM-S7.
- Set SAM-S7 as the VLAN 1 STP root primary.

## Key Technical Terms

| Term | Meaning in this chapter |
|------|-------------------------|
| STP | Spanning Tree Protocol, which prevents Layer 2 loops in redundant switching paths. |
| Root bridge | The switch STP treats as the logical center of the Layer 2 topology. |
| EtherChannel | A bundle of physical links treated as one logical link. |
| LACP | Link Aggregation Control Protocol, used to negotiate EtherChannel membership. |
| Trunk consistency | Matching VLAN, native VLAN, speed, and duplex expectations required for stable bundled links. |

---

## Detailed Walkthrough

### Step 01 - Configure the redundant Layer 2 path

SAM-S5 and SAM-S7 place FastEthernet0/21-22 and FastEthernet0/24 in LACP channel-group 1, creating one logical bundle from three links. SAM-S7 becomes the preferred VLAN 1 STP root.

> EtherChannel member interfaces must use matching speed, duplex, trunk, and VLAN settings. A mismatch can suspend a member or prevent the bundle from forming.

![LAN3 Redundant Switching Topology](../../images/10-stp-etherchannel/01-lan3-redundant-switching-topology.png)

<p><sub><strong>Screenshot 048 - LAN3 Redundant Switching Topology:</strong> Switch7 is designated as STP root and linked to Switch5 through a multi-link EtherChannel.</sub></p>

The command blocks turn the topology into trunk links, an LACP bundle, and a deliberate STP root choice.

#### SAM-S6

SAM-S6 configures GigabitEthernet0/1 and FastEthernet0/23 as static trunks in the redundant LAN3 path.

```cisco
configure terminal
interface range GigabitEthernet0/1, FastEthernet0/23
 switchport mode trunk
end
write memory
```

#### SAM-S5

SAM-S5 places FastEthernet0/21-22 and FastEthernet0/24 into LACP channel-group 1 so VLAN traffic uses the logical bundle.

```cisco
configure terminal
interface FastEthernet0/23
 switchport mode trunk
 exit
interface range FastEthernet0/21 - 22, FastEthernet0/24
 channel-group 1 mode active
 switchport mode trunk
end
write memory
```

#### SAM-S7

SAM-S7 builds the matching LACP bundle and becomes the preferred STP root for VLAN 1. Root placement matters because STP forwarding decisions are based on the root location.

```cisco
configure terminal
interface GigabitEthernet0/2
 switchport mode trunk
 exit
interface range FastEthernet0/21 - 22, FastEthernet0/24
 channel-group 1 mode active
 switchport mode trunk
 exit
spanning-tree vlan 1 root primary
end
write memory
```

---

## Validation and Summary

Validation confirms the STP root choice, trunk links, and LACP bundle so the redundant Layer 2 path is intentional rather than default-driven.

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
