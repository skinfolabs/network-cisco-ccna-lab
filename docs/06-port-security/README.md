# Access-Layer Port Security

SAM-S4 represents the server-room access switch. Unused interfaces are disabled, and active access ports use sticky MAC learning with a one-address limit and restrict mode.

## Technical Context

Restrict mode drops unauthorized MAC frames and increments the violation counter without shutting down the port, making violations observable while preserving the learned endpoint.

> Port Security limits attachment at one switch port; it does not authenticate a user and should complement 802.1X, physical security, and monitoring in production.

**Implemented controls:**

- Shut down unused access and uplink interfaces.
- Enabled one-address sticky Port Security on the used access ports.
- Recorded the configured restrict action and violation count.

## Key Technical Terms

| Term | Meaning in this chapter |
|------|-------------------------|
| Port Security | A switch feature that limits which MAC addresses may use an access port. |
| Sticky MAC | A learned MAC address saved into the running configuration as the allowed address for that port. |
| Restrict mode | A violation action that drops unauthorized frames and increments a counter without shutting down the port. |
| Violation counter | Operational evidence that the switch observed traffic from an unauthorized MAC address. |
| Shutdown unused ports | A hardening step that reduces the chance of an unused wall jack becoming an easy network entry point. |

---

## Detailed Walkthrough

### Step 01 - Disable unused interfaces and enable sticky learning

Unused SAM-S4 ports are shut down, while three active interfaces are placed in access mode and protected. Sticky learning stores the first observed source MAC.

> An overly broad interface range can disable legitimate uplinks. Interface selections must be checked against the physical topology before applying a shutdown command.

#### SAM-S4

SAM-S4 shuts down unused interfaces and protects three active access ports with sticky MAC learning, a one-address limit, and restrict mode.

```cisco
configure terminal
interface range FastEthernet0/3 - 24, GigabitEthernet0/2
 shutdown
 exit
interface range FastEthernet0/1 - 2, GigabitEthernet0/1
 switchport mode access
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation restrict
end
write memory
```

The operational table records one learned address on each secured port and eight violations on FastEthernet0/2.

```text
SAM-S4# show port-security
Secure Port  MaxSecureAddr  CurrentAddr  SecurityViolation  Security Action
Fa0/1        1              1            0                  Restrict
Fa0/2        1              1            8                  Restrict
Gi0/1        1              1            0                  Restrict
```

---

### Step 02 - Review Port Security state and limitations

`show port-security` reports protected interfaces, one learned address per interface, restrict mode, and eight violations on one port. The captured ping targets `172.168.0.1`, which is outside the documented topology, so the timeout is not treated as enforcement proof.

> The violation counter is the relevant control evidence. A generic timeout can also result from an incorrect address, missing route, or unavailable endpoint.

![Initial Port Security Ping Test](../../images/04-port-security/01-initial-port-security-ping.png)

<p><sub><strong>Screenshot 018 - Initial Port Security Ping Test:</strong> The captured test targets 172.168.0.1; this address is outside the documented topology and is used only as limited context.</sub></p>

![Server Room Access Topology](../../images/04-port-security/02-server-room-access-topology.png)

<p><sub><strong>Screenshot 019 - Server Room Access Topology:</strong> Laptop and server connections shown around the protected SAM-S4 access switch.</sub></p>

---

## Validation and Summary

Validation confirms unused-port shutdown and sticky MAC learning. Because the captured ping target is outside the lab addressing plan, the port-security state is the reliable evidence.

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
