# Cisco ISP / NOC Enterprise Network Lab

## Overview

This Cisco Packet Tracer project simulates a small ISP/NOC enterprise network. It provides practical experience with Cisco routers, Layer 2 and Layer 3 switching, VLANs, trunking, inter-VLAN routing, static routing, default routing, and structured NOC troubleshooting.

## Network Topology

```text
                         ISP NETWORK

                     +----------------+
                     |    ISP-R1      |
                     | G0/0           |
                     | 10.0.0.1/30    |
                     +-------+--------+
                             |
                             | 10.0.0.0/30
                             |
                     +-------+--------+
                     |    ISP-R2      |
                     | G0/0 10.0.0.2  |
                     | G0/1 10.0.1.1  |
                     +-------+--------+
                             |
                             | 10.0.1.0/30
                             |
                     +-------+--------+
                     |      SW1       |
                     | Cisco 3560 L3  |
                     | G0/2 10.0.1.2  |
                     +---+---------+--+
                         |         |
                    Trunk|         |Trunk
                         |         |
                 +-------+--+   +--+-------+
                 |   SW2    |   |   SW3    |
                 | L2 2960  |   | L2 2960  |
                 +--+----+--+   +--+----+--+
                    |    |          |    |
                   PC1  PC2        PC3  PC4
```

## Devices

| Device | Model | Role |
|---|---|---|
| ISP-R1 | Cisco 2911 | Upstream ISP router |
| ISP-R2 | Cisco 2911 | ISP/customer edge router |
| SW1 | Cisco 3560-24PS | Layer 3 distribution switch |
| SW2 | Cisco 2960 | Layer 2 access switch |
| SW3 | Cisco 2960 | Layer 2 access switch |
| PC1 | PC | Customer A |
| PC2 | PC | Customer B |
| PC3 | PC | Customer C |
| PC4 | PC | Customer D |

> SW1 must be a Layer 3 switch such as the Cisco 3560. The Cisco 2960 used for SW2/SW3 is Layer 2 and does not support the required `ip routing` configuration in this lab.

## IP Addressing Plan

### ISP Links

| Device | Interface | IP Address | Network |
|---|---|---|---|
| ISP-R1 | G0/0 | 10.0.0.1/30 | 10.0.0.0/30 |
| ISP-R2 | G0/0 | 10.0.0.2/30 | 10.0.0.0/30 |
| ISP-R2 | G0/1 | 10.0.1.1/30 | 10.0.1.0/30 |
| SW1 | G0/2 | 10.0.1.2/30 | 10.0.1.0/30 |

### Customer VLANs

| VLAN | Name | Network | Gateway |
|---|---|---|---|
| 10 | CUSTOMER-A | 192.168.10.0/24 | 192.168.10.1 |
| 20 | CUSTOMER-B | 192.168.20.0/24 | 192.168.20.1 |
| 30 | CUSTOMER-C | 192.168.30.0/24 | 192.168.30.1 |
| 40 | CUSTOMER-D | 192.168.40.0/24 | 192.168.40.1 |
| 99 | MANAGEMENT | 192.168.99.0/24 | 192.168.99.1 |

### PCs

| PC | VLAN | IP Address | Mask | Gateway | DNS |
|---|---:|---|---|---|---|
| PC1 | 10 | 192.168.10.10 | 255.255.255.0 | 192.168.10.1 | 8.8.8.8 |
| PC2 | 20 | 192.168.20.10 | 255.255.255.0 | 192.168.20.1 | 8.8.8.8 |
| PC3 | 30 | 192.168.30.10 | 255.255.255.0 | 192.168.30.1 | 8.8.8.8 |
| PC4 | 40 | 192.168.40.10 | 255.255.255.0 | 192.168.40.1 | 8.8.8.8 |

## Cabling

Use **Copper Straight-Through** cables for the Packet Tracer connections.

| From | Interface | To | Interface |
|---|---|---|---|
| ISP-R1 | G0/0 | ISP-R2 | G0/0 |
| ISP-R2 | G0/1 | SW1 | G0/2 |
| SW1 | G0/1 | SW2 | G0/1 |
| SW1 | Fa0/24 | SW3 | Fa0/24 |
| SW2 | Fa0/1 | PC1 | FastEthernet |
| SW2 | Fa0/2 | PC2 | FastEthernet |
| SW3 | Fa0/1 | PC3 | FastEthernet |
| SW3 | Fa0/2 | PC4 | FastEthernet |

# Configuration

## ISP-R1

```cisco
enable
configure terminal

interface gigabitethernet0/0
 ip address 10.0.0.1 255.255.255.252
 no shutdown
exit

ip route 192.168.0.0 255.255.0.0 10.0.0.2
ip route 10.0.1.0 255.255.255.252 10.0.0.2

end
write memory
```

## ISP-R2

```cisco
enable
configure terminal

interface gigabitethernet0/0
 ip address 10.0.0.2 255.255.255.252
 no shutdown
exit

interface gigabitethernet0/1
 ip address 10.0.1.1 255.255.255.252
 no shutdown
exit

ip route 192.168.10.0 255.255.255.0 10.0.1.2
ip route 192.168.20.0 255.255.255.0 10.0.1.2
ip route 192.168.30.0 255.255.255.0 10.0.1.2
ip route 192.168.40.0 255.255.255.0 10.0.1.2

end
write memory
```

## SW1 — Layer 3 Distribution Switch

### Enable Layer 3 Routing

```cisco
enable
configure terminal

ip routing
```

### Create VLANs

```cisco
vlan 10
 name CUSTOMER-A
exit

vlan 20
 name CUSTOMER-B
exit

vlan 30
 name CUSTOMER-C
exit

vlan 40
 name CUSTOMER-D
exit

vlan 99
 name MANAGEMENT
exit
```

### Routed Link to ISP-R2

```cisco
interface gigabitethernet0/2
 no switchport
 ip address 10.0.1.2 255.255.255.252
 no shutdown
exit

ip route 0.0.0.0 0.0.0.0 10.0.1.1
```

### SVIs / VLAN Gateways

```cisco
interface vlan 10
 ip address 192.168.10.1 255.255.255.0
 no shutdown
exit

interface vlan 20
 ip address 192.168.20.1 255.255.255.0
 no shutdown
exit

interface vlan 30
 ip address 192.168.30.1 255.255.255.0
 no shutdown
exit

interface vlan 40
 ip address 192.168.40.1 255.255.255.0
 no shutdown
exit

interface vlan 99
 ip address 192.168.99.1 255.255.255.0
 no shutdown
exit
```

### Trunk to SW2

```cisco
interface gigabitethernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40,99
 no shutdown
exit
```

### Trunk to SW3

```cisco
interface fastethernet0/24
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40,99
 no shutdown
exit

end
write memory
```

## SW2 — Layer 2 Access Switch

```cisco
enable
configure terminal

vlan 10
 name CUSTOMER-A
exit

vlan 20
 name CUSTOMER-B
exit

vlan 30
 name CUSTOMER-C
exit

vlan 40
 name CUSTOMER-D
exit

vlan 99
 name MANAGEMENT
exit

interface gigabitethernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40,99
 no shutdown
exit

interface fastethernet0/1
 switchport mode access
 switchport access vlan 10
 no shutdown
exit

interface fastethernet0/2
 switchport mode access
 switchport access vlan 20
 no shutdown
exit

end
write memory
```

## SW3 — Layer 2 Access Switch

```cisco
enable
configure terminal

vlan 10
 name CUSTOMER-A
exit

vlan 20
 name CUSTOMER-B
exit

vlan 30
 name CUSTOMER-C
exit

vlan 40
 name CUSTOMER-D
exit

vlan 99
 name MANAGEMENT
exit

interface fastethernet0/24
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40,99
 no shutdown
exit

interface fastethernet0/1
 switchport mode access
 switchport access vlan 30
 no shutdown
exit

interface fastethernet0/2
 switchport mode access
 switchport access vlan 40
 no shutdown
exit

end
write memory
```

# Verification

## ISP-R1

```cisco
show ip interface brief
show ip route
ping 10.0.0.2
ping 10.0.1.2
```

## ISP-R2

```cisco
show ip interface brief
show ip route
ping 10.0.0.1
ping 10.0.1.2
```

## SW1

```cisco
show ip interface brief
show vlan brief
show interfaces trunk
show ip route
show cdp neighbors
```

Test upstream connectivity:

```cisco
ping 10.0.1.1
ping 10.0.0.2
ping 10.0.0.1
```

Test customer gateways:

```cisco
ping 192.168.10.1
ping 192.168.20.1
ping 192.168.30.1
ping 192.168.40.1
```

Test PCs:

```cisco
ping 192.168.10.10
ping 192.168.20.10
ping 192.168.30.10
ping 192.168.40.10
```

# NOC Troubleshooting Methodology

Use a structured approach when investigating a customer fault:

```text
Customer Report
      |
      v
1. Physical / Link
      |
      v
2. Interface Status
      |
      v
3. VLAN Assignment
      |
      v
4. Trunk Configuration
      |
      v
5. IP Address
      |
      v
6. Default Gateway
      |
      v
7. ARP
      |
      v
8. Routing Table
      |
      v
9. Upstream Connectivity
      |
      v
10. DNS / Application
```

## Useful Commands

### Interface status

```cisco
show ip interface brief
```

### VLANs

```cisco
show vlan brief
```

### Trunks

```cisco
show interfaces trunk
```

### MAC addresses

```cisco
show mac address-table
```

### ARP

```cisco
show arp
```

### Routing

```cisco
show ip route
```

### Neighbors

```cisco
show cdp neighbors
```

### Detailed interfaces

```cisco
show interfaces
```

### Connectivity

```cisco
ping <IP-address>
traceroute <IP-address>
```

# Troubleshooting Case: Missing Return Route

During the lab, SW1 could reach ISP-R2 but initially could not reach ISP-R1.

SW1:

```text
SW1#ping 10.0.0.1

.....
Success rate is 0 percent
```

ISP-R2 could reach ISP-R1:

```text
ISP-R2#ping 10.0.0.1

!!!!!
Success rate is 100 percent
```

The investigation showed that ISP-R1 had no route to the `10.0.1.0/30` network:

```text
ISP-R1#show ip route 10.0.1.2

% Subnet not in table
```

The missing route was added:

```cisco
ip route 10.0.1.0 255.255.255.252 10.0.0.2
```

After the correction:

```text
ISP-R1#ping 10.0.1.2

!!!!!
Success rate is 100 percent
```

And:

```text
SW1#ping 10.0.0.1

!!!!!
Success rate is 100 percent
```

## Key NOC Lesson

A packet may successfully travel toward a destination while the reply cannot return.

Therefore:

> Always verify both the forward path and the return path.

# Expected End-to-End Path

Traffic from PC1 toward the ISP:

```text
PC1
 |
 | VLAN 10
 |
SW2
 |
 | 802.1Q Trunk
 |
SW1
 |
 | 10.0.1.0/30
 |
ISP-R2
 |
 | 10.0.0.0/30
 |
ISP-R1
```

Return traffic:

```text
ISP-R1
   |
ISP-R2
   |
SW1
   |
SW2
   |
PC1
```

Both directions require valid connectivity and routing.

# Skills Practiced

- Cisco IOS
- IPv4 addressing
- Subnetting
- Layer 2 switching
- Layer 3 switching
- VLAN configuration
- Access ports
- 802.1Q trunking
- SVI configuration
- Inter-VLAN routing
- Static routing
- Default routing
- ARP
- ICMP
- CDP
- Routing-table analysis
- Forward-path troubleshooting
- Return-path troubleshooting
- NOC fault isolation
- ISP network fundamentals

# Future Improvements

The lab can be expanded with:

- OSPF
- EIGRP
- BGP
- NAT/PAT
- DHCP
- DNS
- ACLs
- SSH management
- Port security
- SNMP
- Syslog
- EtherChannel
- HSRP
- QoS
- PPP/CHAP
- Redundant links
- Customer isolation
- GPON/OLT concepts
- Network monitoring
- Automated NOC fault scenarios

# Project Structure

Recommended GitHub structure:

```text
Cisco-ISP-NOC-Lab/
│
├── README.md
│
├── topology/
│   └── ISP-NOC-Lab.pkt
│
├── configs/
│   ├── ISP-R1.txt
│   ├── ISP-R2.txt
│   ├── SW1.txt
│   ├── SW2.txt
│   └── SW3.txt
│
├── documentation/
│   ├── IP-Addressing-Plan.md
│   └── Troubleshooting-Cases.md
│
└── screenshots/
    ├── topology.png
    ├── routing.png
    └── vlan-trunks.png
```

# Conclusion

This Packet Tracer project provides a practical foundation for ISP and NOC networking work. It combines routing, switching, VLANs, trunking, inter-VLAN routing, and structured troubleshooting.

The most important troubleshooting principle demonstrated by the project is:

**Always verify the forward path and the return path.**
