# VLAN Segmentation and Trunk Hardening

VLAN 10 and VLAN 20 create separate Layer 2 broadcast domains for the user devices. Access ports place endpoints in the correct VLAN, while 802.1Q trunks carry both networks between switches and toward the router-on-a-stick gateway.

## Technical Context

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

---

## Detailed Walkthrough

### Step 01 - Create the user VLANs

The VLAN databases on SAM-S0, SAM-S1, and SAM-S3 are populated with VLAN 10 (`YELLOW`) and VLAN 20 (`BLUE`). Matching VLAN IDs are required on every switch that must transport those frames.

> A VLAN name is operational documentation; the numeric VLAN ID is the value carried in the 802.1Q tag.

The VLAN creation commands are identical on `SAM-S0`, `SAM-S1`, and `SAM-S3`, so they are documented once. Each switch receives the same VLAN IDs and names before access-port and trunk configuration is applied.

```cisco
configure terminal
vlan 10
 name YELLOW
vlan 20
 name BLUE
end
write memory
```

---

### Step 02 - Assign access ports to VLAN 10 and VLAN 20

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

---

### Step 03 - Configure and validate hardened trunks

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

---

## Validation and Summary

The VLAN and trunk configuration is validated through the documented switch configuration and `show interfaces trunk` output. VLAN 10 and VLAN 20 are consistently defined, access ports are assigned, and native VLAN 99 is used on static trunks with DTP disabled.

---

## Project Chapters

| # | Chapter | Description |
|---|---------|-------------|
| 0 | [Project Overview](../../README.md) | Main project overview, objectives, tools, and skills |
| 1 | [Topology and Lab Environment](../01-topology-and-lab-environment/README.md) | Topology, lab areas, devices, addressing, and traffic relationships |
| 2 | [Device Identity and Management Foundation](../02-device-identity-management/README.md) | Hostnames, local access, banners, console/VTY baseline, and device setup |
| 3 | [VLAN Segmentation and Trunk Hardening](../03-vlan-segmentation-trunking/README.md) | VLAN creation, access ports, trunk hardening, and trunk validation |
| 4 | [DHCP and Router-on-a-Stick Routing](../04-dhcp-router-on-a-stick/README.md) | Router subinterfaces, DHCP pools, switch trunk path, and client leases |
| 5 | [Server, DNS, and Wireless Services](../05-server-dns-wireless/README.md) | Static servers, DNS publishing, WLAN profile, WPA2 access, and wireless path validation |
| 6 | [Access-Layer Port Security](../06-port-security/README.md) | Unused-port shutdown, sticky MAC learning, violation mode, and validation limits |
| 7 | [OSPF Dynamic Routing](../07-ospf-routing/README.md) | Routed transit links, OSPF advertisements, adjacency validation, and LAN3 expansion |
| 8 | [SSH Management and Source ACLs](../08-ssh-management-acls/README.md) | SSH version 2 configuration, management access, and source-based ACL restriction |
| 9 | [Inter-VLAN Access Control](../09-inter-vlan-access-control/README.md) | Inter-VLAN isolation policy and validation of blocked and preserved reachability |
| 10 | [PAT and Internal Web Validation](../10-pat-web-validation/README.md) | PAT configuration on SAM-R2 and client DNS/HTTP validation |
| 11 | [HSRP Gateway Redundancy](../11-hsrp-redundancy/README.md) | Redundant gateway topology, HSRP active/standby roles, and validation limits |
| 12 | [STP and LACP EtherChannel](../12-stp-etherchannel/README.md) | STP root control, redundant switching, and LACP EtherChannel configuration |
| 13 | [Centralized Syslog Monitoring](../13-syslog-monitoring/README.md) | Centralized Syslog destination and event collection validation |
| 14 | [Source-Restricted Switch Management](../14-switch-management-acl/README.md) | Switch SVI management access and VLAN-based SSH allow/deny validation |
| 15 | [Final Summary](../15-final-summary/README.md) | Validation summary, production recommendations, skills, and project closure |
