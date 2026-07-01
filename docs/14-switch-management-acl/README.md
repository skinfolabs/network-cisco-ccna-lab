# Source-Restricted Switch Management

SAM-S4 receives a management SVI at `172.19.0.254/16` on VLAN 1 and gateway `172.19.0.1`. The earlier SSH ACL on SAM-R0 permits management from VLAN 10 and denies it from VLAN 20.

## Technical Context

This is a management-plane control, not a general VLAN 20 block. It restricts SSH by source subnet; the switch management SVI itself remains on VLAN 1.

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

SAM-S4 uses interface VLAN 1 with `172.19.0.254/16` and gateway `172.19.0.1`. The SVI provides the SSH target, and the default gateway lets the switch return traffic to remote clients.

> A Layer 2 switch needs an SVI address for management and an IP default gateway to return traffic to administrators outside the local subnet.

#### SAM-S4

SAM-S4 receives an SVI for remote management and a default gateway for off-subnet replies.

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

PC0 in VLAN 10 authenticates to `172.19.0.254`, while PC2 in VLAN 20 times out. The paired tests prove the policy from both sides: allowed administrative source and denied non-administrative source.

> Allowed and denied tests together provide stronger evidence than the ACL configuration alone because they show the observed client behavior on both sides of the policy.

![VLAN 10 Switch SSH Allowed](../../images/12-switch-management-acl/01-vlan10-switch-ssh-allowed.png)

<p><sub><strong>Screenshot 050 - VLAN 10 Switch SSH Allowed:</strong> VLAN 10 client successfully opens an SSH session to 172.19.0.254.</sub></p>

The successful session confirms the management address, return path, SSH service, credentials, and allow rule.

![VLAN 20 Switch SSH Denied](../../images/12-switch-management-acl/02-vlan20-switch-ssh-denied.png)

<p><sub><strong>Screenshot 051 - VLAN 20 Switch SSH Denied:</strong> VLAN 20 client receives a timeout when attempting the same management connection.</sub></p>

The denied test confirms selectivity: VLAN 20 cannot manage the switch over SSH while VLAN 10 remains allowed.

---

## Validation and Summary

Validation confirms source-restricted switch management with one allowed VLAN 10 SSH session and one denied VLAN 20 attempt.

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
