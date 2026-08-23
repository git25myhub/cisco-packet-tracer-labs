# Cisco Packet Tracer Labs 🌐

A hands-on networking laboratory repository containing Cisco Packet Tracer projects for practicing **CCNA-level networking, routing, switching, network services, security, troubleshooting, and enterprise network design**.

The repository is built around practical configuration and troubleshooting rather than theory alone. It contains individual labs progressing from foundational networking concepts to advanced enterprise scenarios, including a comprehensive **Jeremy's IT Lab CCNA Mega Lab**.

> **Repository:** https://github.com/git25myhub/cisco-packet-tracer-labs

---

## 📌 About This Repository

This repository documents my practical networking journey using **Cisco Packet Tracer**.

The labs are designed to build skills in:

- Cisco IOS configuration
- IPv4 and IPv6 addressing
- Subnetting and VLSM
- Ethernet switching
- VLANs and trunking
- DTP and VTP
- STP / RSTP
- EtherChannel
- Inter-VLAN routing
- Layer-3 switching
- Static and dynamic routing
- OSPF and EIGRP
- HSRP
- ACLs
- NAT and PAT
- DHCP, DNS, NTP and Syslog
- SNMP
- SSH
- FTP/TFTP
- Voice VLANs
- QoS
- Port Security
- DHCP Snooping
- Dynamic ARP Inspection
- CDP and LLDP
- IPv6 routing
- Wireless LANs
- GRE tunnels
- Network troubleshooting
- Packet-flow analysis
- Packet capture and Wireshark-based analysis

The repository also includes larger multi-device scenarios intended to simulate the type of work encountered in enterprise networking and NOC environments.

---

# 🗂️ Repository Structure

The repository currently contains a combination of individual labs and larger projects.

```text
cisco-packet-tracer-labs/
│
├── CCNA Mega Lab (Jeremy's IT Lab)/
│   └── Full enterprise-style CCNA mega lab
│
├── Day 01 Lab - Packet Tracer Introduction/
├── Day 08 Lab - IPv4 Addresses/
├── Day 09 Lab - Interface Configuration/
├── Day 11 Lab - Configuring Static Routes/
├── Day 11 Lab - Troubleshooting Static Routes/
├── Day 12 Lab - Life of a Packet/
├── Day 15 Lab - VLSM/
├── Day 16 Lab - VLANs (Part 1)/
├── Day 17 Lab - VLANs (Part 2)/
├── Day 18 Lab - Multilayer Switching/
├── Day 19 Lab - DTP & VTP/
├── Day 20 Lab - Analyzing STP/
├── Day 21 Lab - Configuring Spanning Tree/
├── Day 22 Lab - Rapid STP/
├── Day 23 Lab - EtherChannel/
├── Day 24 Lab - Floating Static Routes/
├── Day 25 Lab - EIGRP Configuration/
├── Day 26 Lab - OSPF (Part 1)/
├── Day 27 Lab - OSPF (Part 2)/
├── Day 28 Lab - OSPF (Part 3)/
├── Day 29 Lab - HSRP Configuration/
├── Day 31 Lab - IPv6 Configuration (Part 1)/
├── Day 32 Lab - IPv6 Configuration (Part 2)/
├── Day 33 Lab - IPv6 Static Routes/
├── Day 34 Lab - Standard ACLs/
├── Day 35 Lab - Extended ACLs/
├── Day 36 Lab - CDP & LLDP/
├── Day 37 Lab - NTP/
├── Day 38 Lab - DNS/
├── Day 39 Lab - DHCP/
├── Day 40 Lab - SNMP/
├── Day 41 Lab - Syslog/
├── Day 42 Lab - SSH/
├── Day 43 Lab - FTP & TFTP/
├── Day 44 Lab - Static NAT/
├── Day 45 Lab - Dynamic NAT/
├── Day 46 Lab - Voice VLANs/
├── Day 47 Lab - QoS/
├── Day 49 Lab - Port Security/
├── Day 50 Lab - DHCP Snooping/
├── Day 51 Lab - Dynamic ARP Inspection/
├── Day 52 Lab - STP & HSRP Synchronization/
├── Day 53 Lab - GRE Tunnels/
├── Day 58 Lab - Wireless LANs/
│
└── *.pkt
    └── Cisco Packet Tracer activity files
```

The exact repository contents may grow as additional labs are completed.

---

# 🚀 Learning Path

The labs are intended to be approached progressively.

## Phase 1 — Networking Fundamentals

Start with:

1. Packet Tracer Introduction
2. Connecting Devices
3. OSI Model
4. Basic Device Security
5. Ethernet LAN Switching
6. IPv4 Addresses
7. Interface Configuration
8. VLSM
9. Life of a Packet

These labs establish the fundamentals required for later routing and switching work.

---

## Phase 2 — Switching

Switching-focused labs include:

- VLANs
- Trunking
- DTP
- VTP
- STP
- Rapid STP
- EtherChannel
- Multilayer Switching
- Voice VLANs
- QoS

Important concepts practiced include:

```text
Access Ports
      ↓
VLAN Assignment
      ↓
802.1Q Trunking
      ↓
STP / RSTP
      ↓
EtherChannel
      ↓
Layer-3 Switching
```

---

# 🌐 Routing

Routing labs cover both static and dynamic routing.

## Static Routing

Topics include:

- Static routes
- Default routes
- Floating static routes
- Recursive next-hop routes
- Fully specified routes
- Troubleshooting routing failures

## Dynamic Routing

The repository includes practical work with:

- OSPF
- EIGRP
- OSPF neighbor relationships
- OSPF router IDs
- OSPF areas
- Passive interfaces
- Route advertisement
- Default route propagation

---

# 🔀 High Availability

The labs include **HSRP** and STP/HSRP synchronization.

Skills practiced include:

- HSRP groups
- Virtual IP addresses
- Active/Standby roles
- HSRP priority
- Preemption
- STP root bridge selection
- Aligning STP root and HSRP active devices

These concepts are especially important when designing resilient enterprise LANs.

---

# 🔐 Network Security

Security-focused labs cover:

### Access Control Lists

- Standard ACLs
- Extended ACLs
- Source-based filtering
- Destination-based filtering
- Protocol-specific filtering
- ACL placement

### Layer-2 Security

- Port Security
- Sticky MAC addresses
- Violation modes
- DHCP Snooping
- Dynamic ARP Inspection
- Trusted/untrusted ports
- DHCP rate limiting
- BPDU Guard
- PortFast

Example security model:

```text
                    Core
                      |
                Distribution
                      |
              ----------------
              |              |
          Trusted         Trusted
              |              |
         Access Switch
              |
        Untrusted Edge
              |
          End Host
```

---

# 🧭 IPv6

IPv6 labs cover:

- IPv6 addressing
- IPv6 routing
- IPv6 static routes
- IPv6 default routes
- EUI-64
- IPv6-enabled interfaces
- Dual-stack concepts

Example address format:

```text
2001:db8::/32
```

---

# 🛠️ Network Services

The repository contains practical labs for common infrastructure services.

## DHCP

Practice includes:

- DHCP pools
- Excluded addresses
- Default gateways
- DNS server options
- DHCP relay
- Centralized DHCP

## DNS

Practice includes:

- DNS records
- Name resolution
- Hostname-to-IP mapping

## NTP

Practice includes:

- NTP servers
- Stratum
- NTP clients
- Authentication

## Syslog

Practice includes:

- Centralized logging
- Log severity levels
- Logging buffers
- Syslog servers

## SNMP

Practice includes:

- SNMP community strings
- Read-only monitoring
- Network management concepts

## SSH

Practice includes:

- Local authentication
- RSA keys
- SSH version 2
- VTY configuration
- Access control
- Secure remote administration

## NAT

Practice includes:

- Static NAT
- Dynamic NAT
- PAT
- Inside local/global addresses
- Internet connectivity

---

# 📡 Wireless Networking

Wireless labs cover:

- Wireless LAN Controllers
- Lightweight Access Points
- WLAN creation
- SSIDs
- WPA2 security
- AES encryption
- Pre-shared keys
- Dynamic interfaces
- Wireless VLANs

The repository includes a wireless LAN lab as well as wireless configuration inside the CCNA Mega Lab.

---

# 📞 Voice & QoS

Additional enterprise concepts include:

- Voice VLANs
- IP phones
- QoS fundamentals
- Traffic classification
- Network performance considerations

These labs demonstrate how data and voice traffic can coexist on the same switching infrastructure.

---

# 🔄 Network Discovery

The labs cover:

### CDP

Cisco Discovery Protocol is practiced for discovering directly connected Cisco devices.

### LLDP

Link Layer Discovery Protocol is used as the vendor-neutral alternative.

The labs also include disabling CDP and configuring LLDP where required.

---

# 🛰️ GRE Tunnels

The GRE lab provides practical exposure to:

- Tunnel interfaces
- Tunnel source/destination
- Encapsulation
- Routing through tunnels
- Site-to-site logical connectivity

Conceptually:

```text
Site A
Router
  |
  |================ GRE ================|
  |                                     |
Router                                Router
Site A                                  Site B
```

---

# 🧪 CCNA Mega Lab

The **CCNA Mega Lab (Jeremy's IT Lab)** is the largest project in this repository.

It combines many CCNA concepts into a single enterprise-style topology.

The topology includes:

```text
                         INTERNET
                            |
                           R1
                            |
                  -------------------
                  |                 |
                 CSW1-------------CSW2
                  |                 |
          --------+--------   ------+--------
          |                 |                |
       Office A          Office B
```

The complete lab introduces:

- Initial device configuration
- VLANs
- VTPv2
- DTP
- Trunking
- Layer-2 EtherChannel
- Layer-3 EtherChannel
- Inter-VLAN routing
- HSRPv2
- Rapid PVST+
- OSPF
- Static routing
- Floating static routes
- DHCP
- DNS
- NTP
- SNMP
- Syslog
- FTP
- SSH
- NAT
- PAT
- CDP
- LLDP
- ACLs
- Port Security
- DHCP Snooping
- Dynamic ARP Inspection
- IPv6
- Wireless LANs

The Mega Lab is intended to be completed progressively because configuration dependencies exist between sections.

### Mega Lab documentation

See:

```text
CCNA Mega Lab (Jeremy's IT Lab)/
```

for the topology, Packet Tracer activity and supporting documentation.

---

# 🧰 Tools Used

## Cisco Packet Tracer

Primary simulation environment for:

- Routers
- Switches
- PCs
- Servers
- Wireless controllers
- Lightweight access points
- IP phones
- Network services

## Wireshark

Used where appropriate for:

- Packet capture
- Protocol analysis
- Packet-flow investigation
- Troubleshooting
- Understanding network behavior

---

# 💻 Requirements

To work with this repository, you should have:

### Required

- Cisco Packet Tracer
- Basic networking knowledge
- Basic Cisco IOS CLI knowledge

### Recommended

- Wireshark
- Git
- A text editor or IDE
- Basic understanding of IPv4 subnetting
- Basic understanding of IPv6

---

# 📥 Getting Started

Clone the repository:

```bash
git clone https://github.com/git25myhub/cisco-packet-tracer-labs.git
```

Enter the repository:

```bash
cd cisco-packet-tracer-labs
```

Open Cisco Packet Tracer and open the required `.pkt` file.

---

# ▶️ How to Use a Lab

For each lab:

### 1. Open the Packet Tracer file

Open the relevant `.pkt` project.

### 2. Read the lab instructions

Read the README or instructions included with the lab where available.

### 3. Inspect the topology

Before configuring anything, identify:

- Routers
- Switches
- End hosts
- Server devices
- VLANs
- Links
- IP addressing
- Routing boundaries

### 4. Configure the devices

Use the Cisco IOS CLI.

Typical workflow:

```text
enable
configure terminal
!
configure hostname
!
configure interfaces
!
configure VLANs
!
configure routing
!
configure services
!
end
```

### 5. Verify

Do not assume that a configuration is correct simply because the command was accepted.

Use verification commands such as:

```text
show running-config
show ip interface brief
show interfaces
show vlan brief
show interfaces trunk
show etherchannel summary
show spanning-tree
show ip route
show ip ospf neighbor
show standby
show access-lists
show cdp neighbors
show lldp neighbors
```

### 6. Test connectivity

Use:

```text
ping
traceroute
```

and inspect packet behavior using Packet Tracer Simulation Mode when necessary.

---

# 🔍 Troubleshooting Methodology

These labs are also intended to develop a structured troubleshooting process.

When connectivity fails, avoid randomly changing configurations.

Use this workflow:

```text
1. Identify the failure
        ↓
2. Check physical/link status
        ↓
3. Check interface status
        ↓
4. Check VLAN assignment
        ↓
5. Check trunking
        ↓
6. Check MAC learning
        ↓
7. Check ARP
        ↓
8. Check IP addressing
        ↓
9. Check default gateway
        ↓
10. Check routing table
        ↓
11. Check ACLs
        ↓
12. Check NAT
        ↓
13. Check services
        ↓
14. Test again
```

Useful commands:

```text
show ip interface brief
show interfaces status
show interfaces switchport
show vlan brief
show interfaces trunk
show mac address-table
show ip arp
show ip route
show access-lists
show ip nat translations
show ip nat statistics
```

For OSPF:

```text
show ip ospf neighbor
show ip ospf interface
show ip protocols
show ip route ospf
```

For HSRP:

```text
show standby
show standby brief
```

For EtherChannel:

```text
show etherchannel summary
show interfaces port-channel
```

For STP:

```text
show spanning-tree
show spanning-tree vlan <vlan-id>
```

---

# 🧠 Networking Concepts Practiced

| Area | Concepts |
|---|---|
| Fundamentals | OSI, packet flow, Ethernet |
| IPv4 | Addressing, subnetting, VLSM |
| IPv6 | Addressing, EUI-64, routing |
| Switching | MAC learning, VLANs, trunks |
| VLANs | Access, trunk, native VLAN |
| DTP/VTP | Dynamic trunking and VLAN propagation |
| STP | STP, RSTP, root bridge |
| EtherChannel | LACP and PAgP |
| Layer 3 | SVIs, multilayer switching |
| Routing | Static, floating static, OSPF, EIGRP |
| High Availability | HSRP |
| Security | ACLs, Port Security, DHCP Snooping, DAI |
| Services | DHCP, DNS, NTP, SNMP, Syslog |
| Management | SSH, CDP, LLDP |
| NAT | Static NAT, Dynamic NAT, PAT |
| Wireless | WLC, LWAP, WLAN, WPA2 |
| Voice | Voice VLAN |
| QoS | Traffic handling and prioritization |
| Tunneling | GRE |
| Analysis | Packet Tracer Simulation Mode, Wireshark |

---

# 📊 CCNA Skills Matrix

This repository provides hands-on practice across the following areas:

```text
Networking Fundamentals        ████████████████████
IPv4 & Subnetting              ████████████████████
Switching & VLANs              ████████████████████
STP / RSTP                     ████████████████████
EtherChannel                   ████████████████████
Routing                        ████████████████████
OSPF                           ████████████████████
EIGRP                          ███████████████
HSRP                           ████████████████████
IPv6                           ███████████████████
ACLs                           ████████████████████
Network Security               ████████████████████
Network Services               ████████████████████
Wireless                       ███████████████████
Troubleshooting                ████████████████████
```

The bars represent areas practiced in the repository, not certification scores.

---

# 📝 Recommended Study Strategy

For maximum benefit, do not simply copy configurations.

For each lab:

### Step 1 — Understand

Identify what the lab is trying to accomplish.

### Step 2 — Design

Before typing commands, determine:

- IP addresses
- Subnets
- VLANs
- Interfaces
- Routing relationships
- Required protocols

### Step 3 — Configure

Enter the configuration manually.

### Step 4 — Verify

Use `show` commands.

### Step 5 — Break It

Intentionally introduce a small configuration error.

Examples:

- Wrong subnet mask
- Wrong default gateway
- Incorrect VLAN
- Incorrect trunk
- Missing route
- Wrong OSPF network
- Incorrect ACL
- Incorrect HSRP priority

### Step 6 — Troubleshoot

Find and correct the problem without rebuilding the topology.

This is particularly useful for developing NOC and network support skills.

---

# 🧪 Packet Tracer Simulation Mode

Packet Tracer's Simulation Mode can be used to observe packet behavior.

Useful protocols to investigate include:

- ARP
- ICMP
- DHCP
- DNS
- TCP
- UDP
- HTTP
- SSH
- Routing protocols

A useful workflow is:

```text
Create traffic
     ↓
Enter Simulation Mode
     ↓
Filter protocols
     ↓
Capture / Forward
     ↓
Inspect each layer
     ↓
Identify packet path
     ↓
Compare expected vs actual behavior
```

---

# 🦈 Wireshark Analysis

Where packet captures are available or applicable, Wireshark can be used to investigate traffic beyond what Packet Tracer displays.

Useful filters include:

```text
arp
icmp
dns
dhcp
tcp
udp
ssh
http
```

Packet analysis can help answer questions such as:

- Which device generated the packet?
- What source and destination IP addresses are used?
- What MAC addresses are present?
- Which protocol is being used?
- Is ARP resolving correctly?
- Is DNS responding?
- Is TCP establishing a session?
- Where is the packet being dropped?

---

# 📁 Lab File Naming

Lab folders generally follow a day/topic naming convention:

```text
Day XX Lab - <Topic>
```

Packet Tracer activity files use:

```text
.pkt
```

These files can be opened directly with Cisco Packet Tracer.

---

# ⚠️ Packet Tracer Limitations

Cisco Packet Tracer is a simulation environment and does not implement every behavior of real Cisco hardware.

Some commands, protocols, hardware behaviors, and services may therefore behave differently from production equipment.

For complex labs:

1. Save the configuration.
2. Save the Packet Tracer file.
3. If Packet Tracer becomes unstable, close it.
4. Reopen the project.
5. Verify configurations again.

For the CCNA Mega Lab in particular, large Packet Tracer topologies can consume significant resources.

---

# 💾 Saving Configurations

Always save device configurations when completing a lab.

Use:

```text
copy running-config startup-config
```

or:

```text
write memory
```

Verify with:

```text
show startup-config
```

A correct running configuration that has not been saved may be lost after a reload.

---

# 🔧 Common Cisco Verification Commands

## Interfaces

```text
show ip interface brief
show interfaces
show interfaces status
```

## VLANs

```text
show vlan brief
```

## Trunks

```text
show interfaces trunk
```

## EtherChannel

```text
show etherchannel summary
```

## STP

```text
show spanning-tree
```

## Routing

```text
show ip route
show ip protocols
```

## OSPF

```text
show ip ospf neighbor
show ip ospf interface
```

## EIGRP

```text
show ip eigrp neighbors
show ip protocols
```

## HSRP

```text
show standby brief
```

## DHCP

```text
show ip dhcp binding
show ip dhcp pool
```

## NAT

```text
show ip nat translations
show ip nat statistics
```

## ACLs

```text
show access-lists
```

## CDP / LLDP

```text
show cdp neighbors
show lldp neighbors
```

## SSH

```text
show ip ssh
show users
```

---

# 🏗️ Enterprise Networking Focus

The repository gradually moves from isolated exercises toward enterprise-style designs.

A typical enterprise architecture practiced in the larger labs resembles:

```text
                         INTERNET
                            |
                        Edge Router
                            |
                    -----------------
                    |               |
                Core SW1 -------- Core SW2
                    |               |
              -----------       -----------
              |         |       |         |
           Dist-A1   Dist-A2  Dist-B1   Dist-B2
              |         |       |         |
            Access    Access   Access    Access
              |         |       |         |
             PCs      Phones   PCs       Servers
```

This introduces important real-world concepts:

- Redundancy
- Routing
- VLAN segmentation
- First-hop redundancy
- Spanning-tree design
- Link aggregation
- Centralized services
- Security controls
- Monitoring
- Remote management

---

# 🎯 Objectives

The main goals of this repository are to:

- Build strong Cisco IOS CLI skills
- Develop practical networking knowledge
- Understand how packets move through networks
- Practice configuration and verification
- Develop systematic troubleshooting skills
- Build CCNA-level networking competence
- Create evidence of practical networking work
- Prepare for technical support, NOC, and network engineering roles
- Develop confidence working with enterprise-style network topologies

---

# 💼 Portfolio Value

This repository is intended to serve not only as a study collection but also as a practical networking portfolio.

It demonstrates hands-on exposure to:

```text
Configuration
     +
Verification
     +
Troubleshooting
     +
Network Design
     +
Security
     +
Network Services
```

The larger labs are particularly useful for demonstrating the ability to work with interconnected routers, multilayer switches, VLANs, routing protocols, redundancy, services, and security controls.

---

# 📈 Progress Tracking

A recommended progress system is:

- [ ] Lab completed
- [ ] Configuration verified
- [ ] Connectivity verified
- [ ] Troubleshooting performed
- [ ] Notes documented
- [ ] Packet behavior analyzed
- [ ] Configuration saved

For larger labs:

- [ ] Initial setup
- [ ] Switching
- [ ] Routing
- [ ] High availability
- [ ] Network services
- [ ] Security
- [ ] IPv6
- [ ] Wireless
- [ ] Final verification

---

# 🧩 Troubleshooting Checklist

When a host cannot communicate:

```text
[ ] Is the cable/link up?
[ ] Is the interface administratively up?
[ ] Is the correct VLAN assigned?
[ ] Is the VLAN present?
[ ] Is the trunk operational?
[ ] Is the native VLAN correct?
[ ] Are the required VLANs allowed?
[ ] Is the host IP correct?
[ ] Is the subnet mask correct?
[ ] Is the default gateway correct?
[ ] Is ARP resolving?
[ ] Is the MAC address being learned?
[ ] Is the routing table correct?
[ ] Is an ACL blocking traffic?
[ ] Is NAT configured correctly?
[ ] Is the required service running?
```

---

# 🔐 Security Notes

Do not use credentials from these educational labs in production environments.

Lab passwords and community strings are intentionally simple for learning and verification.

For real deployments:

- Use strong unique passwords.
- Use SSH instead of Telnet.
- Restrict management access.
- Use appropriate AAA.
- Protect management VLANs.
- Apply least-privilege access controls.
- Secure SNMP appropriately.
- Protect configuration backups.
- Keep IOS/software versions supported and patched.

---

# 📚 Reference

The repository includes labs based on Cisco Packet Tracer exercises and CCNA study material, including work following **Jeremy's IT Lab** concepts.

The Mega Lab is maintained as a practical, multi-topic networking project and should be treated as a large integrated lab rather than a collection of isolated commands.

---

# 🤝 Contributions

This repository is primarily a personal learning and portfolio project.

Suggestions, corrections, troubleshooting ideas, and improvements are welcome.

If contributing:

1. Create a branch.
2. Make your changes.
3. Test the Packet Tracer project.
4. Update the relevant documentation.
5. Commit the changes.
6. Open a pull request.

Example:

```bash
git checkout -b improve-lab-documentation
git add .
git commit -m "Improve lab documentation"
git push origin improve-lab-documentation
```

---

# 📌 Disclaimer

Cisco Packet Tracer is a simulation tool. Results in Packet Tracer may differ from behavior on physical Cisco equipment or other network platforms.

This repository is intended for:

- Education
- Practice
- Certification preparation
- Networking experimentation
- Portfolio development

It should not be treated as production network configuration without appropriate validation and adaptation.

---

# 👨‍💻 Author

**Stephen Kariuki**

GitHub: **git25myhub**

Focus areas:

- Networking
- Cisco technologies
- NOC / Technical Support
- Network troubleshooting
- Full-stack development
- IoT
- Systems engineering

---

# ⭐ Repository

If you find the labs useful, feel free to explore the repository and follow the progression from basic networking concepts to integrated enterprise networking scenarios.

**Cisco Packet Tracer Labs**

https://github.com/git25myhub/cisco-packet-tracer-labs

---

## 🚀 Keep Building. Keep Troubleshooting. Keep Learning.

```text
Learn
  ↓
Configure
  ↓
Verify
  ↓
Break
  ↓
Troubleshoot
  ↓
Understand
  ↓
Build Again
```

> Networking is not just about making devices communicate — it is about understanding **why** they communicate, **how** the traffic moves, and **where** it fails when something goes wrong.
