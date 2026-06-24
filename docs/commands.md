# Command Reference

This reference consolidates the Cisco IOS commands demonstrated in the project. The source material often uses valid IOS abbreviations; the commands below use expanded syntax where it improves readability without changing the configured values.

## Device baseline

The baseline was repeated across the routers and switches, with each device receiving its own `SAM-*` hostname.

```cisco
hostname <DEVICE_NAME>
no ip domain-lookup

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

end
write memory
```

Telnet is part of the initial laboratory sequence only. The SSH configuration later replaces it on the VTY lines.

## VLAN and access-port configuration

```cisco
vlan 10
 name YELLOW
vlan 20
 name BLUE
vlan 99
 name Native
```

Representative access-port assignments:

```cisco
interface FastEthernet0/5
 switchport mode access
 switchport access vlan 20

interface range FastEthernet0/1 - 2
 switchport mode access
 switchport access vlan 10
```

The source screenshots document the individual SAM-S0 and SAM-S1 assignments for both user VLANs.

## Static trunk configuration

```cisco
interface <TRUNK_INTERFACE>
 switchport mode trunk
 switchport trunk native vlan 99
 switchport nonegotiate
```

Validation:

```cisco
show interfaces trunk
```

## Router-on-a-stick and DHCP

VLAN 10 gateway:

```cisco
interface GigabitEthernet0/0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
```

VLAN 20 gateway:

```cisco
interface GigabitEthernet0/0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
```

DHCP pools:

```cisco
ip dhcp pool VLAN10
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 172.19.0.100
 exit
ip dhcp excluded-address 192.168.10.1

ip dhcp pool VLAN20
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
 dns-server 172.19.0.100
 exit
ip dhcp excluded-address 192.168.20.1

ip dhcp excluded-address 192.168.10.129
```

## Server and wireless values

| Component | Address or value |
|-----------|------------------|
| DNS server | `172.19.0.100/16`, gateway `172.19.0.1` |
| Web server | `172.19.0.200/16`, gateway `172.19.0.1`, DNS `172.19.0.100` |
| DNS A record | `www.sam.com` -> `172.19.0.200` |
| WLAN | `SamNet` |
| WLAN security | WPA2-Personal, laboratory PSK `Abcd1234` |
| Wireless client | DHCP in `192.168.10.0/24` |

## Port Security

Disable unused SAM-S4 ports:

```cisco
interface range FastEthernet0/3 - 24, GigabitEthernet0/2
 shutdown
```

Protect the used access ports:

```cisco
interface range FastEthernet0/1 - 2, GigabitEthernet0/1
 switchport mode access
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation restrict
```

Validation:

```cisco
show port-security
```

## OSPF and routed interfaces

SAM-R0 to SAM-R1:

```cisco
interface GigabitEthernet0/0/1
 ip address 172.31.0.1 255.255.0.0
 no shutdown

router ospf 1
 network 192.168.10.0 0.0.0.255 area 0
 network 192.168.20.0 0.0.0.255 area 0
 network 172.31.0.0 0.0.255.255 area 0
 passive-interface GigabitEthernet0/0/0
```

SAM-R1:

```cisco
interface GigabitEthernet0/0/0
 ip address 172.31.0.2 255.255.0.0
 no shutdown
interface GigabitEthernet0/0/1
 ip address 209.165.200.1 255.255.255.0
 no shutdown

router ospf 1
 network 172.31.0.0 0.0.255.255 area 0
 network 209.165.200.0 0.0.0.255 area 0
```

SAM-R2:

```cisco
interface GigabitEthernet0/0/0
 ip address 209.165.200.2 255.255.255.0
 no shutdown

router ospf 1
 network 209.165.200.0 0.0.0.255 area 0
 network 172.19.0.0 0.0.255.255 area 0
 passive-interface GigabitEthernet0/0/1
```

SAM-R1 to SAM-R3 expansion:

```cisco
interface GigabitEthernet0/0/2
 ip address 192.168.13.1 255.255.255.0
 no shutdown

router ospf 1
 network 192.168.13.0 0.0.0.255 area 0
```

SAM-R3:

```cisco
interface GigabitEthernet0/0/0
 ip address 192.168.13.2 255.255.255.0
 no shutdown
interface GigabitEthernet0/0/1
 ip address 10.10.10.1 255.255.255.0
 no shutdown

router ospf 1
 network 192.168.13.0 0.0.0.255 area 0
 network 10.10.10.0 0.0.0.255 area 0
 passive-interface GigabitEthernet0/0/1
```

## SSH version 2

The sequence was repeated on the routers and switches.

```cisco
ip domain-name SSH
crypto key generate rsa
! The source uses 1024 bits on the initial devices.
username Sam secret Samabcd

line vty 0 4
 transport input ssh
 login local
 exit

ip ssh version 2
```

Observed connection tests:

```text
ssh -l Sam 209.165.200.1
ssh -l Sam 172.19.0.1
```

## SSH source restriction

```cisco
ip access-list extended 101
 deny tcp 192.168.20.0 0.0.0.255 any eq 22
 permit tcp 192.168.10.0 0.0.0.255 any eq 22
 permit ip any any

interface GigabitEthernet0/0/0.10
 ip access-group 101 in
interface GigabitEthernet0/0/0.20
 ip access-group 101 in
```

## Inter-VLAN isolation

```cisco
ip access-list standard 20
 deny 192.168.10.0 0.0.0.255
 permit any
interface GigabitEthernet0/0/0.20
 ip access-group 20 out

ip access-list standard 10
 deny 192.168.20.0 0.0.0.255
 permit any
interface GigabitEthernet0/0/0.10
 ip access-group 10 out
```

## PAT on SAM-R2

```cisco
interface GigabitEthernet0/0/0
 ip nat inside
interface GigabitEthernet0/0/1
 ip nat outside

access-list 1 permit 172.31.0.0 0.0.255.255
ip nat inside source list 1 interface GigabitEthernet0/0/1 overload
```

Validation:

```cisco
show ip nat translations
```

## SAM-R4 addressing and OSPF

```cisco
interface GigabitEthernet0/0/0
 ip address 10.10.10.2 255.255.255.0
 no shutdown
interface GigabitEthernet0/0/1
 ip address 192.168.13.3 255.255.255.0
 no shutdown

router ospf 1
 network 10.10.10.0 0.0.0.255 area 0
 network 192.168.13.0 0.0.0.255 area 0
 passive-interface GigabitEthernet0/0/0
```

## HSRP

SAM-R3, preferred active router:

```cisco
interface GigabitEthernet0/0/0
 standby 1 ip 192.168.13.222
 standby 1 priority 150
 standby 1 preempt
interface GigabitEthernet0/0/1
 standby 2 ip 10.10.10.222
 standby 2 priority 150
 standby 2 preempt
```

SAM-R4, standby router:

```cisco
interface GigabitEthernet0/0/1
 standby 1 ip 192.168.13.222
 standby 1 priority 100
 standby 1 preempt
interface GigabitEthernet0/0/0
 standby 2 ip 10.10.10.222
 standby 2 priority 100
 standby 2 preempt
```

Validation:

```cisco
show standby brief
```

## LACP EtherChannel and STP

SAM-S6 trunks the two links shown in the LAN3 switching path:

```cisco
interface range GigabitEthernet0/1, FastEthernet0/23
 switchport mode trunk
```

SAM-S5 and SAM-S7 use the same member ports:

```cisco
interface range FastEthernet0/21 - 22, FastEthernet0/24
 channel-group 1 mode active
 switchport mode trunk
```

SAM-S7 is selected as the VLAN 1 root bridge:

```cisco
spanning-tree vlan 1 root primary
```


## Syslog

```cisco
service timestamps log datetime msec
logging host 172.19.0.200
```

The server receives messages, but its displayed 1993 date shows that time synchronization is not configured.

## SAM-S4 management address

```cisco
interface vlan 1
 ip address 172.19.0.254 255.255.0.0
 no shutdown

ip default-gateway 172.19.0.1
```

ACL 101 permits SSH to this address from VLAN 10 and denies it from VLAN 20.
