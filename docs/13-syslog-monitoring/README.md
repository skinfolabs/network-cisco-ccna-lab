# Centralized Syslog Monitoring

The Packet Tracer web server also hosts the Syslog service for this laboratory. Syslog gives network devices a central place to send event messages instead of keeping every important event only on the local device console. In this lab, SAM-R2 enables millisecond log timestamps and sends messages to `172.19.0.200`, allowing configuration events to be reviewed from one server-side view.

## Technical Context

The server receives messages from `172.19.0.1`, confirming the remote logging path from the router to the collector. The displayed 1993 date is still an important finding: logging works, but the simulated devices are not synchronized to a reliable time source, so the timestamps should not be treated as investigation-ready.

> Central logs lose investigative value when clocks disagree. Production devices and collectors should use authenticated NTP where available, consistent time zones, protected transport, and a dedicated retention policy.

**Implemented controls:**

- Enabled detailed timestamps on SAM-R2 logs.
- Defined the remote Syslog host.
- Verified event arrival on the Packet Tracer service.

## Key Technical Terms

| Term | Meaning in this chapter |
|------|-------------------------|
| Syslog | A standard method for sending device events to a central logging server. |
| Logging host | The server IP address where a network device sends Syslog messages. |
| Timestamps | Time markers on log entries; reliable incident analysis requires accurate NTP-backed time. |
| Event delivery | The validation point showing that messages leave the device and appear on the server. |

---

## Detailed Walkthrough

### Step 01 - Send SAM-R2 events to the Syslog server

SAM-R2 confirms reachability to the server, enables `service timestamps log datetime msec`, and configures `logging host 172.19.0.200`. The timestamp command makes device logs more useful by adding date and millisecond detail, while `logging host` defines the remote collector. The Syslog service then lists the received configuration events, showing that router messages are leaving the network device and arriving at the server.

> Co-locating web and logging services is acceptable in this isolated lab. Production monitoring should use a dedicated, hardened collector separated from the application server.

#### SAM-R2

SAM-R2 adds millisecond timestamps and forwards its log messages to the Packet Tracer Syslog service at `172.19.0.200`.

```cisco
configure terminal
service timestamps log datetime msec
logging host 172.19.0.200
end
write memory
```

![Syslog Server Events](../../images/11-syslog-monitoring/01-syslog-server-events.png)

<p><sub><strong>Screenshot 049 - Syslog Server Events:</strong> Packet Tracer Syslog service receives SAM-R2 configuration events with the unsynchronized lab date.</sub></p>

This evidence confirms event delivery to the Syslog service. It also records the clock limitation, which is why the project treats Syslog as a successful logging-path validation but not as a production-ready incident timeline.

---

## Validation and Summary

Syslog is validated by server-side event collection from SAM-R2. The unsynchronized Packet Tracer date is documented as a production concern because reliable investigations depend on NTP and consistent time handling.

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
