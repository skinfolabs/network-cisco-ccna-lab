# Source-Restricted Switch Management

SAM-S4 receives a management SVI at `172.19.0.254/16` on VLAN 1 and uses `172.19.0.1` as its default gateway. This gives the Layer 2 switch an IP identity for administration even though the switch is not routing user traffic. The earlier SSH ACL on SAM-R0 is then used to permit management traffic from VLAN 10 and deny it from VLAN 20.

## Technical Context

This is a management-plane control. The goal is not to block all traffic from VLAN 20, but to prevent VLAN 20 hosts from opening SSH sessions to the switch management address. It is also a source-subnet restriction, not a claim that the switch management SVI belongs to VLAN 10. The implemented SVI is VLAN 1.

> Production management should use a dedicated management VLAN or out-of-band network rather than VLAN 1, with AAA, least-privilege ACLs, logging, and restricted administrator workstations.

**Implemented controls:**

- Assigned a routed management address to SAM-S4.
- Configured the switch default gateway.
- Validated successful VLAN 10 SSH and denied VLAN 20 SSH.

## Key Technical Terms

| Term | Meaning in this chapter |
|------|-------------------------|
| SVI | Switched Virtual Interface, a Layer 3 management interface on a switch. |
| Management plane | The administrative access path used to configure and monitor the device. |
| Default gateway | The upstream router a Layer 2 switch uses when management traffic leaves its local subnet. |
| Source restriction | A control that allows management only from approved subnets while denying other client networks. |

---

## Detailed Walkthrough

### Step 01 - Configure the management SVI and gateway

SAM-S4 uses interface VLAN 1 with address `172.19.0.254/16` and gateway `172.19.0.1`. The SVI gives administrators a target IP address for SSH, and the default gateway tells the switch where to send replies when the administrator is not on the same local subnet. Without the default gateway, an SSH request might reach the switch, but the switch could fail to return traffic to the remote client.

> A Layer 2 switch needs an SVI address for management and an IP default gateway to return traffic to administrators outside the local subnet.

#### SAM-S4

SAM-S4 receives an SVI address for remote management and a default gateway for returning traffic to administrators outside the local subnet.

```cisco
configure terminal
interface vlan 1
 ip address 172.19.0.254 255.255.0.0
 no shutdown
 exit
ip default-gateway 172.19.0.1
end
write memory
```

---

### Step 02 - Validate allowed and denied SSH sources

PC0 in VLAN 10 successfully authenticates to `172.19.0.254`, while PC2 in VLAN 20 receives a timeout. The paired tests demonstrate that the source-based SSH restriction affects the intended management connection. This is stronger than showing the ACL configuration alone because it proves the policy from both sides: an allowed administrative source and a denied non-administrative source.

> Allowed and denied tests together provide stronger evidence than the ACL configuration alone because they show the observed client behavior on both sides of the policy.

![VLAN 10 Switch SSH Allowed](../../images/12-switch-management-acl/01-vlan10-switch-ssh-allowed.png)

<p><sub><strong>Screenshot 050 - VLAN 10 Switch SSH Allowed:</strong> VLAN 10 client successfully opens an SSH session to 172.19.0.254.</sub></p>

The successful session confirms that the management address, return path, SSH service, local credentials, and allow rule all work together for the approved source network.

![VLAN 20 Switch SSH Denied](../../images/12-switch-management-acl/02-vlan20-switch-ssh-denied.png)

<p><sub><strong>Screenshot 051 - VLAN 20 Switch SSH Denied:</strong> VLAN 20 client receives a timeout when attempting the same management connection.</sub></p>

The denied test confirms that the restriction is selective. VLAN 20 cannot manage the switch over SSH, while the previous screenshot shows that management remains available from the approved VLAN 10 source.

---

## Validation and Summary

Switch management access is validated with one allowed VLAN 10 SSH session and one denied VLAN 20 attempt. The result confirms that source-restricted management is applied to the switch SVI path.

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
