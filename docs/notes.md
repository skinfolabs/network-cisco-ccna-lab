# Implementation Notes

## Evidence scope

The README is based on the complete supplied PDF evidence. The original Packet Tracer `.pkt` file is included unchanged as a reusable lab artifact and was not independently inspected at the user's request.

The README reproduces the demonstrated Cisco IOS settings as readable command blocks. Client results, protocol output, translation tables, and logs are retained as stronger evidence of observed behavior. No result is claimed beyond what the supplied project evidence demonstrates.

## Laboratory credentials

The project intentionally displays and retains the following isolated Packet Tracer values so the laboratory configuration remains reproducible:

- console and initial Telnet password: `Samabcd`;
- enable secret: `Sam1234`;
- SSH user: `Sam` with laboratory secret `Samabcd`;
- wireless pre-shared key: `Abcd1234`.

These values are deliberately open laboratory data, not exposed real-world credentials. They must never be reused on real devices.

## Telnet and stored passwords

Telnet appears during the initial device configuration and is later replaced by SSH-only VTY access. Telnet provides no confidentiality and should not remain enabled in production. A production baseline should permit only SSH version 2 on VTY lines and restrict administrative access through ACLs and centralized AAA.

`service password-encryption` applies reversible Type 7 obfuscation to supported plaintext configuration fields. It is useful only for preventing casual observation and is not equivalent to secure password storage. Production devices should use `enable secret`, individual administrator identities, TACACS+ or RADIUS, strong unique credentials, and a controlled credential-management process.

## SSH key strength and identity

The initial devices generate 1024-bit RSA keys. SAM-R4 and SAM-S8 use 2024-bit keys in the source. A production baseline should use a vendor-supported key size of at least 2048 bits, a real administrative domain name, centralized AAA, and individual administrator accounts.

## OSPF hardening

The source uses passive-interface commands on physical interfaces while some user networks exist on subinterfaces. A production configuration should verify the effective passive state with `show ip ospf interface` and explicitly suppress adjacencies on every user-facing subinterface. OSPF authentication should also be considered on trusted transit links.

The first DNS-server ping loses one packet before receiving three replies. In Packet Tracer this commonly occurs while ARP is resolved; the result demonstrates reachability but is not described as lossless.

## Port Security validation

The retained ping test targets `172.168.0.1`, which does not belong to the documented addressing plan. Its timeout is not used as proof that Port Security worked. The relevant evidence is the `show port-security` output and recorded violation count.

Port Security is a Layer 2 control tied to a switch port and learned MAC address. Production access control should also consider 802.1X, NAC, physical port protection, DHCP snooping, Dynamic ARP Inspection, and monitoring.

## ACL behavior

ACL 101 restricts SSH by source subnet and then explicitly permits all other IP traffic. It should be understood as a management-plane filter, not as a general traffic firewall.

The reciprocal standard ACLs block traffic between VLAN 10 and VLAN 20 while allowing other destinations. Because standard ACLs match only the source address, their outbound placement near the destination VLAN is important.

## PAT evidence

SAM-R2 translates only sources matching `172.31.0.0/16`. The translation table confirms this behavior for `172.31.0.1`. Browser tests from `192.168.10.0/24` and `192.168.20.0/24` validate DNS, routing, ACL permission, and HTTP service availability, but they do not validate PAT.

## HSRP validation limits

`show standby brief` confirms SAM-R3 as active and SAM-R4 as standby for both groups. The source does not show the active router being disabled or SAM-R4 becoming active, so actual failover is not claimed.

A production HSRP design should track upstream interfaces or route reachability. Otherwise, a router can remain active while a different dependency has failed.

## Syslog time accuracy

The Syslog service receives messages, but the displayed date is in 1993. Reliable incident investigation requires synchronized time. Production routers, switches, servers, and collectors should use a trusted NTP hierarchy and consistent time-zone handling.

## Management SVI

SAM-S4 uses interface VLAN 1 with address `172.19.0.254/16`. VLAN 10 is the permitted SSH source network; it is not the switch management VLAN. Production management should move the SVI to a dedicated management VLAN or out-of-band network.

## Production design considerations

- Replace broad `/16` LAN segments with subnets sized for actual capacity and broadcast requirements.
- Separate web and Syslog roles instead of combining them on one server.
- Replace WPA2-Personal with WPA2/WPA3-Enterprise and centralized authentication.
- Use dedicated infrastructure and management VLANs rather than VLAN 1.
- Add explicit device backups, AAA accounting, SNMPv3 or telemetry, NTP, and protected logging transport.
- Test HSRP failover and OSPF convergence under controlled outage conditions before production deployment.
