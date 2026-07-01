# Device Identity and Management Foundation

Each network device receives a predictable hostname and a common baseline configuration. The baseline includes console behavior, an enable secret, an authorization banner, line access, disabled domain lookup, and synchronized logging.

## Technical Context

Telnet appears in the initial lab configuration as a learning step. It is later replaced with SSH-only VTY access because Telnet exposes credentials and commands in clear text.

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

Hostnames identify the device in prompts, logs, SSH keys, and troubleshooting output. The naming pattern separates routers (`SAM-R*`) from switches (`SAM-S*`) and makes multi-device command captures readable.

> Consistent names reduce configuration mistakes when the same command sequence is repeated across many devices.

The same hostname command pattern is used on all initial routers and switches. Only the hostname value changes per device, so the configuration is documented once and mapped to the exact devices below.

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

Console and VTY lines receive the laboratory password, privileged access receives an enable secret, and the MOTD banner warns against unauthorized use. `exec-timeout 0 30` closes an unattended session after 30 seconds, reducing the risk of leaving administrative access open. `no ip domain-lookup` prevents the device from treating a mistyped command as a hostname and waiting for an unnecessary DNS lookup, while `logging synchronous` keeps system messages from interrupting command entry.

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

The same baseline is applied to SAM-R0 through SAM-R3. Because the commands are identical except for the hostname, the implementation is documented as one router template with a device-to-hostname table.

> Repeated configuration should remain consistent, but each device still needs individual verification because a missing line on one router can interrupt remote administration or logging.

The router baseline is identical on `SAM-R0`, `SAM-R1`, `SAM-R2`, and `SAM-R3`; the only per-device value is the hostname. The command block below is the complete baseline template used for those routers.

| Device | Hostname value |
|--------|----------------|
| Router 0 | `SAM-R0` |
| Router 1 | `SAM-R1` |
| Router 2 | `SAM-R2` |
| Router 3 | `SAM-R3` |

```cisco
configure terminal
hostname <ROUTER-NAME>
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
end
write memory
```

`<ROUTER-NAME>` is replaced with the matching router hostname from the table. Telnet is retained only at this stage of the lab and is replaced by SSH later.

---

### Step 04 - Apply the baseline to the switches

SAM-S0, SAM-S1, and SAM-S3 through SAM-S7 receive the same management foundation. The resulting device prompts and saved configurations establish the starting state for VLAN, trunk, Port Security, and EtherChannel work.

> Switch management security is independent of data-plane forwarding. A switch can forward frames while still having incomplete or insecure administrative access.

The switch baseline is identical on the initial access and distribution switches; the only per-device value is the hostname. The command block below is the complete baseline template used for `SAM-S0`, `SAM-S1`, and `SAM-S3` through `SAM-S7`.

| Device group | Hostname values |
|--------------|-----------------|
| Initial switches | `SAM-S0`, `SAM-S1`, `SAM-S3`, `SAM-S4`, `SAM-S5`, `SAM-S6`, `SAM-S7` |

```cisco
configure terminal
hostname <SWITCH-NAME>
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
end
write memory
```

`<SWITCH-NAME>` is replaced with the matching switch hostname from the table. Telnet is retained only at this stage of the lab and is replaced by SSH later.

---

## Validation and Summary

Device identity and the baseline management configuration are validated through the recorded IOS command blocks. The chapter establishes consistent hostnames, local access behavior, banners, line settings, and logging behavior that later management and SSH chapters build on.

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
