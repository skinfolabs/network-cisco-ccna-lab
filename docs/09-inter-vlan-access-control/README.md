# Inter-VLAN Access Control

Two standard ACLs isolate VLAN 10 and VLAN 20 from each other while preserving access to other routed networks. Because a standard ACL matches only source IPv4 addresses, each list is placed outbound on the destination VLAN subinterface.

## Technical Context

The tests show reciprocal blocking between the two user VLANs and continued reachability to local gateways and the LAN3 router address.

> Standard ACL placement matters: placing the same source-only rule too close to the source can unintentionally block that network from every destination.

**Implemented controls:**

- Blocked VLAN 10 sources from exiting toward VLAN 20.
- Blocked VLAN 20 sources from exiting toward VLAN 10.
- Preserved access to destinations outside the isolated VLAN pair.

## Key Technical Terms

| Term | Meaning in this chapter |
|------|-------------------------|
| Inter-VLAN traffic | Routed traffic moving between VLAN 10 and VLAN 20 through the router. |
| Standard ACL | An access list that matches only source addresses, so placement is critical. |
| ACL placement | The interface and direction where a filter is applied; wrong placement can block too much traffic. |
| Destination-unreachable reply | A validation signal showing traffic was refused by the routing path or policy rather than silently ignored. |

---

## Detailed Walkthrough

### Step 01 - Map the segmentation test topology

The topology identifies the clients in both VLANs and the SAM-R0 subinterfaces where the outbound filters are applied.

> The ACLs operate at Layer 3; devices in the same VLAN still communicate through Layer 2 switching unless another control is introduced.

![Inter-VLAN ACL Topology](../../images/07-inter-vlan-acls/01-inter-vlan-acl-topology.png)

<p><sub><strong>Screenshot 029 - Inter-VLAN ACL Topology:</strong> Client VLANs and routed gateway used for the segmentation test.</sub></p>

---

### Step 02 - Apply reciprocal ACLs and validate behavior

ACL 20 denies the VLAN 10 source range before traffic leaves toward VLAN 20, and ACL 10 denies the VLAN 20 source range before traffic leaves toward VLAN 10. Destination-unreachable responses confirm the blocked paths, while successful pings confirm that unrelated routes remain available.

> The gateway-generated unreachable response demonstrates a routed policy decision more clearly than a silent timeout because it identifies the filtering router as the responding device.

#### SAM-R0

The two standard ACLs are placed outbound toward the destination VLANs. Each list blocks the opposite user subnet and permits every other source.

```cisco
configure terminal
ip access-list standard 20
 deny 192.168.10.0 0.0.0.255
 permit any
 exit
interface GigabitEthernet0/0/0.20
 ip access-group 20 out
 exit
ip access-list standard 10
 deny 192.168.20.0 0.0.0.255
 permit any
 exit
interface GigabitEthernet0/0/0.10
 ip access-group 10 out
end
write memory
```

![VLAN 10 to VLAN 20 Blocked](../../images/07-inter-vlan-acls/02-vlan10-to-vlan20-blocked.png)

<p><sub><strong>Screenshot 030 - VLAN 10 to VLAN 20 Blocked:</strong> PC0 receives destination-unreachable responses when pinging a VLAN 20 host.</sub></p>

![VLAN 20 to VLAN 10 Blocked](../../images/07-inter-vlan-acls/03-vlan20-to-vlan10-blocked.png)

<p><sub><strong>Screenshot 031 - VLAN 20 to VLAN 10 Blocked:</strong> PC2 cannot ping a VLAN 10 host.</sub></p>

![LAN3 Reachability Preserved](../../images/07-inter-vlan-acls/04-lan3-reachability-preserved.png)

<p><sub><strong>Screenshot 032 - LAN3 Reachability Preserved:</strong> A permitted ping reaches 10.10.10.1 outside the blocked VLAN pair.</sub></p>

![Gateway Reachability Preserved](../../images/07-inter-vlan-acls/05-gateway-reachability-preserved.png)

<p><sub><strong>Screenshot 033 - Gateway Reachability Preserved:</strong> VLAN 10 client retains reachability to its local default gateway.</sub></p>

---

## Validation and Summary

The inter-VLAN ACLs are validated by blocked traffic between VLAN 10 and VLAN 20 while gateway and unrelated routed destinations remain reachable. This confirms segmentation without breaking all routed connectivity.

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
