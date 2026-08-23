# Jeremy's IT Lab — CCNA Mega Lab

A comprehensive Cisco Packet Tracer laboratory covering the major configuration and troubleshooting topics required for the **Cisco CCNA**.

This README documents the complete lab requirements, topology, addressing plan, configuration objectives, reusable Cisco command reference, verification commands, troubleshooting workflow, and final completion checklist.

> **Important:** This lab is extensive. Set aside several hours to complete it.
>
> **You must save the device configurations** using `copy running-config startup-config` or you may receive no points from the lab checker.

---

## 📌 Lab Overview

This Mega Lab covers:

- Initial Cisco device configuration
- VLANs
- VTPv2
- 802.1Q trunking
- DTP
- Layer-2 EtherChannel
- Layer-3 EtherChannel
- Inter-VLAN routing
- HSRPv2
- Rapid PVST+
- OSPF
- Static routing
- Floating static routes
- DHCP
- DHCP relay
- DNS
- NTP
- SNMP
- Syslog
- FTP
- SSH
- NAT
- PAT
- ACLs
- Port Security
- DHCP Snooping
- Dynamic ARP Inspection
- IPv6
- Wireless LAN Controller
- Lightweight Access Points
- WLAN security
- LLDP

---

# 🏗️ Topology

The lab contains an Internet/WAN connection, a routed core, two redundant office distribution layers, access switches, wired clients, phones, wireless infrastructure, and a server.

## Core / WAN

- R1
- CSW1
- CSW2
- Internet

## Office A

- DSW-A1
- DSW-A2
- ASW-A1
- ASW-A2
- ASW-A3
- WLC1
- LWAP1
- Laptop1
- PC1
- PC2
- Phone1
- Phone2

## Office B

- DSW-B1
- DSW-B2
- ASW-B1
- ASW-B2
- ASW-B3
- LWAP2
- Laptop2
- PC3
- Phone3
- SRV1

---

# 🗺️ VLAN Plan

| VLAN | Name | Office | Purpose |
|---:|---|---|---|
| 10 | PCs | A / B | Computer clients |
| 20 | Phones | A / B | IP phones |
| 30 | Servers | B | Server network |
| 40 | Wi-Fi | A | Wireless clients |
| 99 | Management | A / B | Network management |
| 1000 | Native-Unused | A / B | Unused native VLAN |

### Office A Trunks

Allowed VLANs:

```text
10,20,40,99
```

### Office B Trunks

Allowed VLANs:

```text
10,20,30,99
```

### Native VLAN

All trunks use:

```text
VLAN 1000
```

VLAN 1000 is intentionally unused.

---

# 🌐 IPv4 Addressing Plan

## R1

| Interface | Address |
|---|---|
| G0/0/0 | DHCP client |
| G0/1/0 | DHCP client |
| G0/0 | 10.0.0.33/30 |
| G0/1 | 10.0.0.37/30 |
| Loopback0 | 10.0.0.76/32 |

## CSW1

| Interface | Address |
|---|---|
| G1/0/1 | 10.0.0.34/30 |
| G1/1/1 | 10.0.0.45/30 |
| G1/1/2 | 10.0.0.49/30 |
| G1/1/3 | 10.0.0.53/30 |
| G1/1/4 | 10.0.0.57/30 |
| PortChannel1 | 10.0.0.41/30 |
| Loopback0 | 10.0.0.77/32 |

## CSW2

| Interface | Address |
|---|---|
| G1/0/1 | 10.0.0.38/30 |
| G1/1/1 | 10.0.0.61/30 |
| G1/1/2 | 10.0.0.65/30 |
| G1/1/3 | 10.0.0.69/30 |
| G1/1/4 | 10.0.0.73/30 |
| PortChannel1 | 10.0.0.42/30 |
| Loopback0 | 10.0.0.78/32 |

## DSW-A1

| Interface | Address |
|---|---|
| G1/1/1 | 10.0.0.46/30 |
| G1/1/2 | 10.0.0.62/30 |
| Loopback0 | 10.0.0.79/32 |

## DSW-A2

| Interface | Address |
|---|---|
| G1/1/1 | 10.0.0.50/30 |
| G1/1/2 | 10.0.0.66/30 |
| Loopback0 | 10.0.0.80/32 |

## DSW-B1

| Interface | Address |
|---|---|
| G1/1/1 | 10.0.0.54/30 |
| G1/1/2 | 10.0.0.70/30 |
| Loopback0 | 10.0.0.81/32 |

## DSW-B2

| Interface | Address |
|---|---|
| G1/1/1 | 10.0.0.58/30 |
| G1/1/2 | 10.0.0.74/30 |
| Loopback0 | 10.0.0.82/32 |

## Access Switch Management

| Device | VLAN 99 IP | Subnet |
|---|---|---|
| ASW-A1 | 10.0.0.4 | /28 |
| ASW-A2 | 10.0.0.5 | /28 |
| ASW-A3 | 10.0.0.6 | /28 |
| ASW-B1 | 10.0.0.20 | /28 |
| ASW-B2 | 10.0.0.21 | /28 |
| ASW-B3 | 10.0.0.22 | /28 |

The default gateway is the first usable address in the appropriate management subnet.

## SRV1

| Setting | Value |
|---|---|
| IPv4 Address | 10.5.0.4 |
| Subnet Mask | 255.255.255.0 |
| Default Gateway | 10.5.0.1 |

---

# ❤️ HSRPv2 Addressing

HSRP provides default-gateway redundancy between the two distribution switches in each office.

## Office A

| VLAN | Subnet | HSRP VIP | DSW-A1 | DSW-A2 | Active |
|---:|---|---|---|---|---|
| 99 | 10.0.0.0/28 | 10.0.0.1 | 10.0.0.2 | 10.0.0.3 | DSW-A1 |
| 10 | 10.1.0.0/24 | 10.1.0.1 | 10.1.0.2 | 10.1.0.3 | DSW-A1 |
| 20 | 10.2.0.0/24 | 10.2.0.1 | 10.2.0.2 | 10.2.0.3 | DSW-A2 |
| 40 | 10.6.0.0/24 | 10.6.0.1 | 10.6.0.2 | 10.6.0.3 | DSW-A2 |

## Office B

| VLAN | Subnet | HSRP VIP | DSW-B1 | DSW-B2 | Active |
|---:|---|---|---|---|---|
| 99 | 10.0.0.16/28 | 10.0.0.17 | 10.0.0.18 | 10.0.0.19 | DSW-B1 |
| 10 | 10.3.0.0/24 | 10.3.0.1 | 10.3.0.2 | 10.3.0.3 | DSW-B1 |
| 20 | 10.4.0.0/24 | 10.4.0.1 | 10.4.0.2 | 10.4.0.3 | DSW-B2 |
| 30 | 10.5.0.0/24 | 10.5.0.1 | 10.5.0.2 | 10.5.0.3 | DSW-B2 |

The intended HSRP Active router has priority **105** and preemption enabled.

---

# Part 1 — Initial Setup

## Requirements

1. Configure the appropriate hostname on every router and switch.
2. Configure the enable secret:
   - `jeremysitlab`
   - Use type 9 hashing if available.
   - Otherwise use type 5.
3. Configure the local user:
   - Username: `cisco`
   - Secret: `ccna`
   - Use type 9 hashing if available.
   - Otherwise use type 5.
4. Configure console access.
5. Configure synchronous logging where required.
6. Save the configuration on every device.

## Basic Configuration Template

```cisco
enable
configure terminal

hostname <HOSTNAME>

enable secret jeremysitlab

username cisco secret ccna

line console 0
 login local
 logging synchronous
 exit
```

Save:

```cisco
copy running-config startup-config
```

---

# Part 2 — VLANs and Layer-2 EtherChannel

## 2.1 Office A EtherChannel

Configure a Layer-2 EtherChannel between:

```text
DSW-A1 <----> DSW-A2
```

Requirements:

- Port-channel name/number: PortChannel1
- Protocol: Cisco proprietary PAgP
- Both switches actively negotiate

Use:

```cisco
interface range <PORTS>
 channel-group 1 mode desirable
```

---

## 2.2 Office B EtherChannel

Configure a Layer-2 EtherChannel between:

```text
DSW-B1 <----> DSW-B2
```

Requirements:

- Port-channel: PortChannel1
- Protocol: open standard LACP
- Both switches actively negotiate

Use:

```cisco
interface range <PORTS>
 channel-group 1 mode active
```

---

## 2.3 Trunks

Configure all links between Access and Distribution switches as trunks, including EtherChannels.

Requirements:

- Explicitly disable DTP.
- Native VLAN: 1000.
- Office A allowed VLANs: 10,20,40,99.
- Office B allowed VLANs: 10,20,30,99.

Example:

```cisco
interface <INTERFACE>
 switchport mode trunk
 switchport trunk native vlan 1000
 switchport trunk allowed vlan 10,20,40,99
 switchport nonegotiate
```

Office B:

```cisco
switchport trunk allowed vlan 10,20,30,99
```

---

## 2.4 VTPv2

Configure one Distribution switch in each office as the VTP server.

Domain:

```text
JeremysITLab
```

Server:

```cisco
vtp version 2
vtp domain JeremysITLab
vtp mode server
```

Configure all Access switches as VTP clients:

```cisco
vtp version 2
vtp domain JeremysITLab
vtp mode client
```

Verify:

```cisco
show vtp status
```

---

## 2.5 Office A VLANs

Create on one VTP server:

```cisco
vlan 10
 name PCs

vlan 20
 name Phones

vlan 40
 name Wi-Fi

vlan 99
 name Management
```

Verify propagation:

```cisco
show vlan brief
show vtp status
```

---

## 2.6 Office B VLANs

```cisco
vlan 10
 name PCs

vlan 20
 name Phones

vlan 30
 name Servers

vlan 99
 name Management
```

---

## 2.7 Access Ports

Configure Access switch ports manually.

Requirements:

- LWAPs do not use FlexConnect.
- PCs use VLAN 10.
- Phones use VLAN 20.
- SRV1 uses VLAN 30.
- Access mode must be explicitly configured.
- DTP must be disabled.

Example:

```cisco
interface <PORT>
 switchport mode access
 switchport access vlan 10
 switchport nonegotiate
```

---

## 2.8 ASW-A1 to WLC1

The WLC connection must support:

- Wi-Fi VLAN 40
- Management VLAN 99

Management VLAN must be untagged.

DTP must be disabled.

---

## 2.9 Unused Ports

Administratively shut down all unused ports on Access and Distribution switches.

```cisco
interface range <UNUSED-PORTS>
 shutdown
```

---

# Part 3 — IP Addresses, Layer-3 EtherChannel and HSRP

## 3.1 R1

Configure:

```text
G0/0/0 = DHCP client
G0/1/0 = DHCP client
G0/0   = 10.0.0.33/30
G0/1   = 10.0.0.37/30
Lo0    = 10.0.0.76/32
```

Enable all required interfaces.

---

## 3.2 Layer-3 EtherChannel

Enable IPv4 routing on all Core and Distribution switches.

Create a Layer-3 EtherChannel between CSW1 and CSW2.

Requirements:

- Cisco proprietary protocol
- Both switches actively negotiate
- PortChannel1

Addresses:

```text
CSW1 Po1 = 10.0.0.41/30
CSW2 Po1 = 10.0.0.42/30
```

Example:

```cisco
interface range <PORTS>
 no switchport
 channel-group 1 mode desirable
```

Then:

```cisco
interface port-channel 1
 no switchport
 ip address 10.0.0.41 255.255.255.252
```

Adjust the IP for CSW2.

---

## 3.3 CSW1 Interfaces

```text
G1/0/1 = 10.0.0.34/30
G1/1/1 = 10.0.0.45/30
G1/1/2 = 10.0.0.49/30
G1/1/3 = 10.0.0.53/30
G1/1/4 = 10.0.0.57/30
Lo0    = 10.0.0.77/32
```

Disable unused interfaces.

---

## 3.4 CSW2 Interfaces

```text
G1/0/1 = 10.0.0.38/30
G1/1/1 = 10.0.0.61/30
G1/1/2 = 10.0.0.65/30
G1/1/3 = 10.0.0.69/30
G1/1/4 = 10.0.0.73/30
Lo0    = 10.0.0.78/32
```

Disable unused interfaces.

---

## 3.5 Distribution Interfaces

### DSW-A1

```text
G1/1/1 = 10.0.0.46/30
G1/1/2 = 10.0.0.62/30
Lo0    = 10.0.0.79/32
```

### DSW-A2

```text
G1/1/1 = 10.0.0.50/30
G1/1/2 = 10.0.0.66/30
Lo0    = 10.0.0.80/32
```

### DSW-B1

```text
G1/1/1 = 10.0.0.54/30
G1/1/2 = 10.0.0.70/30
Lo0    = 10.0.0.81/32
```

### DSW-B2

```text
G1/1/1 = 10.0.0.58/30
G1/1/2 = 10.0.0.74/30
Lo0    = 10.0.0.82/32
```

---

## 3.6 SRV1

```text
IP address:   10.5.0.4
Mask:         255.255.255.0
Gateway:      10.5.0.1
```

---

## 3.7 Access Switch Management

Configure VLAN 99:

```text
ASW-A1 = 10.0.0.4/28
ASW-A2 = 10.0.0.5/28
ASW-A3 = 10.0.0.6/28

ASW-B1 = 10.0.0.20/28
ASW-B2 = 10.0.0.21/28
ASW-B3 = 10.0.0.22/28
```

Example:

```cisco
interface vlan 99
 ip address 10.0.0.4 255.255.255.240
 no shutdown

ip default-gateway 10.0.0.1
```

Use the correct address and gateway for each switch.

---

# 3.8 HSRPv2 — Office A

Enable HSRPv2:

```cisco
interface vlan <VLAN>
 standby version 2
```

## VLAN 99

```text
Subnet: 10.0.0.0/28
VIP:    10.0.0.1
A1:     10.0.0.2
A2:     10.0.0.3
Active: DSW-A1
```

Group 1.

DSW-A1 priority:

```text
105
```

Preemption enabled.

---

## VLAN 10

```text
Subnet: 10.1.0.0/24
VIP:    10.1.0.1
A1:     10.1.0.2
A2:     10.1.0.3
Active: DSW-A1
```

Group 2.

---

## VLAN 20

```text
Subnet: 10.2.0.0/24
VIP:    10.2.0.1
A1:     10.2.0.2
A2:     10.2.0.3
Active: DSW-A2
```

Group 3.

DSW-A2 priority:

```text
105
```

Preemption enabled.

---

## VLAN 40

```text
Subnet: 10.6.0.0/24
VIP:    10.6.0.1
A1:     10.6.0.2
A2:     10.6.0.3
Active: DSW-A2
```

Group 4.

---

# 3.9 HSRPv2 — Office B

## VLAN 99

```text
Subnet: 10.0.0.16/28
VIP:    10.0.0.17
B1:     10.0.0.18
B2:     10.0.0.19
Active: DSW-B1
```

Group 1.

DSW-B1 priority 105 and preemption enabled.

---

## VLAN 10

```text
Subnet: 10.3.0.0/24
VIP:    10.3.0.1
B1:     10.3.0.2
B2:     10.3.0.3
Active: DSW-B1
```

Group 2.

---

## VLAN 20

```text
Subnet: 10.4.0.0/24
VIP:    10.4.0.1
B1:     10.4.0.2
B2:     10.4.0.3
Active: DSW-B2
```

Group 3.

---

## VLAN 30

```text
Subnet: 10.5.0.0/24
VIP:    10.5.0.1
B1:     10.5.0.2
B2:     10.5.0.3
Active: DSW-B2
```

Group 4.

---

# Part 4 — Rapid Spanning Tree Protocol

## 4.1 Rapid PVST+

Configure all Access and Distribution switches:

```cisco
spanning-tree mode rapid-pvst
```

The STP root bridge for each VLAN must align with the HSRP Active router.

Configure:

- HSRP Active router = lowest possible STP priority.
- HSRP Standby router = one priority increment above the lowest priority.

Verify:

```cisco
show spanning-tree
show spanning-tree root
```

---

## 4.2 PortFast and BPDU Guard

Enable on all ports connected to end hosts, including WLC1.

```cisco
interface <PORT>
 spanning-tree portfast
 spanning-tree bpduguard enable
```

---

# Part 5 — Static and Dynamic Routing

# 5.1 OSPF

Configure OSPF on:

- R1 LAN-facing interfaces
- CSW1
- CSW2
- DSW-A1
- DSW-A2
- DSW-B1
- DSW-B2

Requirements:

- Process ID: 1
- Area: 0
- Router ID must match Loopback0
- Loopbacks must participate in OSPF
- Loopbacks must be passive
- Distribution switch SVIs except Management VLAN must be passive
- Physical OSPF neighbor links must use a network type that does not elect DR/BDR
- Layer-3 PortChannel between CSW1 and CSW2 remains at its default network type

Example:

```cisco
router ospf 1
 router-id <LOOPBACK-IP>
```

Switch network statements should match exact interface IP addresses:

```cisco
network <INTERFACE-IP> 0.0.0.0 area 0
```

R1 should enable OSPF directly in interface configuration mode:

```cisco
interface <INTERFACE>
 ip ospf 1 area 0
```

Passive interfaces:

```cisco
router ospf 1
 passive-interface <INTERFACE>
```

Point-to-point physical links:

```cisco
interface <INTERFACE>
 ip ospf network point-to-point
```

---

# 5.2 Static Default Routes

Configure one recursive default route for each Internet connection on R1.

The route through G0/1/0 must be a floating static route with:

```text
Administrative Distance = 2
```

R1 must advertise its default route through OSPF:

```cisco
router ospf 1
 default-information originate
```

> Packet Tracer does not provide the `default-information originate always` behavior used on real Cisco IOS for this failover scenario.

---

# Part 6 — Network Services

# 6.1 DHCP

R1 acts as the DHCP server.

Exclude the first ten usable addresses of every DHCP pool.

## A-Mgmt

```text
Network:        10.0.0.0/28
Gateway:        10.0.0.1
Domain:         jeremysitlab.com
DNS:            10.5.0.4
WLC:            10.0.0.7
```

## A-PC

```text
Network:        10.1.0.0/24
Gateway:        10.1.0.1
Domain:         jeremysitlab.com
DNS:            10.5.0.4
```

## A-Phone

```text
Network:        10.2.0.0/24
Gateway:        10.2.0.1
Domain:         jeremysitlab.com
DNS:            10.5.0.4
```

## B-Mgmt

```text
Network:        10.0.0.16/28
Gateway:        10.0.0.17
Domain:         jeremysitlab.com
DNS:            10.5.0.4
WLC:            10.0.0.7
```

## B-PC

```text
Network:        10.3.0.0/24
Gateway:        10.3.0.1
Domain:         jeremysitlab.com
DNS:            10.5.0.4
```

## B-Phone

```text
Network:        10.4.0.0/24
Gateway:        10.4.0.1
Domain:         jeremysitlab.com
DNS:            10.5.0.4
```

## Wi-Fi

```text
Network:        10.6.0.0/24
Gateway:        10.6.0.1
Domain:         jeremysitlab.com
DNS:            10.5.0.4
```

Example:

```cisco
ip dhcp excluded-address <FIRST-USABLE> <TENTH-USABLE>

ip dhcp pool A-PC
 network 10.1.0.0 255.255.255.0
 default-router 10.1.0.1
 domain-name jeremysitlab.com
 dns-server 10.5.0.4
```

---

# 6.2 DHCP Relay

Distribution switches must relay wired DHCP broadcasts to:

```text
R1 Loopback0 = 10.0.0.76
```

Example:

```cisco
interface vlan <VLAN>
 ip helper-address 10.0.0.76
```

---

# 6.3 DNS — SRV1

Configure these DNS entries:

| Name | Address / Target |
|---|---|
| google.com | 172.253.62.100 |
| youtube.com | 152.250.31.93 |
| jeremysitlab.com | 66.235.200.145 |
| www.jeremysitlab.com | jeremysitlab.com |

---

# 6.4 DNS on Network Devices

All routers and switches must use:

```text
Domain: jeremysitlab.com
DNS Server: 10.5.0.4
```

Example:

```cisco
ip domain-name jeremysitlab.com
ip name-server 10.5.0.4
```

---

# 6.5 NTP

## R1

R1 must:

- Act as a Stratum 5 NTP server.
- Learn time from `216.239.35.0`.

Example:

```cisco
ntp master 5
ntp server 216.239.35.0
```

> NTP can take a long time to synchronize in Packet Tracer. Continue with the lab instead of waiting for synchronization.

## Network Clients

All Core, Distribution, and Access switches use:

```text
10.0.0.76
```

as the NTP server.

Authentication:

```text
Key: 1
Password: ccna
```

Example:

```cisco
ntp authentication-key 1 md5 ccna
ntp trusted-key 1
ntp server 10.0.0.76 key 1
```

---

# 6.6 SNMP

Configure all routers and switches with:

```cisco
snmp-server community SNMPSTRING RO
```

Requirements:

- Community string: `SNMPSTRING`
- GET allowed
- SET not allowed

---

# 6.7 Syslog

Send all severity levels to SRV1:

```text
10.5.0.4
```

Enable buffered logging with 8192 bytes.

Example:

```cisco
logging host 10.5.0.4
logging trap debugging
logging buffered 8192
```

Verify:

```cisco
show logging
```

---

# 6.8 FTP / IOS Upgrade

Configure R1 FTP credentials:

```text
Username: cisco
Password: cisco
```

Example:

```cisco
ip ftp username cisco
ip ftp password cisco
```

Download:

```text
c2900-universalk9-mz.SPA.155-3.M4a.bin
```

from SRV1 to R1 flash.

Then:

1. Configure R1 to boot using the new IOS.
2. Reboot R1.
3. Verify the new IOS.
4. Delete the old IOS image from flash.

---

# 6.9 SSH

Configure secure remote access on all routers and switches.

Requirements:

- Largest RSA modulus supported
- SSHv2 only
- Standard ACL 1
- ACL 1 permits only Office A PCs
- ACL applied to all VTY lines
- Only SSH connections allowed
- Local user authentication required
- Synchronous logging enabled

Domain:

```text
jeremysitlab.com
```

Example:

```cisco
ip domain-name jeremysitlab.com
crypto key generate rsa
ip ssh version 2

access-list 1 permit 10.1.0.0 0.0.0.255

line vty 0 15
 login local
 transport input ssh
 access-class 1 in
 logging synchronous
```

---

# 6.10 Static NAT

Allow Internet hosts to access SRV1 using:

```text
203.0.113.113
```

Internal server:

```text
10.5.0.4
```

Example:

```cisco
ip nat inside source static 10.5.0.4 203.0.113.113
```

Configure appropriate inside/outside interfaces.

---

# 6.11 Dynamic PAT

Office networks requiring Internet access:

```text
10.1.0.0/24
10.2.0.0/24
10.3.0.0/24
10.4.0.0/24
10.6.0.0/24
```

Create standard ACL 2 in the required order:

```cisco
access-list 2 permit 10.1.0.0 0.0.0.255
access-list 2 permit 10.2.0.0 0.0.0.255
access-list 2 permit 10.3.0.0 0.0.0.255
access-list 2 permit 10.4.0.0 0.0.0.255
access-list 2 permit 10.6.0.0 0.0.0.255
```

Create:

```text
POOL1
203.0.113.200 - 203.0.113.207
/29
```

Example:

```cisco
ip nat pool POOL1 203.0.113.200 203.0.113.207 netmask 255.255.255.248
ip nat inside source list 2 pool POOL1 overload
```

Test Internet access by pinging:

```text
jeremysitlab.com
```

## Internet Failover Test

1. Disable R1 G0/0/0.
2. Remove and reconfigure `default-information originate`.
3. Verify Internet access again.
4. Re-enable G0/0/0.
5. Remove and reconfigure `default-information originate` again.

---

# 6.12 CDP and LLDP

Disable CDP on all devices:

```cisco
no cdp run
```

Enable LLDP:

```cisco
lldp run
```

On every Access switch F0/1:

```cisco
interface f0/1
 no lldp transmit
```

Verify:

```cisco
show lldp neighbors
```

---

# Part 7 — Security

# 7.1 Extended ACL — OfficeA_to_OfficeB

Create extended ACL:

```text
OfficeA_to_OfficeB
```

Requirements:

1. Allow ICMP from Office A PCs to Office B PCs.
2. Block all other traffic from Office A PCs to Office B PCs.
3. Allow all other traffic.
4. Apply the ACL according to best practice for extended ACL placement.

Example:

```cisco
ip access-list extended OfficeA_to_OfficeB
 permit icmp 10.1.0.0 0.0.0.255 10.3.0.0 0.0.0.255
 deny ip 10.1.0.0 0.0.0.255 10.3.0.0 0.0.0.255
 permit ip any any
```

Extended ACLs should generally be placed close to the source.

---

# 7.2 Port Security

Configure on each Access switch F0/1.

Requirements:

- Minimum necessary MAC addresses
- SRV1 uses one MAC address
- Violation mode must block invalid traffic without affecting valid traffic
- Notifications must be generated
- Secure MAC addresses must be dynamically saved to running-config

Example:

```cisco
interface f0/1
 switchport mode access
 switchport port-security
 switchport port-security maximum 1
 switchport port-security violation restrict
 switchport port-security mac-address sticky
```

Verify:

```cisco
show port-security
show port-security interface f0/1
```

---

# 7.3 DHCP Snooping

Configure on all Access switches.

Requirements:

- Enable for all active VLANs
- Trust appropriate uplinks
- Disable DHCP Option 82 insertion
- Untrusted active ports: 15 packets/sec
- ASW-A1 WLC connection: 100 packets/sec

Example:

```cisco
ip dhcp snooping
ip dhcp snooping vlan 10,20,30,40,99
no ip dhcp snooping information option
```

Trusted uplink:

```cisco
interface <UPLINK>
 ip dhcp snooping trust
```

Untrusted port:

```cisco
interface <PORT>
 ip dhcp snooping limit rate 15
```

WLC connection:

```cisco
interface <WLC-PORT>
 ip dhcp snooping limit rate 100
```

Verify:

```cisco
show ip dhcp snooping
show ip dhcp snooping binding
```

---

# 7.4 Dynamic ARP Inspection

Configure DAI on all Access switches.

Requirements:

- Enable for all active VLANs
- Trust appropriate uplinks
- Enable all optional validation checks

Example:

```cisco
ip arp inspection vlan 10,20,30,40,99
```

Trusted uplink:

```cisco
interface <UPLINK>
 ip arp inspection trust
```

Validation:

```cisco
ip arp inspection validate src-mac dst-mac ip
```

Verify:

```cisco
show ip arp inspection
show ip arp inspection interfaces
```

---

# Part 8 — IPv6

## 8.1 IPv6 Routing

Enable IPv6 routing on:

- R1
- CSW1
- CSW2

```cisco
ipv6 unicast-routing
```

---

## 8.2 IPv6 Addresses

### R1 G0/0/0

```text
2001:db8:a::2/64
```

### R1 G0/1/0

```text
2001:db8:b::2/64
```

### R1 G0/0 and CSW1 G1/0/1

Use:

```text
2001:db8:a1::/64
```

and EUI-64.

Example:

```cisco
ipv6 address 2001:db8:a1::/64 eui-64
```

### R1 G0/1 and CSW2 G1/0/1

Use:

```text
2001:db8:a2::/64
```

and EUI-64.

### CSW1 Po1 and CSW2 Po1

Enable IPv6 without using the `ipv6 address` command:

```cisco
ipv6 enable
```

---

# 8.3 IPv6 Default Routes

R1 requires two default routes.

## Primary

Recursive:

```text
Next hop: 2001:db8:a::1
```

Example:

```cisco
ipv6 route ::/0 2001:db8:a::1
```

## Floating

Fully specified route:

```text
Next hop: 2001:db8:b::1
```

Administrative distance:

```text
2
```

Example:

```cisco
ipv6 route ::/0 <EXIT-INTERFACE> 2001:db8:b::1 2
```

---

# Part 9 — Wireless

# 9.1 WLC Management

Access the WLC GUI from a PC:

```text
https://10.0.0.7
```

Credentials:

```text
Username: admin
Password: adminPW12
```

---

# 9.2 WLC Dynamic Interface

Create a dynamic interface for the Wi-Fi WLAN.

| Setting | Value |
|---|---|
| Name | Wi-Fi |
| VLAN | 40 |
| Port | 1 |
| IP | 10.6.0.4 |
| Gateway | 10.6.0.1 |
| DHCP Server | 10.0.0.76 |

> **Important:** The Mega Lab video reportedly shows `10.6.0.2`, but this is incorrect because it duplicates DSW-A1's VLAN 40 address. Use **10.6.0.4**.

---

# 9.3 WLAN

Configure and enable:

| Setting | Value |
|---|---|
| Profile Name | Wi-Fi |
| SSID | Wi-Fi |
| ID | 1 |
| Status | Enabled |
| Security | WPA2 Policy |
| Encryption | AES |
| PSK | cisco123 |

---

# 9.4 LWAPs

Verify that:

- LWAP1 associates with WLC1.
- LWAP2 associates with WLC1.

> **Packet Tracer limitation:** Wireless clients may not successfully lease an address from the Wi-Fi DHCP pool due to Packet Tracer's wireless implementation limitations.

---

# 🧰 Reusable Cisco Command Reference

This section consolidates commands that are repeated throughout the lab.

## Basic Device Configuration

```cisco
enable
configure terminal

hostname <HOSTNAME>

enable secret jeremysitlab

username cisco secret ccna

service password-encryption

line console 0
 login local
 logging synchronous
 exit
```

---

## Save Configuration

```cisco
copy running-config startup-config
```

or:

```cisco
write memory
```

---

## Interface Configuration

```cisco
interface <INTERFACE>
 description <DESCRIPTION>
 ip address <IP> <MASK>
 no shutdown
```

---

## Shutdown Unused Ports

```cisco
interface range <PORT-RANGE>
 shutdown
```

---

## VLANs

```cisco
vlan 10
 name PCs

vlan 20
 name Phones

vlan 30
 name Servers

vlan 40
 name Wi-Fi

vlan 99
 name Management

vlan 1000
 name Native-Unused
```

---

## Access Port

```cisco
interface <PORT>
 switchport mode access
 switchport access vlan <VLAN>
 switchport nonegotiate
```

---

## Trunk Port

```cisco
interface <PORT>
 switchport mode trunk
 switchport trunk native vlan 1000
 switchport trunk allowed vlan <VLAN-LIST>
 switchport nonegotiate
```

---

## PAgP EtherChannel

```cisco
interface range <PORTS>
 channel-group 1 mode desirable
```

```cisco
interface port-channel 1
```

---

## LACP EtherChannel

```cisco
interface range <PORTS>
 channel-group 1 mode active
```

---

## Layer-3 EtherChannel

```cisco
interface range <PORTS>
 no switchport
 channel-group 1 mode desirable
```

```cisco
interface port-channel 1
 no switchport
 ip address <IP> <MASK>
```

---

## EtherChannel Verification

```cisco
show etherchannel summary
```

---

## VTP

Server:

```cisco
vtp version 2
vtp domain JeremysITLab
vtp mode server
```

Client:

```cisco
vtp version 2
vtp domain JeremysITLab
vtp mode client
```

Verification:

```cisco
show vtp status
```

---

## HSRP

```cisco
interface vlan <VLAN>
 standby version 2
 standby <GROUP> ip <VIP>
```

Active router:

```cisco
standby <GROUP> priority 105
standby <GROUP> preempt
```

Verification:

```cisco
show standby brief
```

---

## Rapid PVST+

```cisco
spanning-tree mode rapid-pvst
```

STP priority:

```cisco
spanning-tree vlan <VLAN> priority <PRIORITY>
```

Host-facing port:

```cisco
interface <PORT>
 spanning-tree portfast
 spanning-tree bpduguard enable
```

Verification:

```cisco
show spanning-tree
show spanning-tree root
```

---

## OSPF

```cisco
router ospf 1
 router-id <LOOPBACK-IP>
```

Exact interface network:

```cisco
network <INTERFACE-IP> 0.0.0.0 area 0
```

Passive interface:

```cisco
passive-interface <INTERFACE>
```

Point-to-point:

```cisco
interface <INTERFACE>
 ip ospf network point-to-point
```

Verification:

```cisco
show ip ospf neighbor
show ip ospf interface brief
show ip route ospf
```

---

## Static Default Route

```cisco
ip route 0.0.0.0 0.0.0.0 <NEXT-HOP>
```

Floating route:

```cisco
ip route 0.0.0.0 0.0.0.0 <NEXT-HOP> 2
```

OSPF default advertisement:

```cisco
router ospf 1
 default-information originate
```

---

## DHCP

Exclude addresses:

```cisco
ip dhcp excluded-address <START-IP> <END-IP>
```

Pool:

```cisco
ip dhcp pool <POOL-NAME>
 network <NETWORK> <MASK>
 default-router <GATEWAY>
 domain-name jeremysitlab.com
 dns-server 10.5.0.4
```

Verification:

```cisco
show ip dhcp pool
show ip dhcp binding
```

---

## DHCP Relay

```cisco
interface vlan <VLAN>
 ip helper-address 10.0.0.76
```

---

## DNS

```cisco
ip domain-name jeremysitlab.com
ip name-server 10.5.0.4
```

---

## NTP

R1:

```cisco
ntp master 5
ntp server 216.239.35.0
```

Clients:

```cisco
ntp authentication-key 1 md5 ccna
ntp trusted-key 1
ntp server 10.0.0.76 key 1
```

Verification:

```cisco
show ntp status
show ntp associations
```

---

## SNMP

```cisco
snmp-server community SNMPSTRING RO
```

---

## Syslog

```cisco
logging host 10.5.0.4
logging trap debugging
logging buffered 8192
```

Verification:

```cisco
show logging
```

---

## FTP

```cisco
ip ftp username cisco
ip ftp password cisco
```

---

## SSH

```cisco
ip domain-name jeremysitlab.com
crypto key generate rsa
ip ssh version 2
```

ACL:

```cisco
access-list 1 permit 10.1.0.0 0.0.0.255
```

VTY:

```cisco
line vty 0 15
 login local
 transport input ssh
 access-class 1 in
 logging synchronous
```

---

## Static NAT

```cisco
ip nat inside source static 10.5.0.4 203.0.113.113
```

---

## PAT

ACL 2:

```cisco
access-list 2 permit 10.1.0.0 0.0.0.255
access-list 2 permit 10.2.0.0 0.0.0.255
access-list 2 permit 10.3.0.0 0.0.0.255
access-list 2 permit 10.4.0.0 0.0.0.255
access-list 2 permit 10.6.0.0 0.0.0.255
```

Pool:

```cisco
ip nat pool POOL1 203.0.113.200 203.0.113.207 netmask 255.255.255.248
```

PAT:

```cisco
ip nat inside source list 2 pool POOL1 overload
```

Inside interface:

```cisco
interface <LAN-INTERFACE>
 ip nat inside
```

Outside interface:

```cisco
interface <WAN-INTERFACE>
 ip nat outside
```

Verification:

```cisco
show ip nat translations
show ip nat statistics
```

---

## Port Security

```cisco
interface f0/1
 switchport mode access
 switchport port-security
 switchport port-security maximum 1
 switchport port-security violation restrict
 switchport port-security mac-address sticky
```

Verification:

```cisco
show port-security
show port-security interface f0/1
```

---

## DHCP Snooping

```cisco
ip dhcp snooping
ip dhcp snooping vlan 10,20,30,40,99
no ip dhcp snooping information option
```

Trusted uplink:

```cisco
interface <UPLINK>
 ip dhcp snooping trust
```

Untrusted port:

```cisco
interface <PORT>
 ip dhcp snooping limit rate 15
```

WLC port:

```cisco
interface <WLC-PORT>
 ip dhcp snooping limit rate 100
```

Verification:

```cisco
show ip dhcp snooping
show ip dhcp snooping binding
```

---

## Dynamic ARP Inspection

```cisco
ip arp inspection vlan 10,20,30,40,99
```

Trusted uplink:

```cisco
interface <UPLINK>
 ip arp inspection trust
```

Validation:

```cisco
ip arp inspection validate src-mac dst-mac ip
```

Verification:

```cisco
show ip arp inspection
show ip arp inspection interfaces
```

---

## Extended ACL

```cisco
ip access-list extended OfficeA_to_OfficeB
 permit icmp 10.1.0.0 0.0.0.255 10.3.0.0 0.0.0.255
 deny ip 10.1.0.0 0.0.0.255 10.3.0.0 0.0.0.255
 permit ip any any
```

Apply:

```cisco
interface <INTERFACE>
 ip access-group OfficeA_to_OfficeB out
```

---

## CDP / LLDP

Disable CDP:

```cisco
no cdp run
```

Enable LLDP:

```cisco
lldp run
```

Disable LLDP transmission on F0/1:

```cisco
interface f0/1
 no lldp transmit
```

Verification:

```cisco
show lldp neighbors
```

---

## IPv6

Enable routing:

```cisco
ipv6 unicast-routing
```

Normal address:

```cisco
ipv6 address <ADDRESS>/<PREFIX>
```

EUI-64:

```cisco
ipv6 address 2001:db8:a1::/64 eui-64
```

Enable IPv6 without an address:

```cisco
ipv6 enable
```

Recursive default route:

```cisco
ipv6 route ::/0 2001:db8:a::1
```

Floating fully specified route:

```cisco
ipv6 route ::/0 <INTERFACE> 2001:db8:b::1 2
```

---

# 🔍 Verification Command Reference

## Interfaces

```cisco
show ip interface brief
show interfaces status
show interfaces description
```

## VLANs

```cisco
show vlan brief
```

## Trunks

```cisco
show interfaces trunk
```

## EtherChannel

```cisco
show etherchannel summary
```

## VTP

```cisco
show vtp status
```

## HSRP

```cisco
show standby brief
```

## STP

```cisco
show spanning-tree
show spanning-tree root
```

## OSPF

```cisco
show ip ospf neighbor
show ip ospf interface brief
show ip route
show ip route ospf
```

## DHCP

```cisco
show ip dhcp pool
show ip dhcp binding
```

## NAT

```cisco
show ip nat translations
show ip nat statistics
```

## SSH

```cisco
show ip ssh
show access-lists
```

## Security

```cisco
show port-security
show port-security interface f0/1
show ip dhcp snooping
show ip dhcp snooping binding
show ip arp inspection
show ip arp inspection interfaces
```

## NTP

```cisco
show ntp status
show ntp associations
```

## Syslog

```cisco
show logging
```

## LLDP

```cisco
show lldp neighbors
show lldp neighbors detail
```

## IPv6

```cisco
show ipv6 interface brief
show ipv6 route
```

---

# 🧪 Troubleshooting Workflow

When a connection or protocol does not work, troubleshoot in layers.

## Step 1 — Physical Layer

Check:

```cisco
show ip interface brief
show interfaces status
```

Confirm interfaces are:

```text
up/up
```

---

## Step 2 — VLANs

```cisco
show vlan brief
```

Confirm the correct access VLAN.

---

## Step 3 — Trunks

```cisco
show interfaces trunk
```

Check:

- Trunk status
- Native VLAN
- Allowed VLANs
- DTP disabled

---

## Step 4 — EtherChannel

```cisco
show etherchannel summary
```

Look for a healthy PortChannel and member interfaces.

---

## Step 5 — STP

```cisco
show spanning-tree
show spanning-tree root
```

Confirm the expected root bridge.

---

## Step 6 — HSRP

```cisco
show standby brief
```

Confirm:

- Correct VIP
- Correct Active router
- Correct Standby router
- Priority
- Preemption

---

## Step 7 — IP Addressing

```cisco
show ip interface brief
```

Check IP addresses and masks.

---

## Step 8 — Routing

```cisco
show ip route
show ip ospf neighbor
```

Confirm routes exist and OSPF neighbors are established.

---

## Step 9 — DHCP

```cisco
show ip dhcp binding
show ip dhcp pool
```

Check the DHCP relay:

```cisco
show running-config interface vlan <VLAN>
```

---

## Step 10 — ACLs

```cisco
show access-lists
```

Check whether an ACL is unexpectedly blocking traffic.

---

## Step 11 — NAT

```cisco
show ip nat translations
show ip nat statistics
```

---

## Step 12 — Security Features

Check:

```cisco
show port-security
show ip dhcp snooping
show ip arp inspection
```

---

## Step 13 — Connectivity Tests

Use:

```cisco
ping <IP>
traceroute <IP>
```

Test progressively:

```text
Host → Default Gateway
Host → Local VLAN
Host → Remote VLAN
Host → R1
Host → Internet
```

---

# ⚠️ Packet Tracer Limitations and Lab Notes

This is a very large Packet Tracer topology.

Packet Tracer may experience:

- Slow performance
- Delayed protocol convergence
- DHCP delays
- NTP synchronization delays
- Wireless limitations
- IOS command limitations
- Unexpected behavior after extensive configuration
- Temporary simulation/realtime inconsistencies

If the lab behaves incorrectly:

1. Save all device configurations.
2. Save the `.pkt` file.
3. Close Packet Tracer.
4. Reopen the lab.
5. Verify the configuration again.

## Important Wireless Limitation

Wireless clients may not successfully obtain DHCP addresses because of Packet Tracer limitations.

## Important NTP Limitation

NTP can take a long time to synchronize. Do not wait indefinitely before continuing.

## Important OSPF Limitation

Packet Tracer does not provide the same behavior as real IOS for:

```cisco
default-information originate always
```

The lab therefore requires removing and reconfiguring:

```cisco
default-information originate
```

during the Internet failover test.

---

# 💾 Configuration Saving Requirement

Before using **Check Results**, save every device.

Use:

```cisco
copy running-config startup-config
```

Do this on:

- R1
- CSW1
- CSW2
- DSW-A1
- DSW-A2
- DSW-B1
- DSW-B2
- ASW-A1
- ASW-A2
- ASW-A3
- ASW-B1
- ASW-B2
- ASW-B3

Failure to save startup configurations may result in lost points.

---

# ✅ Final Completion Checklist

## Initial Configuration

- [ ] All hostnames configured
- [ ] Enable secret configured
- [ ] Local `cisco` user configured
- [ ] Console access configured
- [ ] Synchronous logging configured where required
- [ ] Configurations saved

## Switching

- [ ] VLANs created
- [ ] VLAN names correct
- [ ] VTPv2 configured
- [ ] VTP domain is `JeremysITLab`
- [ ] Access switches are VTP clients
- [ ] Trunks configured
- [ ] DTP disabled
- [ ] Native VLAN 1000 configured
- [ ] Correct allowed VLANs configured
- [ ] Access ports assigned correctly
- [ ] Unused ports shutdown
- [ ] Office A PAgP EtherChannel operational
- [ ] Office B LACP EtherChannel operational

## Layer 3

- [ ] IPv4 addresses configured
- [ ] Loopback interfaces configured
- [ ] IPv4 routing enabled
- [ ] Layer-3 EtherChannel operational
- [ ] Management SVIs configured
- [ ] HSRPv2 configured
- [ ] Correct HSRP Active routers
- [ ] Correct HSRP priorities
- [ ] HSRP preemption enabled

## STP

- [ ] Rapid PVST+ enabled
- [ ] STP root aligns with HSRP Active router
- [ ] Standby router has next STP priority
- [ ] PortFast configured
- [ ] BPDU Guard configured

## Routing

- [ ] OSPF process 1 configured
- [ ] OSPF Area 0 configured
- [ ] Router IDs match loopbacks
- [ ] OSPF loopbacks enabled
- [ ] Loopbacks passive
- [ ] Appropriate SVIs passive
- [ ] Physical OSPF links use point-to-point network type
- [ ] Layer-3 PortChannel keeps default OSPF network type
- [ ] Static default routes configured
- [ ] Floating default route configured
- [ ] OSPF default route advertisement configured

## DHCP / DNS

- [ ] DHCP pools configured
- [ ] First ten usable addresses excluded
- [ ] Correct gateways configured
- [ ] Correct DNS server configured
- [ ] Correct domain configured
- [ ] WLC DHCP information configured where required
- [ ] DHCP relay configured
- [ ] SRV1 DNS entries configured

## Network Services

- [ ] DNS configured
- [ ] NTP configured
- [ ] NTP authentication configured
- [ ] SNMP configured
- [ ] Syslog configured
- [ ] Buffered logging configured
- [ ] FTP credentials configured
- [ ] IOS image copied
- [ ] R1 booted from new IOS
- [ ] Old IOS removed

## SSH

- [ ] Domain name configured
- [ ] RSA keys generated
- [ ] Largest supported RSA modulus used
- [ ] SSHv2 enabled
- [ ] ACL 1 configured
- [ ] ACL 1 restricts VTY access
- [ ] VTY accepts SSH only
- [ ] Local authentication enabled
- [ ] Synchronous logging enabled

## NAT / PAT

- [ ] Static NAT configured
- [ ] Public SRV1 address is `203.0.113.113`
- [ ] ACL 2 configured
- [ ] POOL1 configured
- [ ] PAT enabled
- [ ] Internet connectivity verified
- [ ] Primary Internet link tested
- [ ] Floating Internet link tested
- [ ] Primary link restored
- [ ] OSPF default advertisement restored

## Security

- [ ] Extended ACL `OfficeA_to_OfficeB` configured
- [ ] ICMP Office A → Office B allowed
- [ ] Other Office A → Office B traffic blocked
- [ ] Other traffic allowed
- [ ] ACL applied in appropriate location
- [ ] Port Security configured
- [ ] Sticky MAC enabled
- [ ] Correct maximum MAC addresses configured
- [ ] Violation mode configured
- [ ] DHCP Snooping configured
- [ ] Trusted ports configured
- [ ] Option 82 disabled
- [ ] Rate limits configured
- [ ] DAI configured
- [ ] DAI trusted ports configured
- [ ] DAI validation checks enabled

## IPv6

- [ ] IPv6 routing enabled
- [ ] IPv6 addresses configured
- [ ] EUI-64 configured
- [ ] IPv6 enabled on Layer-3 PortChannel
- [ ] Recursive IPv6 default route configured
- [ ] Floating fully-specified IPv6 default route configured

## Wireless

- [ ] WLC reachable
- [ ] WLC dynamic interface configured
- [ ] Wi-Fi VLAN 40 configured
- [ ] WLC Wi-Fi address is `10.6.0.4`
- [ ] Gateway is `10.6.0.1`
- [ ] DHCP server is `10.0.0.76`
- [ ] WLAN profile created
- [ ] SSID is `Wi-Fi`
- [ ] WLAN ID is 1
- [ ] WLAN enabled
- [ ] WPA2 configured
- [ ] AES encryption configured
- [ ] PSK is `cisco123`
- [ ] LWAP1 associated
- [ ] LWAP2 associated

## Final

- [ ] All configurations saved
- [ ] Packet Tracer file saved
- [ ] Connectivity tested
- [ ] Major protocols verified
- [ ] Check Results completed

---

# 📚 Key Learning Objectives

By completing this Mega Lab, you should be able to demonstrate practical CCNA-level knowledge of:

## Switching

- VLANs
- Trunking
- VTP
- EtherChannel
- Rapid PVST+
- STP root bridge selection
- Layer-2 security

## Routing

- IPv4 addressing
- Static routing
- Floating static routes
- OSPF
- HSRP
- Inter-VLAN routing

## Network Services

- DHCP
- DHCP relay
- DNS
- NTP
- SNMP
- Syslog
- FTP
- SSH
- NAT
- PAT

## Security

- Standard ACLs
- Extended ACLs
- Port Security
- DHCP Snooping
- Dynamic ARP Inspection
- BPDU Guard
- SSH security

## IPv6

- IPv6 addressing
- EUI-64
- IPv6 routing
- IPv6 default routes
- Floating IPv6 routes

## Wireless

- WLC
- LWAP
- WLAN
- VLAN-backed wireless networks
- WPA2/AES
- Wireless DHCP

---

# 🧠 Lab Philosophy

Do not simply copy commands.

For every configuration, understand:

> **What problem does this solve?**

> **Why is this configured on this device?**

> **Why is this configured on this interface?**

> **What happens if the primary device fails?**

> **How would I verify that it is working?**

> **How would I troubleshoot it if it failed?**

The objective of the Mega Lab is not only to complete the Packet Tracer checker, but to develop the practical troubleshooting mindset expected from a network engineer.

---

# 🔧 Essential Commands to Memorize

If you are preparing for the CCNA or a Network/NOC role, become comfortable with these commands:

```cisco
show ip interface brief
show interfaces status
show vlan brief
show interfaces trunk
show etherchannel summary
show vtp status
show spanning-tree
show spanning-tree root
show standby brief
show ip route
show ip ospf neighbor
show ip ospf interface brief
show ip dhcp pool
show ip dhcp binding
show ip nat translations
show ip nat statistics
show access-lists
show ip ssh
show port-security
show ip dhcp snooping
show ip arp inspection
show ntp status
show ntp associations
show logging
show lldp neighbors
show ipv6 interface brief
show ipv6 route
ping <ADDRESS>
traceroute <ADDRESS>
copy running-config startup-config
```

---

# 🎯 Final Goal

The completed topology should provide:

```text
                         INTERNET
                            |
                           R1
                      /-----------\
                     /             \
                  CSW1===========CSW2
                    |               |
             -----------         -----------
             |         |         |         |
          DSW-A1====DSW-A2    DSW-B1====DSW-B2
             |         |         |         |
           Access    Access     Access    Access
           Switches  Switches   Switches  Switches
             |         |         |         |
           Clients   Clients   Clients   SRV1

                      WLC1
                       |
                    LWAPs
                       |
                    Wi-Fi
```

The design demonstrates:

```text
Redundancy
    ↓
HSRP + EtherChannel + RSTP

Routing
    ↓
OSPF + Static Routing

Services
    ↓
DHCP + DNS + NTP + SNMP + Syslog + FTP + SSH

Security
    ↓
ACL + Port Security + DHCP Snooping + DAI

Wireless
    ↓
WLC + LWAP + WLAN + WPA2

Internet Access
    ↓
Static NAT + PAT + Floating Default Route

IPv6 Readiness
    ↓
IPv6 Addressing + EUI-64 + IPv6 Static Routing
```

---

## 🏁 Completion

When the entire lab is configured:

1. Verify all major protocols.
2. Test end-to-end connectivity.
3. Test HSRP failover.
4. Test Internet route failover.
5. Test DHCP.
6. Test DNS.
7. Test SSH.
8. Test NAT/PAT.
9. Verify security features.
10. Save every device configuration.
11. Save the Packet Tracer file.
12. Click **Check Results**.

---
