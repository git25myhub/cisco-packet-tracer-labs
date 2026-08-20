# EtherChannel, Layer 3 EtherChannel & Load Balancing Lab

## Overview

This lab focuses on configuring and verifying EtherChannel using different negotiation methods, establishing a Layer 3 EtherChannel between multilayer switches, configuring static routes, and modifying the EtherChannel load-balancing method.

The lab uses:

- **LACP** for the Layer 2 EtherChannel between ASW1 and DSW1
- **PAgP** for the Layer 2 EtherChannel between ASW2 and DSW2
- **Static EtherChannel (`mode on`)** for the Layer 3 EtherChannel between DSW1 and DSW2
- **Static routing** to provide connectivity between the user networks
- **Source-and-destination IP load balancing**

End-host and SVI IP addresses were pre-configured before beginning the lab.

---

## Lab Objectives

1. Configure a Layer 2 EtherChannel between **ASW1 and DSW1** using **LACP**.
2. Configure the EtherChannel as a trunk.
3. Configure a Layer 2 EtherChannel between **ASW2 and DSW2** using **PAgP**.
4. Configure the second EtherChannel as a trunk.
5. Configure a Layer 3 EtherChannel between **DSW1 and DSW2** using static EtherChannel.
6. Configure routes so that PCs can reach **SRV1**.
7. Identify the default EtherChannel load-balancing method.
8. Configure all switches to load-balance using source and destination IP addresses.

---

## Topology

```text
             Layer 2 EtherChannel
              LACP / Trunk
       ┌─────────────────────────┐
       │                         │
     ASW1 ===================== DSW1
                                  ||
                                  || Layer 3 EtherChannel
                                  || Static / mode on
                                  ||
     ASW2 ===================== DSW2
       │                         │
       └─────────────────────────┘
             Layer 2 EtherChannel
              PAgP / Trunk
```

The Layer 3 EtherChannel uses the `10.0.0.0/30` point-to-point network:

| Device | Interface | IP Address |
|---|---|---|
| DSW1 | Port-channel2 | `10.0.0.1/30` |
| DSW2 | Port-channel2 | `10.0.0.2/30` |

The user networks are:

| Device | Network | Gateway |
|---|---|---|
| DSW1 | `172.16.1.0/24` | `172.16.1.254` |
| DSW2 | `172.16.2.0/24` | `172.16.2.254` |

---

# 1. ASW1 ↔ DSW1 — LACP Layer 2 EtherChannel

The first EtherChannel uses **LACP**.

ASW1 was configured with both physical interfaces as trunks and placed into **channel-group 1 in active mode**:

```cisco
interface range GigabitEthernet0/1 - 2
 switchport mode trunk
 channel-group 1 mode active
```

The resulting logical interface was configured as a trunk:

```cisco
interface Port-channel1
 switchport mode trunk
```

Verification on DSW1 confirmed that the EtherChannel was operational:

```text
Group  Port-channel  Protocol    Ports
1      Po1(SU)       LACP        Gig1/0/3(P) Gig1/0/4(P)
```

`SU` indicates that Port-channel 1 is:

- **S** — Layer 2
- **U** — In use

The member ports were successfully bundled using LACP.

---

# 2. ASW2 ↔ DSW2 — PAgP Layer 2 EtherChannel

The second Layer 2 EtherChannel uses **PAgP**.

ASW2 was configured with its two GigabitEthernet interfaces in **PAgP desirable mode**:

```cisco
interface range GigabitEthernet0/1 - 2
 channel-group 1 mode desirable
```

The Port-channel was then configured as a trunk:

```cisco
interface Port-channel1
 switchport mode trunk
```

The final configuration showed:

```text
interface Port-channel1
 switchport mode trunk
```

and:

```text
interface GigabitEthernet0/1
 switchport mode trunk
 channel-group 1 mode desirable

interface GigabitEthernet0/2
 switchport mode trunk
 channel-group 1 mode desirable
```

PAgP initially reported suspended interfaces because the remote side had not yet enabled PAgP. Once the corresponding configuration was applied, Port-channel 1 transitioned to the **up/up** state.

---

# 3. DSW1 ↔ DSW2 — Layer 3 EtherChannel

The connection between DSW1 and DSW2 was configured as a **Layer 3 EtherChannel** using static EtherChannel.

Unlike LACP and PAgP, static EtherChannel does not use a negotiation protocol. Both sides must therefore use:

```cisco
channel-group 2 mode on
```

The physical interfaces were converted to Layer 3 interfaces:

```cisco
interface range GigabitEthernet1/0/1 - 2
 no switchport
 channel-group 2 mode on
```

The logical Port-channel interface was then assigned the Layer 3 address.

### DSW1

```cisco
interface Port-channel2
 no switchport
 ip address 10.0.0.1 255.255.255.252
```

### DSW2

```cisco
interface Port-channel2
 no switchport
 ip address 10.0.0.2 255.255.255.252
```

The resulting EtherChannel summary on DSW1 showed:

```text
Group  Port-channel  Protocol    Ports
2      Po2(RU)       -           Gig1/0/1(P) Gig1/0/2(P)
```

The flags indicate:

- **R** — Layer 3
- **U** — In use
- **P** — Port is bundled in the EtherChannel



The Layer 3 connection was verified by pinging DSW2:

```text
DSW1# ping 10.0.0.2

Success rate is 80 percent (4/5)
```

The first packet loss is consistent with initial ARP resolution in Packet Tracer.

---

# 4. Routing Between the Networks

Both multilayer switches were configured for Layer 3 routing:

```cisco
ip routing
```

## DSW1

DSW1 directly connects to:

```text
172.16.1.0/24
```

through:

```text
Vlan1 = 172.16.1.254
```

A static route was added for the DSW2 network:

```cisco
ip route 172.16.2.0 255.255.255.0 10.0.0.2
```

Therefore, traffic destined for `172.16.2.0/24` is forwarded to DSW2 through `10.0.0.2`.

## DSW2

DSW2 directly connects to:

```text
172.16.2.0/24
```

through:

```text
Vlan1 = 172.16.2.254
```

The reverse static route was configured:

```cisco
ip route 172.16.1.0 255.255.255.0 10.0.0.1
```

This allows DSW2 to reach the network behind DSW1.

---

# 5. EtherChannel Load Balancing

## Default Load-Balancing Method

Before modification, ASW1 reported:

```text
EtherChannel Load-Balancing Operational State (src-mac):

Non-IP: Source MAC address
IPv4:   Source MAC address
IPv6:   Source MAC address
```

Therefore, the default EtherChannel load-balancing method observed in this lab was:

```text
src-mac
```

Traffic was initially distributed based on the **source MAC address**.

---

# 6. Configure Source-and-Destination IP Load Balancing

All switches were configured to use source and destination IP addresses for EtherChannel load balancing:

```cisco
port-channel load-balance src-dst-ip
```

After configuration, verification showed:

```text
EtherChannel Load-Balancing Operational State (src-dst-ip):

Non-IP: Source XOR Destination MAC address
IPv4:   Source XOR Destination IP address
IPv6:   Source XOR Destination IP address
```

This was configured on ASW1, ASW2, DSW1, and DSW2.

On the multilayer switches, the final configuration confirmed:

```text
port-channel load-balance src-dst-ip
```



and:

```text
port-channel load-balance src-dst-ip
```

on DSW2 as well.

---

# Verification Commands

Useful commands for verifying the lab include:

### Verify EtherChannel

```cisco
show etherchannel summary
```

### Verify EtherChannel load balancing

```cisco
show etherchannel load-balance
```

### Verify trunk interfaces

```cisco
show interfaces trunk
```

### Verify the running configuration

```cisco
show running-config
```

### Verify routing

```cisco
show ip route
```

### Test Layer 3 connectivity

```cisco
ping 10.0.0.2
```

or:

```cisco
ping 10.0.0.1
```

---

# Final Configuration Summary

| Link | EtherChannel Type | Protocol/Mode | Layer | Purpose |
|---|---|---|---|---|
| ASW1 ↔ DSW1 | EtherChannel 1 | LACP | Layer 2 | Trunk |
| ASW2 ↔ DSW2 | EtherChannel 1 | PAgP | Layer 2 | Trunk |
| DSW1 ↔ DSW2 | EtherChannel 2 | Static `mode on` | Layer 3 | Routed link |

### Layer 3 Addressing

```text
DSW1 Po2: 10.0.0.1/30
DSW2 Po2: 10.0.0.2/30
```

### User Networks

```text
DSW1 VLAN 1: 172.16.1.0/24
DSW2 VLAN 1: 172.16.2.0/24
```

### Static Routes

```text
DSW1 → 172.16.2.0/24 via 10.0.0.2
DSW2 → 172.16.1.0/24 via 10.0.0.1
```

### EtherChannel Load Balancing

```text
src-dst-ip
```

---

# Connectivity Test

A PC on the `172.16.2.0/24` network was able to reach `172.16.2.1` successfully:

```text
C:\>ping 172.16.2.1

Reply from 172.16.2.1: bytes=32 time=15ms TTL=126
Reply from 172.16.2.1: bytes=32 time=11ms TTL=126
Reply from 172.16.2.1: bytes=32 time=7ms TTL=126
```

The test recorded **3 successful replies out of 4 packets**, with 25% packet loss.

---

## Key Takeaways

- **LACP** provides standards-based EtherChannel negotiation.
- **PAgP** is Cisco's EtherChannel negotiation protocol.
- **Static EtherChannel (`mode on`)** forms the bundle without negotiation.
- A Layer 2 EtherChannel can operate as a **trunk**.
- A Layer 3 EtherChannel can operate as a **routed point-to-point connection**.
- `ip routing` enables Layer 3 routing on the multilayer switches.
- Static routes provide reachability between the two user networks.
- The default load-balancing method observed was **source MAC**.
- `src-dst-ip` provides load balancing based on the source and destination IP addresses.