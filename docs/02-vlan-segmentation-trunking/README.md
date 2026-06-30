# VLAN Segmentation and Trunk Hardening

VLAN 10 and VLAN 20 create separate Layer 2 broadcast domains for the user devices. Access ports place endpoints in the correct VLAN, while 802.1Q trunks carry both networks between switches and toward the router-on-a-stick gateway.

VLAN 99 is assigned as the native VLAN on the trunks and DTP negotiation is disabled. This removes reliance on the default native VLAN and prevents links from dynamically negotiating trunk mode.

> VLANs provide segmentation but do not independently enforce routed security. Inter-VLAN traffic is controlled later by router ACLs.

**Implemented controls:**

- Created and named VLAN 10 and VLAN 20.
- Assigned endpoint-facing interfaces as access ports.
- Configured static 802.1Q trunks with native VLAN 99 and `nonegotiate`.

## Key Technical Terms

| Term | Meaning in this chapter |
|------|-------------------------|
| VLAN | A Layer 2 broadcast domain used to separate clients even when they share physical switches. |
| Access port | A switch port assigned to one VLAN for an endpoint such as a PC, printer, or server. |
| Trunk port | A link that carries multiple VLANs between switches or toward a router-on-a-stick gateway. |
| 802.1Q tag | The VLAN identifier added to Ethernet frames while they cross a trunk. |
| Native VLAN | The VLAN used for untagged traffic on an 802.1Q trunk; matching it on both sides prevents confusing Layer 2 behavior. |
| DTP | Dynamic Trunking Protocol. Disabling negotiation keeps trunk mode intentional instead of automatically negotiated. |

### Create the user VLANs

The VLAN databases on SAM-S0, SAM-S1, and SAM-S3 are populated with VLAN 10 (`YELLOW`) and VLAN 20 (`BLUE`). Matching VLAN IDs are required on every switch that must transport those frames.

> A VLAN name is operational documentation; the numeric VLAN ID is the value carried in the 802.1Q tag.

#### SAM-S0

SAM-S0 creates both user VLANs before its access ports and trunks are assigned.

```cisco
configure terminal
vlan 10
 name YELLOW
vlan 20
 name BLUE
end
write memory
```

#### SAM-S1

SAM-S1 uses the same VLAN IDs and names so tagged traffic remains consistent across the switching path.

```cisco
configure terminal
vlan 10
 name YELLOW
vlan 20
 name BLUE
end
write memory
```

#### SAM-S3

SAM-S3 creates the same user VLANs before carrying them toward the router-on-a-stick gateway.

```cisco
configure terminal
vlan 10
 name YELLOW
vlan 20
 name BLUE
end
write memory
```

### Assign access ports to VLAN 10 and VLAN 20

Endpoint-facing FastEthernet ports are forced into access mode and assigned to their intended user VLAN. This prevents attached clients from negotiating a trunk and defines which broadcast domain each client joins.

> An incorrect access VLAN can place a client in the wrong IP subnet, bypass the intended ACL source classification, or prevent DHCP from reaching the correct pool.

#### SAM-S0

SAM-S0 places FastEthernet0/1-2 in VLAN 10 and FastEthernet0/5 in VLAN 20.

```cisco
configure terminal
interface range FastEthernet0/1 - 2
 switchport mode access
 switchport access vlan 10
 exit
interface FastEthernet0/5
 switchport mode access
 switchport access vlan 20
end
write memory
```

#### SAM-S1

SAM-S1 uses the opposite endpoint distribution: FastEthernet0/5 joins VLAN 10 and FastEthernet0/1-2 join VLAN 20.

```cisco
configure terminal
interface FastEthernet0/5
 switchport mode access
 switchport access vlan 10
 exit
interface range FastEthernet0/1 - 2
 switchport mode access
 switchport access vlan 20
end
write memory
```

### Configure and validate hardened trunks

The switch uplinks are placed in trunk mode, VLAN 99 is selected as the native VLAN, and DTP is disabled. `show interfaces trunk` then confirms 802.1Q status and native VLAN consistency across the participating switches.

> A native-VLAN mismatch can leak untagged traffic into the wrong VLAN and produce difficult Layer 2 troubleshooting symptoms.

#### SAM-S0

SAM-S0 creates native VLAN 99 and statically configures its uplinks as non-negotiating trunks.

```cisco
configure terminal
vlan 99
 name Native
interface FastEthernet0/3
 switchport mode trunk
 switchport trunk native vlan 99
 switchport nonegotiate
 exit
interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk native vlan 99
 switchport nonegotiate
end
write memory
```

#### SAM-S1

SAM-S1 applies the same native VLAN and static-trunk policy to both GigabitEthernet uplinks.

```cisco
configure terminal
vlan 99
 name Native
interface range GigabitEthernet0/1 - 2
 switchport mode trunk
 switchport trunk native vlan 99
 switchport nonegotiate
end
write memory
```

#### SAM-S3

SAM-S3 carries the user VLANs through FastEthernet0/24 and its GigabitEthernet uplinks.

```cisco
configure terminal
vlan 99
 name Native
interface FastEthernet0/24
 switchport mode trunk
 switchport trunk native vlan 99
 switchport nonegotiate
 exit
interface range GigabitEthernet0/1 - 2
 switchport mode trunk
 switchport trunk native vlan 99
 switchport nonegotiate
end
write memory
```

#### Trunk validation

The `show interfaces trunk` output confirms 802.1Q operation and native VLAN 99 on every participating link.

```text
SAM-S0# show interfaces trunk
Port      Mode  Encapsulation  Status    Native vlan
Fa0/3     on    802.1q         trunking  99
Gi0/1     on    802.1q         trunking  99

SAM-S1# show interfaces trunk
Port      Mode  Encapsulation  Status    Native vlan
Gi0/1     on    802.1q         trunking  99
Gi0/2     on    802.1q         trunking  99

SAM-S3# show interfaces trunk
Port      Mode  Encapsulation  Status    Native vlan
Fa0/24    on    802.1q         trunking  99
Gi0/1     on    802.1q         trunking  99
Gi0/2     on    802.1q         trunking  99
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
