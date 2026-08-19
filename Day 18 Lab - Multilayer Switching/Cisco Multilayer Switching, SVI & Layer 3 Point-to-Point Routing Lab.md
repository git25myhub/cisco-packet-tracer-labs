# Cisco Multilayer Switching, SVI & Layer 3 Point-to-Point Routing Lab

## 📌 Lab Overview

This lab builds on the previous day's network configuration. In this scenario, **SW2 has been replaced with a multilayer switch**, allowing it to perform Layer 3 routing between VLANs.

The previous design used **Router-on-a-Stick (ROAS)** between `R1` and `SW2`. The objective of this lab is to replace that ROAS configuration with a **point-to-point Layer 3 connection** and move the inter-VLAN routing function from the router to the multilayer switch.

The final design uses:

- `SW1` ↔ `SW2` — 802.1Q trunk
- `SW2` — Multilayer switch performing inter-VLAN routing
- `SW2` ↔ `R1` — Point-to-point Layer 3 connection
- SVIs on `SW2` — Default gateways for VLANs
- Static default route on `SW2` — Internet-bound traffic forwarded to `R1`
- Existing routing on `R1` and the Internet router — Used to provide Internet connectivity

---

# 🎯 Lab Objectives

The main objectives are:

1. Remove the existing ROAS configuration from `R1`.
2. Configure a point-to-point Layer 3 connection between `R1` and `SW2`.
3. Use the IP addresses specified in the network design.
4. Configure a default route on `SW2` pointing toward `R1`.
5. Enable Layer 3 routing on the multilayer switch.
6. Configure one SVI for each VLAN.
7. Assign the last usable IP address of each VLAN subnet to its SVI.
8. Use the SVIs as the default gateways for hosts.
9. Test inter-VLAN connectivity.
10. Test Internet connectivity by pinging `1.1.1.1`.

---

# 🗺️ Network Design

The final topology is:

```text
                 Internet
                    |
                Internet Rtr
                    |
                    |
                   R1
              G0/0/0 = 1.1.1.2
                    |
              G0/0 = 10.0.0.194
                    |
             Layer 3 P2P Link
                    |
              G1/0/2 = 10.0.0.193
                   SW2
            Multilayer Switch
             IP Routing Enabled
                    |
             802.1Q Trunk
                    |
                   SW1
                    |
                  Hosts
```

SW2 is now responsible for routing between VLANs.

---

# 📋 IP Addressing Plan

The VLAN subnets remain the same as in the previous lab.

| VLAN | Network | Subnet Mask | Last Usable Address | SW2 SVI |
|---|---|---|---|---|
| VLAN 10 | `10.0.0.0/26` | `255.255.255.192` | `10.0.0.62` | `10.0.0.62` |
| VLAN 20 | `10.0.0.64/26` | `255.255.255.192` | `10.0.0.126` | `10.0.0.126` |
| VLAN 30 | `10.0.0.128/26` | `255.255.255.192` | `10.0.0.190` | `10.0.0.190` |

The point-to-point Layer 3 link uses:

| Device | Interface | IP Address | Mask |
|---|---|---|---|
| SW2 | G1/0/2 | `10.0.0.193` | `/30` |
| R1 | G0/0 | `10.0.0.194` | `/30` |

The `/30` network is:

```text
Network:    10.0.0.192
SW2:        10.0.0.193
R1:         10.0.0.194
Broadcast:  10.0.0.195
```

---

# 🔄 Part 1 — Replace ROAS with a Layer 3 Connection

## Previous Design

In the previous lab, R1 used Router-on-a-Stick:

```text
R1
 |
 | 802.1Q Trunk
 |
SW2
 |
 +-- VLAN 10
 +-- VLAN 20
 +-- VLAN 30
```

R1 had subinterfaces such as:

```text
G0/0.10
G0/0.20
G0/0.30
```

These subinterfaces provided the default gateways for the VLANs.

---

# ❌ Remove ROAS Subinterfaces from R1

The existing ROAS subinterfaces were removed:

```cisco id="7vl8z7"
configure terminal

no interface gigabitEthernet0/0.10
no interface gigabitEthernet0/0.20
no interface gigabitEthernet0/0.30

exit
```

After removing the subinterfaces, R1 no longer provides the VLAN gateways.

Verification:

```cisco id="n6j8e1"
show ip interface brief
```

The physical interface appeared as:

```text
GigabitEthernet0/0     unassigned      up    up
```

---

# 🔗 Configure R1's Layer 3 Interface

The R1 interface connected to SW2 was configured with:

```text
IP Address: 10.0.0.194
Subnet Mask: 255.255.255.252
```

Configuration:

```cisco id="q55ddn"
configure terminal

interface gigabitEthernet0/0
 ip address 10.0.0.194 255.255.255.252
 no shutdown
exit

end
write memory
```

The final R1 interface configuration was:

```text
interface GigabitEthernet0/0
 ip address 10.0.0.194 255.255.255.252
 duplex auto
 speed auto
```

---

# 🔀 Configure SW2 as a Layer 3 Device

Because SW2 is now a multilayer switch, its interface connected to R1 must operate as a **routed Layer 3 interface**, rather than as a switchport.

The command used is:

```cisco id="k4l1zz"
interface gigabitEthernet1/0/2
 no switchport
```

This converts the physical switch interface from a Layer 2 switchport into a Layer 3 routed interface.

The IP address assigned to SW2 is:

```text
10.0.0.193/30
```

Configuration:

```cisco id="t8ljzw"
configure terminal

interface gigabitEthernet1/0/2
 no switchport
 ip address 10.0.0.193 255.255.255.252
 no shutdown
exit
```

The resulting configuration was:

```text
interface GigabitEthernet1/0/2
 no switchport
 ip address 10.0.0.193 255.255.255.252
 duplex auto
 speed auto
```

---

# 🌐 Enable IP Routing on SW2

A multilayer switch can operate at both Layer 2 and Layer 3.

To enable Layer 3 routing:

```cisco id="00c2g3"
ip routing
```

SW2's running configuration confirms:

```text
ip routing
```

This allows SW2 to route traffic between its VLAN interfaces and toward R1.

---

# 🛣️ Configure a Default Route on SW2

SW2 needs a route for destinations that are not part of its directly connected networks.

R1 is the next-hop router:

```text
R1 = 10.0.0.194
```

Therefore, SW2 was configured with:

```cisco id="4zz5ki"
ip route 0.0.0.0 0.0.0.0 10.0.0.194
```

This means:

> If SW2 does not have a more specific route for a destination, forward the traffic to R1 at `10.0.0.194`.

The resulting configuration was:

```text
ip classless
ip route 0.0.0.0 0.0.0.0 10.0.0.194
```

---

# 🔀 Part 2 — Configure SVIs on SW2

The multilayer switch now performs inter-VLAN routing.

Instead of R1 providing VLAN gateways using ROAS subinterfaces, SW2 provides the gateways using **Switch Virtual Interfaces (SVIs)**.

The required SVIs are:

```text
Vlan10
Vlan20
Vlan30
```

Each SVI receives the **last usable address** of its corresponding subnet.

---

# VLAN 10 SVI

VLAN 10 uses:

```text
Network:       10.0.0.0/26
Last usable:   10.0.0.62
```

Configuration:

```cisco id="wptg27"
interface vlan 10
 ip address 10.0.0.62 255.255.255.192
 no shutdown
exit
```

---

# VLAN 20 SVI

VLAN 20 uses:

```text
Network:       10.0.0.64/26
Last usable:   10.0.0.126
```

Configuration:

```cisco id="fytvgr"
interface vlan 20
 ip address 10.0.0.126 255.255.255.192
 no shutdown
exit
```

---

# VLAN 30 SVI

VLAN 30 uses:

```text
Network:       10.0.0.128/26
Last usable:   10.0.0.190
```

Configuration:

```cisco id="z1xpxa"
interface vlan 30
 ip address 10.0.0.190 255.255.255.192
 no shutdown
exit
```

---

# 📋 Final SVI Configuration

SW2's final SVI configuration was:

```text
interface Vlan10
 ip address 10.0.0.62 255.255.255.192

interface Vlan20
 ip address 10.0.0.126 255.255.255.192

interface Vlan30
 ip address 10.0.0.190 255.255.255.192
```

Verification using:

```cisco id="y1s4rg"
show ip interface brief
```

produced:

```text
Vlan10                 10.0.0.62       up    up
Vlan20                 10.0.0.126      up    up
Vlan30                 10.0.0.190      up    up
```

This confirms that the SVIs were operational.

---

# 🔗 SW1 ↔ SW2 Trunk

The connection between SW1 and SW2 remains a Layer 2 trunk.

SW2's trunk interface is:

```text
GigabitEthernet1/0/1
```

The configuration is:

```text
interface GigabitEthernet1/0/1
 switchport trunk allowed vlan 10,30
 switchport mode trunk
```

This allows VLAN traffic to travel between SW1 and the multilayer switch.

The Layer 3 routing occurs on SW2 through the SVIs.

---

# 🧠 How the New Architecture Works

The previous architecture was:

```text
PC
 |
SW1
 |
SW2
 |
Trunk
 |
R1
 |
ROAS Subinterfaces
 |
Inter-VLAN Routing
```

The new architecture is:

```text
PC
 |
SW1
 |
Trunk
 |
SW2
 |
 +-- SVI VLAN 10
 +-- SVI VLAN 20
 +-- SVI VLAN 30
 |
Layer 3 Routing
 |
Routed Link
 |
R1
 |
Internet
```

This is a significant architectural change.

### Previous Model

R1 performed inter-VLAN routing.

### New Model

SW2 performs inter-VLAN routing.

R1 is primarily used as the upstream router toward the Internet.

---

# 🧪 Part 3 — Test Inter-VLAN Connectivity

Inter-VLAN connectivity was tested from a PC using the Packet Tracer command prompt.

The PC successfully pinged a host in another VLAN:

```text
C:\>ping 10.0.0.129
```

Result:

```text
Pinging 10.0.0.129 with 32 bytes of data:

Request timed out.
Reply from 10.0.0.129: bytes=32 time<1ms TTL=127
Reply from 10.0.0.129: bytes=32 time<1ms TTL=127
Reply from 10.0.0.129: bytes=32 time<1ms TTL=127

Packets: Sent = 4
Received = 3
Lost = 1 (25% loss)
```

The successful replies demonstrate that traffic was able to travel between VLANs.

The first packet timeout is commonly seen in Packet Tracer when the devices initially need to resolve Layer 2 information through ARP.

---

# 🌍 Part 4 — Test Internet Connectivity

The lab requires testing connectivity to:

```text
1.1.1.1
```

The first test produced:

```text
C:\>ping 1.1.1.1

Request timed out.
Request timed out.
Request timed out.
Reply from 1.1.1.1: bytes=32 time<1ms TTL=253

Packets: Sent = 4
Received = 1
Lost = 3 (75% loss)
```

The test was repeated.

The second test produced:

```text
C:\>ping 1.1.1.1

Reply from 1.1.1.1: bytes=32 time<1ms TTL=253
Reply from 1.1.1.1: bytes=32 time<1ms TTL=253
Reply from 1.1.1.1: bytes=32 time<1ms TTL=253
Reply from 1.1.1.1: bytes=32 time<1ms TTL=253

Packets: Sent = 4
Received = 4
Lost = 0 (0% loss)
```

### Result

**Internet connectivity was successfully established.**

The `0% loss` result confirms that the end-to-end path to `1.1.1.1` was working.

---

# 🔍 Verification Commands

## Verify Layer 3 Interfaces

On SW2:

```cisco
show ip interface brief
```

Important interfaces:

```text
G1/0/2     10.0.0.193
Vlan10     10.0.0.62
Vlan20     10.0.0.126
Vlan30     10.0.0.190
```

---

## Verify Routing Table

On SW2:

```cisco
show ip route
```

The routing table should contain:

### Connected VLAN Networks

```text
10.0.0.0/26
10.0.0.64/26
10.0.0.128/26
```

### Point-to-Point Network

```text
10.0.0.192/30
```

### Default Route

```text
0.0.0.0/0 → 10.0.0.194
```

---

## Verify the Default Route

```cisco
show ip route
```

The default route should appear similar to:

```text
S* 0.0.0.0/0 [1/0] via 10.0.0.194
```

This confirms that SW2 forwards unknown destinations toward R1.

---

## Verify the Layer 3 Link

On SW2:

```cisco
show interfaces gigabitEthernet1/0/2
```

The interface should be:

```text
Status: up
Protocol: up
```

On R1:

```cisco
show ip interface brief
```

The corresponding interface should show:

```text
GigabitEthernet0/0    10.0.0.194    up    up
```

---

# 🔎 Verify Connectivity to R1

Before testing the Internet, SW2 can test the directly connected R1 address:

```cisco
ping 10.0.0.194
```

A successful response confirms that the point-to-point Layer 3 connection is working.

---

# 🧪 Recommended Verification Sequence

A useful troubleshooting sequence is:

### 1. Check VLANs

```cisco
show vlan brief
```

### 2. Check SVIs

```cisco
show ip interface brief
```

### 3. Check Layer 3 routing

```cisco
show ip route
```

### 4. Ping R1

```cisco
ping 10.0.0.194
```

### 5. Ping another VLAN host

```text
ping <remote-PC-IP>
```

### 6. Ping the Internet

```text
ping 1.1.1.1
```

This sequence helps isolate whether a problem is occurring at the VLAN, SVI, routing, upstream, or Internet level.

---

# 🛠️ Troubleshooting Notes

## ROAS Subinterfaces Still Present

If the old ROAS configuration remains on R1, the network may still be using the previous routing design.

The following interfaces were removed:

```cisco
no interface gigabitEthernet0/0.10
no interface gigabitEthernet0/0.20
no interface gigabitEthernet0/0.30
```

After removal, R1's physical G0/0 became the Layer 3 point-to-point interface.

---

## Layer 3 Interface on SW2

A common mistake when configuring a multilayer switch is attempting to assign an IP address to a normal Layer 2 switchport.

The interface must first be converted to a routed port:

```cisco
interface gigabitEthernet1/0/2
 no switchport
```

Only then can an IP address be assigned:

```cisco
ip address 10.0.0.193 255.255.255.252
```

---

## IP Routing on SW2

Without:

```cisco
ip routing
```

SW2 would not perform Layer 3 routing between the VLANs.

The running configuration confirms that routing was enabled:

```text
ip routing
```

---

## SVI Must Be Operational

The SVI must be both:

```text
up
up
```

For example:

```text
Vlan10    10.0.0.62    up    up
```

If an SVI is down, verify that the corresponding VLAN exists and that at least one active switchport belongs to that VLAN.

---

## Default Route

SW2 needs a default route because its directly connected routes only cover the local VLANs and the R1 point-to-point network.

The configured route is:

```cisco
ip route 0.0.0.0 0.0.0.0 10.0.0.194
```

Without this route, traffic destined for the Internet would not know where to go.

---

# 📊 Final Addressing Summary

| Device | Interface | Function | IP Address |
|---|---|---|---|
| SW2 | G1/0/2 | Routed link to R1 | `10.0.0.193/30` |
| R1 | G0/0 | Routed link to SW2 | `10.0.0.194/30` |
| SW2 | VLAN 10 | Default gateway | `10.0.0.62/26` |
| SW2 | VLAN 20 | Default gateway | `10.0.0.126/26` |
| SW2 | VLAN 30 | Default gateway | `10.0.0.190/26` |
| R1 | G0/0/0 | Internet-facing | `1.1.1.2/24` |

---

# 📋 Final SW2 Configuration Highlights

The important parts of the final SW2 configuration are:

```cisco
ip routing

interface GigabitEthernet1/0/1
 switchport trunk allowed vlan 10,30
 switchport mode trunk

interface GigabitEthernet1/0/2
 no switchport
 ip address 10.0.0.193 255.255.255.252

interface Vlan10
 ip address 10.0.0.62 255.255.255.192

interface Vlan20
 ip address 10.0.0.126 255.255.255.192

interface Vlan30
 ip address 10.0.0.190 255.255.255.192

ip route 0.0.0.0 0.0.0.0 10.0.0.194
```

---

# 📋 Final R1 Configuration Highlights

R1 no longer uses ROAS for VLAN routing.

The relevant configuration is:

```cisco
interface GigabitEthernet0/0
 ip address 10.0.0.194 255.255.255.252

interface GigabitEthernet0/0/0
 ip address 1.1.1.2 255.255.255.0

ip route 0.0.0.0 0.0.0.0 GigabitEthernet0/0
```

The VLAN subinterfaces have been removed.

---

# 🧠 Key Networking Concepts

## Router-on-a-Stick vs Multilayer Switching

### Router-on-a-Stick

ROAS uses one router interface with multiple subinterfaces:

```text
R1
 |
 +-- G0/0.10 → VLAN 10
 +-- G0/0.20 → VLAN 20
 +-- G0/0.30 → VLAN 30
 |
SW2
```

The router performs the inter-VLAN routing.

### Multilayer Switching

With a multilayer switch:

```text
SW2
 |
 +-- VLAN10 SVI
 +-- VLAN20 SVI
 +-- VLAN30 SVI
 |
Layer 3 Routing
 |
R1
```

The switch performs the inter-VLAN routing locally.

This generally provides a more scalable design because traffic between local VLANs does not need to travel to an external router.

---

# 🧠 What Is an SVI?

An **SVI (Switch Virtual Interface)** is a logical Layer 3 interface associated with a VLAN.

For example:

```text
interface Vlan10
 ip address 10.0.0.62 255.255.255.192
```

The SVI becomes the default gateway for devices in VLAN 10.

Therefore:

```text
VLAN 10 PCs
      |
      ↓
10.0.0.62
      |
    SW2
      |
   Routing
```

---

# 🧠 What Is a Routed Port?

A routed port is a physical switch interface operating at Layer 3 rather than Layer 2.

It is created using:

```cisco
no switchport
```

In this lab:

```text
SW2 G1/0/2
      |
10.0.0.193/30
      |
10.0.0.194/30
      |
R1 G0/0
```

This creates a simple point-to-point Layer 3 connection.

---

# 🧠 Default Route

The default route:

```cisco
ip route 0.0.0.0 0.0.0.0 10.0.0.194
```

can be interpreted as:

> Send any packet for which there is no more specific route to R1 at `10.0.0.194`.

This allows hosts in VLANs 10, 20, and 30 to reach destinations outside their local networks.

---

# 💾 Save Configuration

After completing the configuration:

```cisco
write memory
```

or:

```cisco
do write
```

Expected result:

```text
Building configuration...

[OK]
```

---

# ✅ Final Lab Checklist

- [x] Existing ROAS configuration identified
- [x] R1 ROAS subinterfaces removed
- [x] R1 G0/0 configured as a Layer 3 interface
- [x] R1 G0/0 assigned `10.0.0.194/30`
- [x] SW2 G1/0/2 converted to a routed port
- [x] SW2 G1/0/2 assigned `10.0.0.193/30`
- [x] IP routing enabled on SW2
- [x] SW2 SVI for VLAN 10 configured
- [x] SW2 SVI for VLAN 20 configured
- [x] SW2 SVI for VLAN 30 configured
- [x] Last usable IP assigned to each SVI
- [x] Default route configured on SW2
- [x] R1 configured as the next hop
- [x] SW1-SW2 trunk retained
- [x] Inter-VLAN connectivity tested
- [x] Internet connectivity tested
- [x] `1.1.1.1` successfully reachable
- [x] Configuration saved

---

# 🏁 Conclusion

This lab demonstrated the transition from a **Router-on-a-Stick architecture** to **multilayer switching**.

The major architectural change was moving the inter-VLAN routing function from R1 to SW2.

### Before

```text
VLANs
  |
 SW2
  |
  | Trunk
  |
 R1
  |
 ROAS
  |
Inter-VLAN Routing
```

### After

```text
VLANs
  |
 SW2
  |
 SVIs
  |
Inter-VLAN Routing
  |
Layer 3 P2P Link
  |
 R1
  |
Internet
```

The final configuration successfully achieved both required goals:

1. **Inter-VLAN connectivity** through the SVIs on the multilayer switch.
2. **Internet connectivity** through the Layer 3 point-to-point connection and default route toward R1.

The successful ping to `1.1.1.1` with **0% packet loss** confirmed that the complete path from the VLAN hosts through SW2, R1, and the Internet router was operational.