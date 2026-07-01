# Centralized Syslog Monitoring

The Packet Tracer web server also hosts the lab Syslog service. SAM-R2 enables millisecond log timestamps and sends messages to `172.19.0.200`, giving the lab one server-side view for device events.

## Technical Context

The server receives messages from `172.19.0.1`, confirming the router-to-collector logging path. The 1993 date is an important limitation: logging works, but timestamps are not investigation-ready without reliable time sync.

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

SAM-R2 enables `service timestamps log datetime msec` and configures `logging host 172.19.0.200`. The Syslog service then lists received configuration events, proving that router messages reach the server.

> Co-locating web and logging services is acceptable in this isolated lab. Production monitoring should use a dedicated, hardened collector separated from the application server.

#### SAM-R2

SAM-R2 adds millisecond timestamps and forwards logs to `172.19.0.200`.

```cisco
configure terminal
service timestamps log datetime msec
logging host 172.19.0.200
end
write memory
```

![Syslog Server Events](../../images/11-syslog-monitoring/01-syslog-server-events.png)

<p><sub><strong>Screenshot 049 - Syslog Server Events:</strong> Packet Tracer Syslog service receives SAM-R2 configuration events with the unsynchronized lab date.</sub></p>

This confirms event delivery and records the clock limitation, so Syslog is treated as logging-path validation rather than a production-ready incident timeline.

---

## Validation and Summary

Validation confirms server-side event collection from SAM-R2. The unsynchronized Packet Tracer date remains a production concern because investigations depend on accurate time.

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
