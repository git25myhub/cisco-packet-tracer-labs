# IPv6 EUI-64 Addressing and Static Routing Lab

## Overview

This lab introduces **IPv6 EUI-64 addressing, IPv6 link-local addressing, and static IPv6 routing** using Cisco Packet Tracer.

The IPv4 configuration of the network is already complete. The objective is to add IPv6 connectivity without replacing the existing IPv4 configuration.

In this lab, IPv6 addresses are automatically generated on the LAN-facing interfaces of R1 and R2 using **EUI-64**. The router-to-router interfaces are enabled for IPv6 without assigning explicit global IPv6 addresses. Static IPv6 routes are then configured so that PC1 and PC2 can communicate across the two routers.

> **Note:** IPv6 static routing will be studied in greater depth in Day 33. This lab provides an introduction to the `ipv6 route` command and the use of link-local next-hop addresses.

---

# Lab Objectives

By completing this lab, you will:

- Enable IPv6 unicast routing on R1 and R2.
- Calculate an IPv6 EUI-64 interface identifier.
- Configure IPv6 addresses using EUI-64.
- Enable IPv6 on router-to-router interfaces without assigning a global IPv6 address.
- Understand IPv6 link-local addresses.
- Configure IPv6 addresses and default gateways on PCs.
- Configure IPv6 static routes.
- Use `?` to explore Cisco IOS IPv6 routing commands.
- Verify IPv6 connectivity between PCs.
- Troubleshoot IPv6 routing and addressing.

---

# Existing IPv4 Network

The IPv4 configuration is already complete.

The network uses the following IPv4 addressing:

| Device | Interface | IPv4 Address | Network |
|---|---|---|---|
| R1 | G0/0 | `192.168.1.1/30` | `192.168.1.0/30` |
| R1 | G0/1 | `10.0.1.254/24` | `10.0.1.0/24` |
| R2 | G0/0 | `192.168.1.2/30` | `192.168.1.0/30` |
| R2 | G0/1 | `10.0.2.254/24` | `10.0.2.0/24` |

The IPv4 routing configuration is already present and should remain unchanged.

---

# IPv6 Addressing Plan

The IPv6 LAN networks are:

| Network | Purpose |
|---|---|
| `2001:DB8::/64` | R1 LAN / PC1 |
| `2001:DB8:0:1::/64` | R2 LAN / PC2 |
| Link-local addresses | R1-R2 router-to-router link |

The router LAN interfaces will use EUI-64 to generate their interface identifiers.

---

# Task 1 — Calculate the EUI-64 Interface IDs

Before configuring EUI-64, calculate the interface identifier that Cisco IOS will generate.

EUI-64 is created from the interface's **48-bit MAC address**.

The basic process is:

1. Take the 48-bit MAC address.
2. Split it into two 24-bit halves.
3. Insert `FFFE` in the middle.
4. Flip the **Universal/Local (U/L) bit** of the first byte.

---

## R1 G0/1

The interface MAC address is:

```text
0030.F236.4502
```

Split the MAC:

```text
0030.F2    36.45.02
```

Insert `FFFE`:

```text
0030.F2FF.FE36.4502
```

Flip the U/L bit of the first byte:

```text
00 → 02
```

Therefore, the EUI-64 interface identifier is:

```text
0230:F2FF:FE36:4502
```

Cisco displays this in compressed IPv6 notation as:

```text
230:F2FF:FE36:4502
```

Therefore, R1's IPv6 address becomes:

```text
2001:DB8::230:F2FF:FE36:4502/64
```

---

## R2 G0/1

The relevant MAC address is:

```text
0030.F2B0.B802
```

Split the MAC:

```text
0030.F2    B0.B8.02
```

Insert `FFFE`:

```text
0030.F2FF.FEB0.B802
```

Flip the U/L bit:

```text
00 → 02
```

The resulting EUI-64 interface identifier is:

```text
0230:F2FF:FEB0:B802
```

Cisco displays the address in compressed notation as:

```text
230:F2FF:FEB0:B802
```

Therefore, R2's IPv6 address becomes:

```text
2001:DB8:0:1:230:F2FF:FEB0:B802/64
```

---

# EUI-64 Calculation Summary

| Router | Interface | MAC Address | EUI-64 Interface ID | IPv6 Address |
|---|---|---|---|---|
| R1 | G0/1 | `0030.F236.4502` | `0230:F2FF:FE36:4502` | `2001:DB8::230:F2FF:FE36:4502/64` |
| R2 | G0/1 | `0030.F2B0.B802` | `0230:F2FF:FEB0:B802` | `2001:DB8:0:1:230:F2FF:FEB0:B802/64` |

> **Important:** Always obtain the actual MAC address from the device before calculating EUI-64. Do not assume that two interfaces will generate the same interface ID.

---

# Task 2 — Enable IPv6 Routing on R1

Enter global configuration mode:

```cisco
R1> enable
R1# configure terminal
R1(config)# ipv6 unicast-routing
```

This enables R1 to forward IPv6 packets between IPv6 networks.

---

# Task 3 — Configure EUI-64 on R1 G0/1

Configure the IPv6 network prefix and tell IOS to generate the interface ID automatically:

```cisco
R1(config)# interface gigabitEthernet0/1
R1(config-if)# ipv6 address 2001:DB8::/64 eui-64
```

Verify the result:

```cisco
R1# show ipv6 interface brief
```

Expected result:

```text
GigabitEthernet0/1    [up/up]
    FE80::230:F2FF:FE36:4502
    2001:DB8::230:F2FF:FE36:4502
```

The global IPv6 address is generated from:

```text
Prefix:       2001:DB8::/64
EUI-64 ID:    230:F2FF:FE36:4502
```

---

# Task 4 — Enable IPv6 on R1 G0/0

The R1-R2 link does not require an explicit global IPv6 address for this lab.

Instead, enable IPv6 on the interface:

```cisco
R1(config)# interface gigabitEthernet0/0
R1(config-if)# ipv6 enable
```

Verify:

```cisco
R1# show ipv6 interface brief
```

R1 G0/0 should now have a link-local address similar to:

```text
GigabitEthernet0/0    [up/up]
    FE80::230:F2FF:FE36:4501
```

### Why is there no global address?

The command:

```cisco
ipv6 enable
```

enables IPv6 processing and creates a link-local address, but it does not assign a global unicast address.

This is sufficient for using the interface as part of a static route with a link-local next hop.

---

# Task 5 — Configure R2

Enable IPv6 routing:

```cisco
R2> enable
R2# configure terminal
R2(config)# ipv6 unicast-routing
```

Configure EUI-64 on G0/1:

```cisco
R2(config)# interface gigabitEthernet0/1
R2(config-if)# ipv6 address 2001:DB8:0:1::/64 eui-64
```

Enable IPv6 on the router-to-router interface:

```cisco
R2(config)# interface gigabitEthernet0/0
R2(config-if)# ipv6 enable
```

Verify:

```cisco
R2# show ipv6 interface brief
```

Expected result:

```text
GigabitEthernet0/0    [up/up]
    FE80::230:F2FF:FEB0:B801

GigabitEthernet0/1    [up/up]
    FE80::230:F2FF:FEB0:B802
    2001:DB8:0:1:230:F2FF:FEB0:B802
```

---

# Task 6 — Configure IPv6 on PC1 and PC2

The PCs require a global IPv6 address and an appropriate IPv6 default gateway.

## PC1

PC1 belongs to:

```text
2001:DB8::/64
```

Configure:

| Setting | Value |
|---|---|
| IPv6 Address | `2001:DB8::2` |
| Prefix Length | `64` |
| IPv6 Default Gateway | `2001:DB8::230:F2FF:FE36:4502` |

PC1 should continue using its existing IPv4 configuration.

---

## PC2

PC2 belongs to:

```text
2001:DB8:0:1::/64
```

Configure:

| Setting | Value |
|---|---|
| IPv6 Address | `2001:DB8:0:1::2` |
| Prefix Length | `64` |
| IPv6 Default Gateway | `2001:DB8:0:1:230:F2FF:FEB0:B802` |

Verify the configuration from the PC command prompt:

```text
C:\> ipconfig
```

---

# Task 7 — Explore the `ipv6 route` Command

Before configuring static routes, use Cisco IOS context-sensitive help:

```cisco
R1(config)# ipv6 route ?
```

Cisco IOS shows:

```text
X:X:X:X::X/<0-128>    IPv6 prefix
```

Specify the destination network:

```cisco
R1(config)# ipv6 route 2001:DB8:0:1::/64 ?
```

IOS displays the available next-hop options, including:

```text
GigabitEthernet
X:X:X:X              IPv6 address of next-hop
```

This demonstrates that an IPv6 static route can use a next-hop IPv6 address and, when using a link-local address, the outgoing interface must also be specified.

---

# Task 8 — Configure an IPv6 Static Route on R1

R1 needs a route to the IPv6 network behind R2:

```text
2001:DB8:0:1::/64
```

R2's link-local address on G0/0 is:

```text
FE80::201:63FF:FEB0:B801
```

Because this is a **link-local next-hop address**, specify both the outgoing interface and next-hop address:

```cisco
R1(config)# ipv6 route 2001:DB8:0:1::/64 gigabitEthernet0/0 FE80::201:63FF:FEB0:B801
```

### Important

If you try:

```cisco
R1(config)# ipv6 route 2001:DB8:0:1::/64 FE80::201:63FF:FEB0:B801
```

IOS returns:

```text
% Interface has to be specified for a link-local nexthop
```

This is expected because link-local IPv6 addresses are only meaningful on the local link. The router therefore needs to know which interface to use.

---

# Task 9 — Configure an IPv6 Static Route on R2

R2 needs a route back to R1's IPv6 LAN:

```text
2001:DB8::/64
```

R1's link-local address on G0/0 is:

```text
FE80::230:F2FF:FE36:4501
```

Configure:

```cisco
R2(config)# ipv6 route 2001:DB8::/64 gigabitEthernet0/0 FE80::230:F2FF:FE36:4501
```

Save both router configurations:

```cisco
R1# write memory
R2# write memory
```

---

# Static Route Summary

| Router | Destination Network | Outgoing Interface | Next Hop |
|---|---|---|---|
| R1 | `2001:DB8:0:1::/64` | G0/0 | `FE80::201:63FF:FEB0:B801` |
| R2 | `2001:DB8::/64` | G0/0 | `FE80::230:F2FF:FE36:4501` |

---

# Task 10 — Verify IPv6 Static Routes

On R1:

```cisco
R1# show ipv6 route
```

Look for the static route:

```text
S   2001:DB8:0:1::/64
```

On R2:

```cisco
R2# show ipv6 route
```

Look for:

```text
S   2001:DB8::/64
```

You can also inspect the running configuration:

```cisco
R1# show running-config
R2# show running-config
```

The static routes should appear near the bottom of the configuration.

---

# Task 11 — Test PC Connectivity

From PC1, first test its local R1 gateway:

```text
C:\> ping 2001:DB8::230:F2FF:FE36:4502
```

Then test PC2:

```text
C:\> ping 2001:DB8:0:1::2
```

A successful result should resemble:

```text
Reply from 2001:DB8:0:1::2: bytes=32 time<1ms TTL=126
Reply from 2001:DB8:0:1::2: bytes=32 time<1ms TTL=126
Reply from 2001:DB8:0:1::2: bytes=32 time<1ms TTL=126
Reply from 2001:DB8:0:1::2: bytes=32 time<1ms TTL=126

Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

The successful ping confirms that:

1. PC1 has a valid IPv6 address.
2. PC1 has the correct default gateway.
3. R1 has an IPv6 route to PC2's network.
4. R1 can forward the packet to R2.
5. R2 has an IPv6 route back to PC1's network.
6. PC2 can return the IPv6 traffic.

---

# Packet Path

When PC1 communicates with PC2, the traffic follows this path:

```text
PC1
 │
 │ 2001:DB8::/64
 ▼
R1 G0/1
 │
 │ R1 G0/0
 │ FE80::230:F2FF:FE36:4501
 │
 │ R2 G0/0
 │ FE80::201:63FF:FEB0:B801
 ▼
R2 G0/1
 │
 │ 2001:DB8:0:1::/64
 ▼
PC2
```

The static routes tell each router how to reach the remote LAN.

---

# Verification Commands

## R1

```cisco
show ipv6 interface brief
show ipv6 interface gigabitEthernet0/0
show ipv6 interface gigabitEthernet0/1
show ipv6 route
show running-config
```

## R2

```cisco
show ipv6 interface brief
show ipv6 interface gigabitEthernet0/0
show ipv6 interface gigabitEthernet0/1
show ipv6 route
show running-config
```

## PCs

```text
ipconfig
ping <IPv6-address>
tracert <IPv6-address>
```

---

# Troubleshooting

## Problem 1 — EUI-64 Address Does Not Match

Check the interface MAC address:

```cisco
show interface gigabitEthernet0/1
```

Look for:

```text
Hardware is CN Gigabit Ethernet, address is ...
```

Recalculate the EUI-64 interface ID from the actual MAC address.

---

## Problem 2 — PC Cannot Ping Its Gateway

Check:

- IPv6 address
- `/64` prefix length
- IPv6 default gateway
- Router interface status
- Correct IPv6 network

On the router:

```cisco
show ipv6 interface brief
```

The interface should be:

```text
[up/up]
```

---

## Problem 3 — R1 Cannot Reach PC2

Check R1's IPv6 routing table:

```cisco
R1# show ipv6 route
```

Confirm that the route to:

```text
2001:DB8:0:1::/64
```

exists.

---

## Problem 4 — Static Route Using Link-Local Address Fails

If you configure:

```cisco
ipv6 route 2001:DB8:0:1::/64 FE80::201:63FF:FEB0:B801
```

IOS reports:

```text
% Interface has to be specified for a link-local nexthop
```

Correct the route by specifying the outgoing interface:

```cisco
ipv6 route 2001:DB8:0:1::/64 gigabitEthernet0/0 FE80::201:63FF:FEB0:B801
```

---

## Problem 5 — PC2 Receives the Packet but Cannot Reply

IPv6 routing must work in **both directions**.

R2 therefore needs a return route:

```cisco
R2(config)# ipv6 route 2001:DB8::/64 gigabitEthernet0/0 FE80::230:F2FF:FE36:4501
```

Without this route, PC1 may be able to send traffic toward PC2, but PC2 will not know how to return traffic to PC1.

---

# Final Configuration Summary

## R1

```cisco
ipv6 unicast-routing

interface GigabitEthernet0/0
 ip address 192.168.1.1 255.255.255.252
 ipv6 enable

interface GigabitEthernet0/1
 ip address 10.0.1.254 255.255.255.0
 ipv6 address 2001:DB8::/64 eui-64

ipv6 route 2001:DB8:0:1::/64 GigabitEthernet0/0 FE80::201:63FF:FEB0:B801
```

## R2

```cisco
ipv6 unicast-routing

interface GigabitEthernet0/0
 ip address 192.168.1.2 255.255.255.252
 ipv6 enable

interface GigabitEthernet0/1
 ip address 10.0.2.254 255.255.255.0
 ipv6 address 2001:DB8:0:1::/64 eui-64

ipv6 route 2001:DB8::/64 GigabitEthernet0/0 FE80::230:F2FF:FE36:4501
```

---

# Verification Checklist

- [ ] IPv4 configuration remains unchanged.
- [ ] IPv6 unicast routing is enabled on R1.
- [ ] IPv6 unicast routing is enabled on R2.
- [ ] R1 G0/1 uses EUI-64 addressing.
- [ ] R2 G0/1 uses EUI-64 addressing.
- [ ] R1 EUI-64 interface ID was calculated from its MAC address.
- [ ] R2 EUI-64 interface ID was calculated from its MAC address.
- [ ] IPv6 is enabled on R1 G0/0.
- [ ] IPv6 is enabled on R2 G0/0.
- [ ] R1 G0/0 has a link-local IPv6 address.
- [ ] R2 G0/0 has a link-local IPv6 address.
- [ ] PC1 has the correct IPv6 address and default gateway.
- [ ] PC2 has the correct IPv6 address and default gateway.
- [ ] R1 has a static route to PC2's IPv6 network.
- [ ] R2 has a static route to PC1's IPv6 network.
- [ ] PC1 can ping PC2 using IPv6.
- [ ] IPv4 connectivity remains operational.

---

# Lab Results

The lab successfully demonstrated IPv6 EUI-64 addressing and static routing.

R1 G0/1 generated:

```text
2001:DB8::230:F2FF:FE36:4502
```

using the MAC address:

```text
0030.F236.4502
```

R2 G0/1 generated:

```text
2001:DB8:0:1:230:F2FF:FEB0:B802
```

using its interface MAC address.

IPv6 was enabled on the router-to-router G0/0 interfaces without explicitly assigning global IPv6 addresses. These interfaces automatically generated link-local addresses, which were then used as the next-hop addresses for the static IPv6 routes.

The final connectivity test was successful:

```text
Pinging 2001:DB8:0:1::2

Reply from 2001:DB8:0:1::2
Reply from 2001:DB8:0:1::2
Reply from 2001:DB8:0:1::2
Reply from 2001:DB8:0:1::2

Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

This confirms end-to-end IPv6 connectivity between PC1 and PC2.

---

# Key Takeaways

### 1. EUI-64 automatically generates the interface ID

Instead of manually configuring the complete IPv6 address:

```cisco
ipv6 address 2001:DB8::230:F2FF:FE36:4502/64
```

EUI-64 allows IOS to generate the interface identifier:

```cisco
ipv6 address 2001:DB8::/64 eui-64
```

### 2. `ipv6 enable` creates a link-local address

The command:

```cisco
ipv6 enable
```

enables IPv6 on an interface and automatically generates a link-local address.

### 3. Link-local addresses can be used as next hops

IPv6 static routes can use link-local addresses as next hops, but the outgoing interface must also be specified.

### 4. Static routing must work in both directions

For PC1 to successfully ping PC2, R1 needs a route to PC2's network and R2 needs a return route to PC1's network.

### 5. IPv6 can operate alongside IPv4

The existing IPv4 configuration remains operational while IPv6 is added, demonstrating another example of an **IPv4/IPv6 dual-stack network**.