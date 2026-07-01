# Device Identity and Management Foundation

Each device receives a predictable hostname and common management baseline: console behavior, enable secret, authorization banner, line access, disabled domain lookup, and synchronized logging.

## Technical Context

Telnet appears only in the initial learning baseline. It is later replaced with SSH-only VTY access because Telnet exposes credentials and commands in clear text.

> `service password-encryption` provides reversible Cisco Type 7 obfuscation for some stored passwords; it is not a strong password-protection mechanism. For production, use `enable secret`, unique administrator accounts, SSH-only management, centralized TACACS+ or RADIUS, and securely managed credentials.

**Implemented controls:**

- Assigned `SAM-*` hostnames to routers and switches.
- Applied console, VTY, enable-secret, banner, and logging settings.
- Disabled DNS lookup delays caused by unrecognized CLI input.

## Key Technical Terms

| Term | Meaning in this chapter |
|------|-------------------------|
| Hostname | The device name shown in CLI prompts, logs, and remote sessions. It makes multi-device troubleshooting readable. |
| Enable secret | The protected privileged-mode credential used before configuration-level access. It is stronger than a plain `enable password`. |
| VTY line | The virtual terminal line used for remote administrative access such as Telnet or SSH. |
| MOTD banner | A login warning banner that communicates authorized-use expectations before access. |
| Telnet | A clear-text remote access protocol kept only as an early lab step and later replaced with SSH. |

---

## Detailed Walkthrough

### Step 01 - Assign device hostnames

Hostnames identify devices in prompts, logs, SSH keys, and troubleshooting output. The `SAM-R*` and `SAM-S*` naming pattern separates routers from switches.

> Consistent names reduce configuration mistakes when the same command sequence is repeated across many devices.

The same hostname pattern is used on all initial routers and switches; only the device name changes.

| Device role | Hostnames assigned |
|-------------|--------------------|
| Switches | `SAM-S0`, `SAM-S1`, `SAM-S3`, `SAM-S4`, `SAM-S5`, `SAM-S6`, `SAM-S7` |
| Routers | `SAM-R0`, `SAM-R1`, `SAM-R2`, `SAM-R3` |

```cisco
configure terminal
hostname <DEVICE-NAME>
end
write memory
```

`<DEVICE-NAME>` is replaced with the specific hostname from the table. This keeps the command sequence readable while preserving every device name configured in the lab.

---

### Step 02 - Configure initial local access controls

Console and VTY lines receive the lab password, privileged mode receives an enable secret, and the MOTD banner warns against unauthorized use. `exec-timeout 0 30` closes idle sessions, `no ip domain-lookup` prevents mistyped commands from triggering DNS lookups, and `logging synchronous` keeps messages from interrupting command entry.

> The initial `transport input telnet` setting is retained as historical lab evidence. The SSH chapter later replaces it with encrypted management, which is the appropriate operational state.

```cisco
no ip domain-lookup
enable secret Sam1234
service password-encryption
banner motd $
****************************************
* Welcome to Cisco Device.             *
* Authorized Access Only.              *
* This device is the property of -     *
* Samuel Kim!                          *
* Unauthorized access is prohibited!! *
****************************************
$
line console 0
 logging synchronous
 exec-timeout 0 30
 password Samabcd
 login
 exit
line vty 0 4
 transport input telnet
 password Samabcd
 login
 exit
```

---

### Step 03 - Apply the baseline to the routers

The same baseline from Step 02 is applied to SAM-R0 through SAM-R3. Because only the hostname changes, the router implementation is documented with a device-to-hostname table.

> Repeated configuration should remain consistent, but each device still needs individual verification because a missing line on one router can interrupt remote administration or logging.

| Device | Hostname value |
|--------|----------------|
| Router 0 | `SAM-R0` |
| Router 1 | `SAM-R1` |
| Router 2 | `SAM-R2` |
| Router 3 | `SAM-R3` |

`<DEVICE-NAME>` in the Step 02 baseline is replaced with the matching router hostname. Telnet is retained only at this stage and replaced by SSH later.

---

### Step 04 - Apply the baseline to the switches

SAM-S0, SAM-S1, and SAM-S3 through SAM-S7 receive the same management baseline for later VLAN, trunk, Port Security, and EtherChannel work.

> Switch management security is independent of data-plane forwarding. A switch can forward frames while still having incomplete or insecure administrative access.

The switch baseline is also identical except for hostname.

| Device group | Hostname values |
|--------------|-----------------|
| Initial switches | `SAM-S0`, `SAM-S1`, `SAM-S3`, `SAM-S4`, `SAM-S5`, `SAM-S6`, `SAM-S7` |

`<DEVICE-NAME>` in the Step 02 baseline is replaced with each switch hostname. Telnet is retained only at this stage and replaced by SSH later.

---

## Validation and Summary

Validation confirms consistent hostnames, local access behavior, banners, line settings, and logging behavior for later SSH and management chapters.

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
