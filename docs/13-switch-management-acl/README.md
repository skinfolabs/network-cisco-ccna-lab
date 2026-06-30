# Source-Restricted Switch Management

SAM-S4 receives a management SVI at `172.19.0.254/16` on VLAN 1 and uses `172.19.0.1` as its default gateway. This gives the Layer 2 switch an IP identity for administration even though the switch is not routing user traffic. The earlier SSH ACL on SAM-R0 is then used to permit management traffic from VLAN 10 and deny it from VLAN 20.

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

### Configure the management SVI and gateway

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

### Validate allowed and denied SSH sources

PC0 in VLAN 10 successfully authenticates to `172.19.0.254`, while PC2 in VLAN 20 receives a timeout. The paired tests demonstrate that the source-based SSH restriction affects the intended management connection. This is stronger than showing the ACL configuration alone because it proves the policy from both sides: an allowed administrative source and a denied non-administrative source.

> Allowed and denied tests together provide stronger evidence than the ACL configuration alone because they show the observed client behavior on both sides of the policy.

![VLAN 10 Switch SSH Allowed](../../images/12-switch-management-acl/01-vlan10-switch-ssh-allowed.png)

<p><sub><strong>Screenshot 050 - VLAN 10 Switch SSH Allowed:</strong> VLAN 10 client successfully opens an SSH session to 172.19.0.254.</sub></p>

The successful session confirms that the management address, return path, SSH service, local credentials, and allow rule all work together for the approved source network.

![VLAN 20 Switch SSH Denied](../../images/12-switch-management-acl/02-vlan20-switch-ssh-denied.png)

<p><sub><strong>Screenshot 051 - VLAN 20 Switch SSH Denied:</strong> VLAN 20 client receives a timeout when attempting the same management connection.</sub></p>

The denied test confirms that the restriction is selective. VLAN 20 cannot manage the switch over SSH, while the previous screenshot shows that management remains available from the approved VLAN 10 source.

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
