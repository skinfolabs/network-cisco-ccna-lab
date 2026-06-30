# Testing and Verification

- Confirmed six wired DHCP leases and one wireless DHCP lease with the expected gateway and DNS values.
- Confirmed OSPF FULL adjacencies and routed access to the DNS/server LAN.
- Confirmed SSH succeeds from permitted VLAN 10 sources and is denied from VLAN 20.
- Confirmed reciprocal inter-VLAN ACL blocking while unrelated routed destinations remain reachable.
- Confirmed a PAT translation entry for traffic sourced from `172.31.0.0/16`.
- Confirmed all wired and wireless clients can resolve and open `www.sam.com`.
- Confirmed HSRP active and standby roles under normal operation; failover itself was not captured.
- Confirmed Syslog event delivery; the laboratory clock remains unsynchronized.
- Recorded Port Security violation state; the captured ping to `172.168.0.1` is included only as context and is not treated as validation.

---

# Results

The completed lab demonstrates how core CCNA routing and switching features work together in one environment. VLANs and router subinterfaces create the user segments, DHCP and DNS provide client services, OSPF carries routes between the sites, ACLs constrain management and inter-VLAN traffic, and PAT demonstrates source translation toward the server segment.

The later stages add encrypted management, access-port protection, HSRP gateway roles, STP root selection, LACP EtherChannel, and centralized event collection. The project separates configuration evidence from operational validation: command blocks show what was configured, while client results, protocol state, translation tables, and logs show what was actually observed in the lab.

---

# Skills Demonstrated

- Cisco IOS device initialization and management-plane configuration
- VLAN creation, access-port assignment, 802.1Q trunks, and router-on-a-stick
- Router-based DHCP, static server addressing, DNS, and wireless integration
- OSPF single-area routing and route-expansion troubleshooting
- Standard and extended ACL design and validation
- SSH version 2, source-restricted management, and Port Security
- PAT, HSRP, STP root selection, and LACP EtherChannel
- Syslog configuration, evidence interpretation, and production-risk analysis

---

# Evidence and Production Considerations

- Credentials shown in command blocks and screenshots are isolated Packet Tracer laboratory values. They are kept visible for reproducibility and must not be reused on real devices.
- The Packet Tracer `.pkt` file is retained unchanged as the original lab artifact.
- Telnet appears only in the initial learning workflow and is later replaced by SSH. Production VTY access should allow SSH version 2 only, use centralized AAA, and restrict administrative sources.
- `service password-encryption` is Cisco Type 7 obfuscation, not strong password storage. Production designs should use unique administrator identities, strong secrets, TACACS+ or RADIUS, and controlled credential rotation.
- The initial RSA key size is preserved as lab evidence. Production devices should use vendor-supported key sizes of at least 2048 bits and an organization-controlled domain name.
- OSPF passive-interface behavior should be verified with `show ip ospf interface`. Production OSPF should explicitly suppress user-facing adjacencies and consider authentication on trusted transit links.
- Port Security evidence is based on `show port-security` and the recorded violation count. The captured ping to `172.168.0.1` is outside the documented addressing plan and is not used as proof of enforcement.
- ACL 101 is a management-plane source restriction for SSH, not a general firewall policy. The reciprocal VLAN ACLs work because standard ACLs match source addresses and are placed near the destination VLANs.
- PAT is proven by the SAM-R2 translation table for `172.31.0.0/16`. Browser tests from `192.168.10.0/24` and `192.168.20.0/24` validate DNS, routing, ACL permission, and HTTP reachability, but they do not prove PAT.
- HSRP active and standby roles are confirmed with `show standby brief`; actual failover is not claimed because the lab evidence does not show the active router being disabled and SAM-R4 taking over.
- The Syslog service receives messages, but the displayed 1993 date shows that time synchronization is missing. Production investigation depends on reliable NTP and consistent time-zone handling.
- SAM-S4 uses VLAN 1 for the management SVI in this lab. Production management should use a dedicated management VLAN or an out-of-band network.
- The `/16` laboratory LANs, WPA2-Personal WLAN, VLAN 1 management SVI, Telnet history, and combined web/Syslog server should be redesigned before production use.
- The remaining non-CLI visual evidence is indexed in [IMAGE_MANIFEST.md](../../IMAGE_MANIFEST.md).

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
