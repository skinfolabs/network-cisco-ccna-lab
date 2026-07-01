# Server, DNS, and Wireless Services

LAN2 hosts the DNS and web servers on static addresses so clients and network services can depend on stable endpoints. DNS maps `www.sam.com` to the web server, while the wireless controller and lightweight access point extend VLAN 10 connectivity to a laptop.

## Technical Context

The wireless workflow uses WPA2-Personal in the isolated lab and confirms that the laptop receives the same gateway and DNS information as wired VLAN 10 clients.

> A production wireless network should normally use WPA2/WPA3-Enterprise with centralized authentication, separate infrastructure management, and stronger key lifecycle controls than a shared PSK.

**Implemented controls:**

- Assigned static addressing to DNS and web services.
- Published the web server through an internal DNS A record.
- Built and validated the `SamNet` wireless client path.

**Lab values used in this chapter:**

| Component | Address or value |
|-----------|------------------|
| DNS server | `172.19.0.100/16`, gateway `172.19.0.1` |
| Web server | `172.19.0.200/16`, gateway `172.19.0.1`, DNS `172.19.0.100` |
| DNS A record | `www.sam.com` -> `172.19.0.200` |
| WLAN | `SamNet` |
| WLAN security | WPA2-Personal with the laboratory PSK `Abcd1234` |
| Wireless client | DHCP lease in `192.168.10.0/24` |

## Key Technical Terms

| Term | Meaning in this chapter |
|------|-------------------------|
| Static server address | A fixed IP address used by infrastructure services so clients and DNS records always point to the same host. |
| DNS A record | A DNS record that maps a hostname such as `www.sam.com` to an IPv4 address. |
| SSID | The wireless network name presented to clients. |
| WPA2-Personal | A pre-shared-key wireless security mode suitable for a lab, but weaker than centralized enterprise authentication. |
| Lightweight AP | An access point managed by a wireless controller instead of being configured independently. |

---

## Detailed Walkthrough

### Step 01 - Configure static server addressing

The DNS server uses `172.19.0.100/16` and the web server uses `172.19.0.200/16`; both use `172.19.0.1` as their gateway. The web server points to the DNS server so the same resolver can be used for service testing.

> Static service addresses prevent a DNS record or logging destination from becoming stale after a lease change.

![DNS Server Static Address](../../images/03-server-dns-wireless/01-dns-server-static-address.png)

<p><sub><strong>Screenshot 008 - DNS Server Static Address:</strong> DNS server configured as 172.19.0.100/16 with gateway 172.19.0.1.</sub></p>

![Web Server Static Address](../../images/03-server-dns-wireless/02-web-server-static-address.png)

<p><sub><strong>Screenshot 009 - Web Server Static Address:</strong> Web server configured as 172.19.0.200/16 and pointed to the laboratory DNS server.</sub></p>

---

### Step 02 - Publish the internal web name

Packet Tracer DNS is enabled and an A record maps `www.sam.com` directly to `172.19.0.200`. An A record stores an IPv4 address; it is not a CNAME alias.

> DNS resolution proves name-to-address mapping. The browser tests later confirm that the HTTP service is reachable at the resolved address.

![Web DNS A Record](../../images/03-server-dns-wireless/03-web-dns-a-record.png)

<p><sub><strong>Screenshot 010 - Web DNS A Record:</strong> A record maps www.sam.com to 172.19.0.200.</sub></p>

---

### Step 03 - Configure the WLAN profile and obtain a lease

The `SamNet` WLAN uses WPA2-Personal and places the wireless client in the VLAN 10 addressing domain. The laptop receives `192.168.10.5`, gateway `192.168.10.1`, and DNS server `172.19.0.100`.

> The controller profile defines wireless authentication and client forwarding; the wired trunk and DHCP configuration still determine whether the client can reach the routed network.

![SamNet WLAN Profile](../../images/03-server-dns-wireless/04-samnet-wlan-profile.png)

<p><sub><strong>Screenshot 011 - SamNet WLAN Profile:</strong> Wireless profile configured for SamNet with WPA2-Personal and the management setting shown in Packet Tracer.</sub></p>

![Wireless Client DHCP Lease](../../images/03-server-dns-wireless/05-wireless-client-dhcp-lease.png)

<p><sub><strong>Screenshot 012 - Wireless Client DHCP Lease:</strong> Laptop receives 192.168.10.5, the VLAN 10 gateway, and DNS settings over WLAN.</sub></p>

---

### Step 04 - Build and validate the wireless client path

FlexConnect options are enabled, a compatible wireless module is installed in the laptop, and the client discovers and joins SamNet through the lightweight access point. The access-point infrastructure address is excluded from the DHCP pool, and the switch links toward the access point and controller are configured as trunks.

> The visible pre-shared key is retained as laboratory data only. A real key must not be committed to a public repository.

![FlexConnect Settings](../../images/03-server-dns-wireless/06-flexconnect-settings.png)

<p><sub><strong>Screenshot 013 - FlexConnect Settings:</strong> Local switching and local authentication enabled for the lightweight access-point workflow.</sub></p>

![Laptop Wireless Module](../../images/03-server-dns-wireless/07-laptop-wireless-module.png)

<p><sub><strong>Screenshot 014 - Laptop Wireless Module:</strong> Compatible wireless module installed in the Packet Tracer laptop.</sub></p>

![Wireless Client and Access Point](../../images/03-server-dns-wireless/08-wireless-client-and-ap.png)

<p><sub><strong>Screenshot 015 - Wireless Client and Access Point:</strong> Laptop associated through the lightweight access point.</sub></p>

![SamNet Discovery](../../images/03-server-dns-wireless/09-samnet-discovery.png)

<p><sub><strong>Screenshot 016 - SamNet Discovery:</strong> Laptop detects the SamNet WLAN with WPA2-PSK security.</sub></p>

![SamNet WPA2 Authentication](../../images/03-server-dns-wireless/10-samnet-wpa2-authentication.png)

<p><sub><strong>Screenshot 017 - SamNet WPA2 Authentication:</strong> Laboratory pre-shared key entered to join the wireless network.</sub></p>

#### SAM-R0

The intended access-point address is excluded from DHCP so it remains available for infrastructure use.

```cisco
configure terminal
ip dhcp excluded-address 192.168.10.129
end
write memory
```

#### SAM-S0

SAM-S0 statically trunks the access-point connection and uses VLAN 99 as the native VLAN.

```cisco
configure terminal
interface FastEthernet0/3
 switchport mode trunk
 switchport trunk native vlan 99
 switchport nonegotiate
end
write memory
```

#### SAM-S1

SAM-S1 applies the matching trunk policy to the wireless-controller connection.

```cisco
configure terminal
interface GigabitEthernet0/2
 switchport mode trunk
 switchport trunk native vlan 99
 switchport nonegotiate
end
write memory
```

---

## Validation and Summary

Static server addressing, DNS publishing, WLAN configuration, and wireless client evidence validate the service layer. The wired server network and wireless client path both become usable dependencies for browser, DNS, and reachability testing later in the lab.

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
