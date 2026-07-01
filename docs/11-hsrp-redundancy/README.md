# HSRP Gateway Redundancy

SAM-R4 and SAM-S8 extend the topology so SAM-R3 and SAM-R4 can share virtual gateways on `192.168.13.0/24` and `10.10.10.0/24`. OSPF advertises the new router, and HSRP priorities make SAM-R3 active while SAM-R4 remains standby.

## Technical Context

Clients use the virtual IP instead of either router's physical address. Preemption lets the preferred higher-priority router reclaim the active role after returning.

> The captured outputs validate HSRP roles and normal-path connectivity, but they do not show SAM-R3 being disabled and SAM-R4 becoming active. The project therefore documents configured redundancy without claiming a completed failover test.

**Implemented controls:**

- Added and secured SAM-R4 and SAM-S8.
- Advertised the redundant path through OSPF.
- Configured two HSRP groups with active and standby priorities.

## Key Technical Terms

| Term | Meaning in this chapter |
|------|-------------------------|
| HSRP | Hot Standby Router Protocol, which lets routers share a virtual default gateway. |
| Virtual IP | The gateway address used by clients instead of a physical router interface. |
| Active router | The router currently forwarding traffic for the HSRP virtual gateway. |
| Standby router | The router waiting to take over if the active router fails. |
| Preemption | A setting that allows the preferred router to reclaim the active role after it returns. |

---

## Detailed Walkthrough

### Step 01 - Add the redundant router and switch

The expanded topology adds SAM-R4 as an alternate gateway and SAM-S8 as an additional switch. Both receive the same management and SSH baseline used elsewhere.

> HSRP provides router gateway redundancy. It does not make a switch a standby device; Layer 2 path selection is handled separately by STP and EtherChannel.

![Expanded HSRP Topology](../../images/09-hsrp-redundancy/01-expanded-hsrp-topology.png)

<p><sub><strong>Screenshot 042 - Expanded HSRP Topology:</strong> Router4 and Switch8 added to provide an alternate routed path for the two shared LANs.</sub></p>

The same management baseline is applied to both new devices; only the hostname changes.

| New device | Hostname value |
|------------|----------------|
| Router 4 | `SAM-R4` |
| Switch 8 | `SAM-S8` |

The full common baseline is documented in [Device Identity and Management Foundation](../02-device-identity-management/README.md). Here, `<DEVICE-NAME>` is replaced with `SAM-R4` or `SAM-S8`, then VTY access is hardened with SSH.

Both new devices receive the stronger SSH baseline with a 2048-bit RSA key request.

```cisco
configure terminal
ip domain-name SSH
crypto key generate rsa
! Enter 2048 when IOS requests the modulus size.
username Sam secret Samabcd
line vty 0 4
 transport input ssh
 login local
 exit
ip ssh version 2
end
write memory
```

---

### Step 02 - Address SAM-R4 and advertise its networks

SAM-R4 receives `10.10.10.2/24` on LAN3 and `192.168.13.3/24` on the transit network. OSPF advertises both, and a ping confirms reachability.

> Routing must be operational before HSRP can provide useful end-to-end redundancy; a virtual gateway cannot compensate for a missing upstream route.

#### SAM-R4

SAM-R4 addresses both shared networks and advertises them in OSPF area 0 before HSRP.

```cisco
configure terminal
interface GigabitEthernet0/0/0
 ip address 10.10.10.2 255.255.255.0
 no shutdown
 exit
interface GigabitEthernet0/0/1
 ip address 192.168.13.3 255.255.255.0
 no shutdown
 exit
router ospf 1
 network 10.10.10.0 0.0.0.255 area 0
 network 192.168.13.0 0.0.0.255 area 0
 passive-interface GigabitEthernet0/0/0
end
write memory
```

![SAM-R4 Ping Validation](../../images/09-hsrp-redundancy/02-sam-r4-ping-validation.png)

<p><sub><strong>Screenshot 043 - SAM-R4 Ping Validation:</strong> PC0 successfully pings the new SAM-R4 transit address.</sub></p>

---

### Step 03 - Configure the HSRP virtual gateways

HSRP group 1 uses virtual IP `192.168.13.222`, and group 2 uses `10.10.10.222`. SAM-R3 uses priority 150, SAM-R4 uses priority 100, and both have preemption enabled.

> Production HSRP commonly tracks uplink or routing health. Without tracking, an active router can retain the gateway role even when a different upstream dependency fails.

![HSRP Router Pair](../../images/09-hsrp-redundancy/03-hsrp-router-pair.png)

<p><sub><strong>Screenshot 044 - HSRP Router Pair:</strong> SAM-R3 and SAM-R4 share virtual gateway addresses across both connected networks.</sub></p>

#### SAM-R3

SAM-R3 uses priority 150 and preemption, making it the preferred active gateway for both groups.

```cisco
configure terminal
interface GigabitEthernet0/0/0
 standby 1 ip 192.168.13.222
 standby 1 priority 150
 standby 1 preempt
 exit
interface GigabitEthernet0/0/1
 standby 2 ip 10.10.10.222
 standby 2 priority 150
 standby 2 preempt
end
write memory
```

#### SAM-R4

SAM-R4 uses priority 100 and remains standby for the same two virtual IPs.

```cisco
configure terminal
interface GigabitEthernet0/0/1
 standby 1 ip 192.168.13.222
 standby 1 priority 100
 standby 1 preempt
 exit
interface GigabitEthernet0/0/0
 standby 2 ip 10.10.10.222
 standby 2 priority 100
 standby 2 preempt
end
write memory
```

---

### Step 04 - Validate active and standby state

PC6 uses the LAN3 virtual IP as its default gateway. `show standby brief` shows SAM-R3 active and SAM-R4 standby for both groups, while ping and traceroute show normal connectivity.

> These checks prove role election and current reachability. A complete failover test would also shut down the active path and capture SAM-R4 taking over.

![PC6 Virtual Gateway](../../images/09-hsrp-redundancy/04-pc6-virtual-gateway.png)

<p><sub><strong>Screenshot 045 - PC6 Virtual Gateway:</strong> PC6 uses 10.10.10.222 as its HSRP default gateway.</sub></p>

#### HSRP role verification

The operational output places SAM-R3 active and SAM-R4 standby for both virtual gateways.

```text
SAM-R4# show standby brief
Interface  Grp  Pri  P  State    Active        Standby  Virtual IP
Gi0/0/0    2    100  P  Standby  10.10.10.1    local    10.10.10.222
Gi0/0/1    1    100  P  Standby  192.168.13.2  local    192.168.13.222

SAM-R3# show standby brief
Interface  Grp  Pri  P  State   Active  Standby       Virtual IP
Gi0/0/0    1    150  P  Active  local   192.168.13.3  192.168.13.222
Gi0/0/1    2    150  P  Active  local   10.10.10.2    10.10.10.222
```

![PC6 Routed Path Validation](../../images/09-hsrp-redundancy/05-pc6-routed-path-validation.png)

<p><sub><strong>Screenshot 046 - PC6 Routed Path Validation:</strong> PC6 ping and traceroute test normal routed connectivity through the active gateway.</sub></p>

![PC4 Routed Path Validation](../../images/09-hsrp-redundancy/06-pc4-routed-path-validation.png)

<p><sub><strong>Screenshot 047 - PC4 Routed Path Validation:</strong> A second client validates normal connectivity across the HSRP-enabled topology.</sub></p>

---

## Validation and Summary

Validation confirms virtual gateway configuration and active/standby roles. Failover is not claimed because the evidence shows normal state, not a captured failure test.

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
