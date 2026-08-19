# Cisco VLAN, Trunking & Router-on-a-Stick Lab

## 📌 Lab Overview

This lab demonstrates the configuration and verification of a small Cisco switched network using:

- VLAN segmentation
- Access ports
- 802.1Q trunking
- Native VLAN configuration
- Restricted VLANs on trunks
- Router-on-a-Stick inter-VLAN routing
- Connectivity testing using ICMP/Ping
- Basic Cisco IOS verification and troubleshooting

The objective is to configure two switches (`SW1` and `SW2`) and one router (`R1`) so that PCs in different VLANs can communicate through the router.

---

## 🎯 Lab Objectives

By completing this lab, the following objectives should be achieved:

1. Configure switch interfaces connected to PCs as access ports in the correct VLAN.
2. Configure the link between `SW1` and `SW2` as an 802.1Q trunk.
3. Restrict the `SW1-SW2` trunk to only the required VLANs.
4. Configure an unused VLAN as the native VLAN.
5. Ensure all required VLANs exist on both switches.
6. Configure the `SW2-R1` connection as a trunk.
7. Configure Router-on-a-Stick using router subinterfaces.
8. Assign the last usable IP address in each subnet to the router subinterfaces.
9. Configure PCs with appropriate IP addresses and default gateways.
10. Verify connectivity between PCs across different VLANs.

---

# 🗺️ Network Design

The lab uses three VLANs:

| VLAN | Purpose | Subnet | Router Gateway |
|---|---|---|---|
| VLAN 10 | User Network | `10.0.0.0/26` | `10.0.0.62` |
| VLAN 20 | User Network | `10.0.0.64/26` | `10.0.0.126` |
| VLAN 30 | User Network | `10.0.0.128/26` | `10.0.0.190` |
| VLAN 1001 | Native VLAN | Unused | N/A |

### Subnet Details

Each `/26` subnet provides:

- 64 total addresses
- 62 usable host addresses
- Subnet mask: `255.255.255.192`

### Addressing Plan

| VLAN | Network Address | Usable Range | Broadcast | R1 Gateway |
|---|---|---|---|---|
| 10 | `10.0.0.0` | `10.0.0.1 – 10.0.0.62` | `10.0.0.63` | `10.0.0.62` |
| 20 | `10.0.0.64` | `10.0.0.65 – 10.0.0.126` | `10.0.0.127` | `10.0.0.126` |
| 30 | `10.0.0.128` | `10.0.0.129 – 10.0.0.190` | `10.0.0.191` | `10.0.0.190` |

The router uses the **last usable address** of each subnet, as required.

---

# 🔌 VLAN and Port Assignment

## SW1

| Interface | Device/Connection | Mode | VLAN |
|---|---|---|---|
| Fa0/1 | PC | Access | VLAN 10 |
| Fa0/2 | PC | Access | VLAN 10 |
| Fa0/3 | PC | Access | VLAN 30 |
| Fa0/4 | PC | Access | VLAN 30 |
| Gi0/1 | SW2 | Trunk | VLANs 10, 30 |
| Gi0/2 | Unused | — | — |

## SW2

| Interface | Device/Connection | Mode | VLAN |
|---|---|---|---|
| Fa0/1 | PC | Access | VLAN 20 |
| Fa0/2 | PC | Access | VLAN 10 |
| Fa0/3 | PC | Access | VLAN 10 |
| Gi0/1 | SW1 | Trunk | VLANs 10, 30 |
| Gi0/2 | R1 | Trunk | VLANs 10, 20, 30 |

---

# 🧱 VLAN Configuration

The necessary VLANs must exist on the switches before they can be used.

## SW1

```cisco
enable
configure terminal

vlan 10
 name VLAN0010
exit

vlan 30
 name VLAN0030
exit

vlan 1001
 name NATIVE_VLAN
exit
```

## SW2

```cisco
enable
configure terminal

vlan 10
 name VLAN0010
exit

vlan 20
 name VLAN0020
exit

vlan 30
 name VLAN0030
exit

vlan 1001
 name NATIVE_VLAN
exit
```

> **Important:** VLAN 30 initially did not exist on SW2. It was created with:
>
> ```cisco
> SW2(config)#vlan 30
> ```
>
> This was necessary because VLANs must exist on a switch before they can become active and forward traffic across a trunk.

---

# 🔹 SW1 Access Port Configuration

The PC-facing interfaces on SW1 are configured as access ports.

```cisco
interface range fa0/1-2
 switchport mode access
 switchport access vlan 10
exit

interface range fa0/3-4
 switchport mode access
 switchport access vlan 30
exit
```

### Verification

```cisco
show vlan brief
```

Expected result:

```text
10   VLAN0010   active    Fa0/1, Fa0/2
30   VLAN0030   active    Fa0/3, Fa0/4
```

---

# 🔹 SW2 Access Port Configuration

SW2 has PCs in VLANs 10 and 20.

```cisco
interface fa0/1
 switchport mode access
 switchport access vlan 20
exit

interface range fa0/2-3
 switchport mode access
 switchport access vlan 10
exit
```

### Verification

```cisco
show vlan brief
```

Expected result:

```text
10   VLAN0010   active    Fa0/2, Fa0/3
20   VLAN0020   active    Fa0/1
```

---

# 🔗 SW1 ↔ SW2 Trunk Configuration

The connection between SW1 and SW2 uses `GigabitEthernet0/1`.

Only VLANs 10 and 30 are required across this trunk.

## SW1

```cisco
interface gigabitEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,30
 switchport trunk native vlan 1001
exit
```

## SW2

```cisco
interface gigabitEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,30
 switchport trunk native vlan 1001
exit
```

---

# ⚠️ Native VLAN Mismatch Encountered

During configuration, SW1 reported:

```text
%CDP-4-NATIVE_VLAN_MISMATCH:
Native VLAN mismatch discovered on GigabitEthernet0/1
(1001), with SW2 GigabitEthernet0/1 (1).
```

This occurred because:

- SW1 was configured with native VLAN `1001`
- SW2 was still using the default native VLAN `1`

### Problem

```text
SW1 Gi0/1 → Native VLAN 1001
SW2 Gi0/1 → Native VLAN 1
```

### Solution

The native VLAN must match on both ends of the trunk.

On SW2:

```cisco
interface gigabitEthernet0/1
 switchport trunk native vlan 1001
exit
```

After the correction:

```text
SW1 Gi0/1 → Native VLAN 1001
SW2 Gi0/1 → Native VLAN 1001
```

The mismatch was therefore resolved.

### Verification

```cisco
show interfaces trunk
```

Expected:

```text
Port        Mode         Encapsulation  Status        Native vlan
Gi0/1       on           802.1q         trunking      1001
```

And:

```text
Port        Vlans allowed on trunk
Gi0/1       10,30
```

---

# 🔗 SW2 ↔ R1 Trunk Configuration

The connection between SW2 and R1 uses:

```text
SW2 Gi0/2 ↔ R1 Gi0/0
```

Because the router must receive traffic from multiple VLANs over a single physical interface, the SW2 interface is configured as a trunk.

## SW2

```cisco
interface gigabitEthernet0/2
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
 switchport trunk native vlan 1001
exit
```

### Verification

```cisco
show interfaces trunk
```

Expected:

```text
Gi0/2       on           802.1q         trunking      1001

Vlans allowed on trunk
Gi0/2       10,20,30
```

---

# 🌐 Router-on-a-Stick Configuration

Router-on-a-Stick allows a single physical router interface to route traffic between multiple VLANs using logical subinterfaces.

The physical interface is:

```text
R1 Gi0/0
```

The router uses three subinterfaces:

```text
Gi0/0.10 → VLAN 10
Gi0/0.20 → VLAN 20
Gi0/0.30 → VLAN 30
```

First, enable the physical interface:

```cisco
enable
configure terminal

interface gigabitEthernet0/0
 no shutdown
exit
```

---

## VLAN 10 Subinterface

VLAN 10 uses the subnet:

```text
10.0.0.0/26
```

The last usable address is:

```text
10.0.0.62
```

Configuration:

```cisco
interface gigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 10.0.0.62 255.255.255.192
exit
```

---

## VLAN 20 Subinterface

VLAN 20 uses:

```text
10.0.0.64/26
```

The last usable address is:

```text
10.0.0.126
```

Configuration:

```cisco
interface gigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 10.0.0.126 255.255.255.192
exit
```

---

## VLAN 30 Subinterface

VLAN 30 uses:

```text
10.0.0.128/26
```

The last usable address is:

```text
10.0.0.190
```

Configuration:

```cisco
interface gigabitEthernet0/0.30
 encapsulation dot1Q 30
 ip address 10.0.0.190 255.255.255.192
exit
```

Save the configuration:

```cisco
end
write memory
```

---

# 📋 Final R1 Configuration

The resulting router configuration contains:

```cisco
interface GigabitEthernet0/0
 no ip address
 duplex auto
 speed auto

interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 10.0.0.62 255.255.255.192

interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 10.0.0.126 255.255.255.192

interface GigabitEthernet0/0.30
 encapsulation dot1Q 30
 ip address 10.0.0.190 255.255.255.192
```

---

# 💻 PC Configuration

Each PC must be configured with an IP address belonging to its VLAN subnet.

The default gateway must point to the corresponding R1 subinterface.

### VLAN 10

Example:

```text
IP Address:      10.0.0.1
Subnet Mask:     255.255.255.192
Default Gateway: 10.0.0.62
```

### VLAN 20

Example:

```text
IP Address:      10.0.0.65
Subnet Mask:     255.255.255.192
Default Gateway: 10.0.0.126
```

### VLAN 30

Example:

```text
IP Address:      10.0.0.129
Subnet Mask:     255.255.255.192
Default Gateway: 10.0.0.190
```

---

# 🧪 Verification

## 1. Verify VLANs

On both switches:

```cisco
show vlan brief
```

Confirm that the required VLANs exist and that access ports are assigned correctly.

---

## 2. Verify Trunks

On both switches:

```cisco
show interfaces trunk
```

Confirm:

### SW1 Gi0/1

```text
Native VLAN: 1001
Allowed VLANs: 10,30
```

### SW2 Gi0/1

```text
Native VLAN: 1001
Allowed VLANs: 10,30
```

### SW2 Gi0/2

```text
Native VLAN: 1001
Allowed VLANs: 10,20,30
```

---

## 3. Verify Router Interfaces

On R1:

```cisco
show ip interface brief
```

Expected logical interfaces:

```text
GigabitEthernet0/0.10    10.0.0.62    up    up
GigabitEthernet0/0.20    10.0.0.126   up    up
GigabitEthernet0/0.30    10.0.0.190   up    up
```

---

# 🖥️ Connectivity Testing

Connectivity was tested from a PC using the Packet Tracer command prompt.

## Test VLAN 10 PC

First, the PC successfully reached another host in VLAN 10:

```text
C:\>ping 10.0.0.1

Reply from 10.0.0.1: bytes=32 time=15ms TTL=128
Reply from 10.0.0.1: bytes=32 time<1ms TTL=128
Reply from 10.0.0.1: bytes=32 time<1ms TTL=128
Reply from 10.0.0.1: bytes=32 time<1ms TTL=128

Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

This confirms connectivity within VLAN 10.

---

## Test VLAN 20

The PC then tested:

```text
C:\>ping 10.0.0.65
```

Result:

```text
Packets: Sent = 4
Received = 3
Lost = 1 (25% loss)
```

The first packet timed out, but subsequent packets succeeded.

This behavior can occur during the initial communication because the devices may need to resolve the destination MAC address using ARP before normal ICMP communication begins.

---

## Test VLAN 30

The PC also tested:

```text
C:\>ping 10.0.0.129
```

Result:

```text
Packets: Sent = 4
Received = 3
Lost = 1 (25% loss)
```

Again, the first packet timed out while subsequent packets succeeded.

This confirms that traffic was successfully routed between VLANs.

---

# 🔍 Troubleshooting Notes

## Native VLAN Mismatch

### Symptom

```text
%CDP-4-NATIVE_VLAN_MISMATCH
```

### Cause

The two ends of the trunk had different native VLANs.

### Resolution

Configure the same native VLAN on both ends:

```cisco
switchport trunk native vlan 1001
```

---

## VLAN Missing from Switch

During verification, SW2 initially showed:

```text
10   VLAN0010
20   VLAN0020
```

but VLAN 30 was missing.

The trunk allowed VLAN 30, but the VLAN did not yet exist on SW2.

### Resolution

```cisco
vlan 30
exit
```

After creating the VLAN, verification showed:

```text
Port        Vlans allowed on trunk
Gi0/1       10,30

Vlans allowed and active in management domain
Gi0/1       10,30
```

This confirmed that VLAN 30 was now active on the trunk.

---

# 🧠 Key Networking Concepts Demonstrated

## Access Port

An access port belongs to a single VLAN and is normally used to connect end devices such as PCs.

Example:

```cisco
switchport mode access
switchport access vlan 10
```

---

## Trunk Port

A trunk carries traffic for multiple VLANs over a single physical connection.

Example:

```cisco
switchport mode trunk
switchport trunk allowed vlan 10,30
```

---

## Native VLAN

The native VLAN is the VLAN used for untagged traffic on an 802.1Q trunk.

In this lab:

```text
Native VLAN = 1001
```

VLAN 1001 was selected because it is not being used by the PCs.

The native VLAN must match on both ends of a trunk.

---

## Router-on-a-Stick

Router-on-a-Stick uses one physical router interface with multiple logical subinterfaces.

For example:

```text
Gi0/0.10 → VLAN 10 → 10.0.0.62
Gi0/0.20 → VLAN 20 → 10.0.0.126
Gi0/0.30 → VLAN 30 → 10.0.0.190
```

The router can therefore route traffic between VLANs.

---

# 💾 Saving the Configuration

The configurations were saved using:

```cisco
do write
```

or:

```cisco
write memory
```

The expected confirmation is:

```text
Building configuration...

[OK]
```

---

# ✅ Final Verification Checklist

- [x] VLAN 10 created
- [x] VLAN 20 created
- [x] VLAN 30 created
- [x] VLAN 1001 created as unused native VLAN
- [x] SW1 PC ports configured as access ports
- [x] SW2 PC ports configured as access ports
- [x] SW1 ↔ SW2 configured as a trunk
- [x] SW1 ↔ SW2 restricted to VLANs 10 and 30
- [x] Native VLAN configured as VLAN 1001
- [x] Native VLAN mismatch corrected
- [x] SW2 ↔ R1 configured as a trunk
- [x] SW2 ↔ R1 allows VLANs 10, 20 and 30
- [x] Router-on-a-Stick configured
- [x] R1 assigned the last usable address in each subnet
- [x] VLAN 10 connectivity verified
- [x] VLAN 20 connectivity verified
- [x] VLAN 30 connectivity verified
- [x] Inter-VLAN routing verified
- [x] Configurations saved

---

# 🏁 Conclusion

This lab successfully demonstrated how VLANs can be used to segment a switched network while still allowing communication between VLANs through a router.

The main configuration components were:

```text
PCs
 │
 ├── Access Ports
 │
SW1
 │
 └── 802.1Q Trunk
       VLAN 10,30
       Native VLAN 1001
 │
SW2
 │
 └── 802.1Q Trunk
       VLAN 10,20,30
       Native VLAN 1001
 │
R1
 │
 ├── Gi0/0.10 → 10.0.0.62
 ├── Gi0/0.20 → 10.0.0.126
 └── Gi0/0.30 → 10.0.0.190
```

The lab also provided practical troubleshooting experience, particularly with **native VLAN mismatches** and **missing VLANs on trunk switches**. Successful ping tests confirmed that the configured VLANs, trunks, and Router-on-a-Stick inter-VLAN routing were functioning correctly.