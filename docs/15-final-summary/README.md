# Final Summary

This Cisco Packet Tracer lab shows how CCNA routing, switching, management, and security controls work together in one enterprise-style topology. VLANs and router subinterfaces create user segments, DHCP and DNS provide client services, OSPF carries routes, ACLs constrain traffic, and PAT demonstrates address translation for the selected source network.

Later chapters add encrypted management, access-port protection, HSRP gateway roles, STP root selection, LACP EtherChannel, and centralized event collection. Command blocks show configuration, while client results, protocol state, translation tables, browser tests, and Syslog messages show observed behavior.

## Validation Summary

| Area | Validation captured |
|------|---------------------|
| DHCP | Six wired DHCP leases and one wireless DHCP lease received expected addressing, gateway, and DNS values |
| DNS and HTTP | Wired and wireless clients resolved and opened `www.sam.com` |
| OSPF | FULL adjacencies and routed access to the DNS/server LAN were confirmed |
| SSH management | VLAN 10 sources succeeded, while VLAN 20 sources were denied by management ACLs |
| Inter-VLAN ACLs | Reciprocal VLAN 10 and VLAN 20 blocking was confirmed while unrelated routed destinations remained reachable |
| PAT | SAM-R2 showed a PAT translation entry for traffic sourced from `172.31.0.0/16` |
| HSRP | Active and standby gateway roles were confirmed with normal-state evidence |
| Port Security | Violation state was recorded; the unrelated `172.168.0.1` ping is kept only as context |
| STP and EtherChannel | Redundant switching and bundled-link configuration were documented |
| Syslog | The Syslog server received SAM-R2 events, with unsynchronized lab time noted |

## Production Recommendations

- Keep the visible usernames and passwords only as isolated Packet Tracer lab values; real environments need unique credentials, secret management, and credential rotation.
- Replace the initial Telnet workflow with SSHv2, centralized AAA, and source-restricted administration.
- Treat `service password-encryption` as obfuscation, not strong password storage. Use strong secrets, TACACS+ or RADIUS, and role-based administration in production.
- Use vendor-supported RSA keys of at least 2048 bits and an organization-controlled domain name for SSH.
- Verify OSPF passive interfaces with `show ip ospf interface`; suppress user-facing adjacencies and consider authentication on trusted transit links.
- Use a dedicated management VLAN or out-of-band management network instead of relying on VLAN 1 for switch management.
- Add reliable NTP and consistent time-zone handling before treating Syslog data as investigation-ready evidence.
- Redesign `/16` lab LANs, WPA2-Personal WLAN, combined web/Syslog service, and simplified Packet Tracer assumptions before production use.
- Validate HSRP failover with an actual failure test before claiming operational gateway failover.

## Skills Demonstrated

- Cisco IOS device initialization and management-plane configuration
- VLAN creation, access-port assignment, 802.1Q trunking, and router-on-a-stick routing
- Router-based DHCP, static server addressing, DNS, and wireless integration
- OSPF single-area routing and route-expansion troubleshooting
- Standard and extended ACL design and validation
- SSH version 2, source-restricted management, and Port Security
- PAT, HSRP, STP root selection, and LACP EtherChannel
- Syslog configuration, evidence interpretation, and production-risk analysis

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
| 15 | [Final Summary](README.md) |
