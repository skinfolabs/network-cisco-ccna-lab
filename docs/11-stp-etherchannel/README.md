# STP and LACP EtherChannel

LAN3 receives redundant switching links between SAM-S5 and SAM-S7. This chapter combines two Layer 2 technologies that solve different problems: LACP bundles multiple physical links into one logical EtherChannel, while STP controls which switching path should forward traffic when redundant paths exist.

The design increases aggregate link capacity, reduces the chance of accidental Layer 2 loops, and gives the switching topology a predictable root bridge. SAM-S6 and the additional trunks complete the alternate LAN3 switching path so the redundant area is not just drawn in the topology, but also configured at the switch level.

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

### Configure the redundant Layer 2 path

SAM-S5 and SAM-S7 place FastEthernet0/21-22 and FastEthernet0/24 in channel-group 1 using LACP active mode. This creates one logical bundle from three member links. SAM-S7 is then configured as the preferred VLAN 1 root bridge so STP has a planned control point instead of relying only on default bridge priorities.

> EtherChannel member interfaces must use matching speed, duplex, trunk, and VLAN settings. A mismatch can suspend a member or prevent the bundle from forming.

![LAN3 Redundant Switching Topology](../../images/10-stp-etherchannel/01-lan3-redundant-switching-topology.png)

<p><sub><strong>Screenshot 048 - LAN3 Redundant Switching Topology:</strong> Switch7 is designated as STP root and linked to Switch5 through a multi-link EtherChannel.</sub></p>

The topology evidence shows the intended redundant Layer 2 design. The command blocks below document the actual switch-side configuration that turns the drawing into trunk links, an LACP bundle, and a deliberate STP root choice.

#### SAM-S6

SAM-S6 configures GigabitEthernet0/1 and FastEthernet0/23 as static trunks within the redundant LAN3 switching path. These trunks allow VLAN traffic to continue across the redundant switch area instead of limiting traffic to access-only links.

```cisco
configure terminal
interface range GigabitEthernet0/1, FastEthernet0/23
 switchport mode trunk
end
write memory
```

#### SAM-S5

SAM-S5 places FastEthernet0/21-22 and FastEthernet0/24 into LACP channel-group 1 and carries VLAN traffic over the logical bundle. `channel-group 1 mode active` tells the switch to actively negotiate LACP instead of forming a static, negotiation-free bundle.

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

SAM-S7 builds the matching LACP bundle and becomes the preferred STP root for VLAN 1. Selecting the root bridge deliberately matters because STP path decisions are based on the root location; in a real network, a poor root placement can send traffic over inefficient or unexpected links.

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
