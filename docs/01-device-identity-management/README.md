# Device Identity and Management Foundation

Each network device receives a predictable hostname and a common baseline configuration. The baseline includes console behavior, an enable secret, an authorization banner, line access, disabled domain lookup, and synchronized logging.

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

### Assign device hostnames

Hostnames identify the device in prompts, logs, SSH keys, and troubleshooting output. The naming pattern separates routers (`SAM-R*`) from switches (`SAM-S*`) and makes multi-device command captures readable.

> Consistent names reduce configuration mistakes when the same command sequence is repeated across many devices.

#### SAM-S0

The switch is assigned the `SAM-S0` hostname so it can be identified consistently in prompts, logs, and remote sessions.

```cisco
configure terminal
hostname SAM-S0
end
write memory
```

#### SAM-S1

The switch is assigned the `SAM-S1` hostname so it can be identified consistently in prompts, logs, and remote sessions.

```cisco
configure terminal
hostname SAM-S1
end
write memory
```

#### SAM-S3

The switch is assigned the `SAM-S3` hostname so it can be identified consistently in prompts, logs, and remote sessions.

```cisco
configure terminal
hostname SAM-S3
end
write memory
```

#### SAM-S4

The switch is assigned the `SAM-S4` hostname so it can be identified consistently in prompts, logs, and remote sessions.

```cisco
configure terminal
hostname SAM-S4
end
write memory
```

#### SAM-S5

The switch is assigned the `SAM-S5` hostname so it can be identified consistently in prompts, logs, and remote sessions.

```cisco
configure terminal
hostname SAM-S5
end
write memory
```

#### SAM-S6

The switch is assigned the `SAM-S6` hostname so it can be identified consistently in prompts, logs, and remote sessions.

```cisco
configure terminal
hostname SAM-S6
end
write memory
```

#### SAM-S7

The switch is assigned the `SAM-S7` hostname so it can be identified consistently in prompts, logs, and remote sessions.

```cisco
configure terminal
hostname SAM-S7
end
write memory
```

#### SAM-R0

The router is assigned the `SAM-R0` hostname so it can be identified consistently in prompts, logs, and remote sessions.

```cisco
configure terminal
hostname SAM-R0
end
write memory
```

#### SAM-R1

The router is assigned the `SAM-R1` hostname so it can be identified consistently in prompts, logs, and remote sessions.

```cisco
configure terminal
hostname SAM-R1
end
write memory
```

#### SAM-R2

The router is assigned the `SAM-R2` hostname so it can be identified consistently in prompts, logs, and remote sessions.

```cisco
configure terminal
hostname SAM-R2
end
write memory
```

#### SAM-R3

The router is assigned the `SAM-R3` hostname so it can be identified consistently in prompts, logs, and remote sessions.

```cisco
configure terminal
hostname SAM-R3
end
write memory
```

### Configure initial local access controls

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

### Apply the baseline to the routers

The same baseline is applied to SAM-R0 through SAM-R3. Each router configuration is written separately and in device order so the complete implementation can be reviewed without reading commands from images.

> Repeated configuration should remain consistent, but each device still needs individual verification because a missing line on one router can interrupt remote administration or logging.

#### SAM-R0

This is the complete initial management baseline for router `SAM-R0`. Telnet is retained only at this stage of the lab and is replaced by SSH later.

```cisco
configure terminal
hostname SAM-R0
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

#### SAM-R1

This is the complete initial management baseline for router `SAM-R1`. Telnet is retained only at this stage of the lab and is replaced by SSH later.

```cisco
configure terminal
hostname SAM-R1
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

#### SAM-R2

This is the complete initial management baseline for router `SAM-R2`. Telnet is retained only at this stage of the lab and is replaced by SSH later.

```cisco
configure terminal
hostname SAM-R2
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

#### SAM-R3

This is the complete initial management baseline for router `SAM-R3`. Telnet is retained only at this stage of the lab and is replaced by SSH later.

```cisco
configure terminal
hostname SAM-R3
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

### Apply the baseline to the switches

SAM-S0, SAM-S1, and SAM-S3 through SAM-S7 receive the same management foundation. The resulting device prompts and saved configurations establish the starting state for VLAN, trunk, Port Security, and EtherChannel work.

> Switch management security is independent of data-plane forwarding. A switch can forward frames while still having incomplete or insecure administrative access.

#### SAM-S0

This is the complete initial management baseline for switch `SAM-S0`. Telnet is retained only at this stage of the lab and is replaced by SSH later.

```cisco
configure terminal
hostname SAM-S0
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

#### SAM-S1

This is the complete initial management baseline for switch `SAM-S1`. Telnet is retained only at this stage of the lab and is replaced by SSH later.

```cisco
configure terminal
hostname SAM-S1
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

#### SAM-S3

This is the complete initial management baseline for switch `SAM-S3`. Telnet is retained only at this stage of the lab and is replaced by SSH later.

```cisco
configure terminal
hostname SAM-S3
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

#### SAM-S4

This is the complete initial management baseline for switch `SAM-S4`. Telnet is retained only at this stage of the lab and is replaced by SSH later.

```cisco
configure terminal
hostname SAM-S4
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

#### SAM-S5

This is the complete initial management baseline for switch `SAM-S5`. Telnet is retained only at this stage of the lab and is replaced by SSH later.

```cisco
configure terminal
hostname SAM-S5
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

#### SAM-S6

This is the complete initial management baseline for switch `SAM-S6`. Telnet is retained only at this stage of the lab and is replaced by SSH later.

```cisco
configure terminal
hostname SAM-S6
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

#### SAM-S7

This is the complete initial management baseline for switch `SAM-S7`. Telnet is retained only at this stage of the lab and is replaced by SSH later.

```cisco
configure terminal
hostname SAM-S7
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

---------

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
