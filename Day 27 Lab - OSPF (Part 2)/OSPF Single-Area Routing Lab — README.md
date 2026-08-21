# OSPF Single-Area Routing Lab

## Lab Overview

In this lab, a four-router topology is configured using **OSPF (Open Shortest Path First)** as the dynamic routing protocol.

The lab focuses on:

- Basic router addressing and interface configuration
- Loopback interfaces
- OSPF configuration directly on router interfaces
- OSPF passive interfaces
- OSPF reference bandwidth
- Configuring an ASBR
- Injecting a default route into an OSPF domain
- Verifying OSPF-learned routes
- Observing OSPF Hello packets in Packet Tracer Simulation Mode
- Observing how OSPF reconverges when a link fails

> **Note:** ISPR1 does not need to be configured.

---

## Topology

The topology consists of:

```text
                    G0/0
              10.0.12.0/30
        R1 ---------------- R2
        |                    |
        |                    |
   F1/0 |                    | F1/0
        |                    |
        |                    |
        R3 ---------------- R4
              10.0.34.0/30

R1 --- R3: 10.0.13.0/30
R2 --- R4: 10.0.24.0/30
R3 --- R4: 10.0.34.0/30
```

R4 also has a LAN:

```text
R4 G0/0 → 192.168.4.0/24
```

---

## IP Addressing

| Router | Interface | IP Address | Purpose |
|---|---|---|---|
| R1 | Loopback0 | 1.1.1.1/32 | Router ID/Loopback |
| R1 | G0/0 | 10.0.12.1/30 | R1-R2 link |
| R1 | F1/0 | 10.0.13.1/30 | R1-R3 link |
| R2 | Loopback0 | 2.2.2.2/32 | Router ID/Loopback |
| R2 | G0/0 | 10.0.12.2/30 | R1-R2 link |
| R2 | F1/0 | 10.0.24.1/30 | R2-R4 link |
| R3 | Loopback0 | 3.3.3.3/32 | Router ID/Loopback |
| R3 | F1/0 | 10.0.13.2/30 | R1-R3 link |
| R3 | F2/0 | 10.0.34.1/30 | R3-R4 link |
| R4 | Loopback0 | 4.4.4.4/32 | Router ID/Loopback |
| R4 | F1/0 | 10.0.24.2/30 | R2-R4 link |
| R4 | F2/0 | 10.0.34.2/30 | R3-R4 link |
| R4 | G0/0 | 192.168.4.254/24 | R4 LAN |

---

# Lab Tasks

## 1. Configure Hostnames and IP Addresses

Configure the appropriate hostname and IP addressing on each router.

Bring all required router interfaces up using:

```cisco
no shutdown
```

ISPR1 does not need to be configured.

Example:

```cisco
hostname R1

interface GigabitEthernet0/0
 ip address 10.0.12.1 255.255.255.252
 no shutdown
```

---

## 2. Configure Loopback Interfaces

Each router should have a `/32` loopback address corresponding to its router number.

| Router | Loopback |
|---|---|
| R1 | 1.1.1.1/32 |
| R2 | 2.2.2.2/32 |
| R3 | 3.3.3.3/32 |
| R4 | 4.4.4.4/32 |

Example:

```cisco
interface Loopback0
 ip address 1.1.1.1 255.255.255.255
```

---

## 3. Configure OSPF

Enable **OSPF process 1** directly on each participating interface.

All interfaces participating in the routing domain should use:

```cisco
ip ospf 1 area 0
```

Example:

```cisco
interface Loopback0
 ip ospf 1 area 0

interface GigabitEthernet0/0
 ip ospf 1 area 0

interface FastEthernet1/0
 ip ospf 1 area 0
```

Configure loopback interfaces as passive where appropriate.

Example:

```cisco
router ospf 1
 passive-interface Loopback0
```

The loopbacks should be advertised into OSPF, but they do not need to form OSPF neighbor adjacencies.

Verify OSPF neighbors with:

```cisco
show ip ospf neighbor
```

A successful adjacency should reach the:

```text
FULL
```

state.

---

## 4. Configure OSPF Reference Bandwidth

Configure the OSPF reference bandwidth so that a **FastEthernet interface has an OSPF cost of 100**.

Use:

```cisco
router ospf 1
 auto-cost reference-bandwidth 10000
```

The reference bandwidth must be configured consistently on **all OSPF routers**.

The lab produced the following warning when the value was configured:

```text
% OSPF: Reference bandwidth is changed.

Please ensure reference bandwidth is consistent across all routers.
```

This is an important reminder that mismatched reference bandwidths can result in inconsistent OSPF cost calculations.

Verify with:

```cisco
show ip ospf
```

---

## 5. Configure R1 as an ASBR

R1 acts as the **Autonomous System Boundary Router (ASBR)** because it has a route leading outside the OSPF domain.

Configure a static default route on R1 toward the ISP:

```cisco
ip route 0.0.0.0 0.0.0.0 203.0.113.2
```

Then configure OSPF to advertise the default route:

```cisco
router ospf 1
 default-information originate
```

R1 should therefore advertise:

```text
0.0.0.0/0
```

into the OSPF domain.

---

# Verification

## Verify R1's Routing Table

Use:

```cisco
show ip route
```

R1 should have a static default route pointing toward the ISP.

Example configuration:

```text
router ospf 1
 default-information originate

ip route 0.0.0.0 0.0.0.0 203.0.113.2
```

---

## 6. Check R4's Routing Table

Run:

```cisco
show ip route
```

Initially, R4's routing table in the captured output showed:

```text
Gateway of last resort is not set
```

and there was **no OSPF default route**.

The routing table contained OSPF routes to the loopbacks and inter-router networks, for example:

```text
O       1.1.1.1 [110/111] via 10.0.24.1
O       2.2.2.2 [110/101] via 10.0.24.1
O       3.3.3.3 [110/101] via 10.0.34.1
```

The important observation is that R4 initially did not have a default route.

### Why?

R1 was configured with:

```cisco
default-information originate
```

but the static default route was added afterward. Once R1 had:

```cisco
ip route 0.0.0.0 0.0.0.0 203.0.113.2
```

it was able to originate the default route into OSPF.

After convergence, check R4 again:

```cisco
show ip route
```

Look specifically for:

```text
O*E2 0.0.0.0/0
```

The exact next hop depends on the topology and OSPF path calculation.

---

# OSPF Cost and Path Selection

The reference bandwidth was configured as:

```cisco
auto-cost reference-bandwidth 10000
```

OSPF cost is calculated approximately as:

```text
Cost = Reference Bandwidth / Interface Bandwidth
```

With a reference bandwidth of **10,000 Mbps**:

### FastEthernet

FastEthernet = 100 Mbps

```text
10,000 / 100 = 100
```

Therefore:

```text
FastEthernet OSPF cost = 100
```

### GigabitEthernet

GigabitEthernet = 1,000 Mbps

```text
10,000 / 1,000 = 10
```

Therefore:

```text
GigabitEthernet OSPF cost = 10
```

This explains why the routing table shows different OSPF metrics depending on the path.

---

# Link Failure Test

One of the useful observations from the lab was what happened when R4's connection to R2 was shut down.

On R4:

```cisco
interface FastEthernet1/0
 shutdown
```

OSPF immediately detected the neighbor failure:

```text
%OSPF-5-ADJCHG: Process 1, Nbr 2.2.2.2 on FastEthernet1/0
from FULL to DOWN
```

R4 then recalculated its routes and used R3 as the next hop.

For example, R4's route to R1's loopback changed from:

```text
O 1.1.1.1 [110/111] via 10.0.24.1
```

to:

```text
O 1.1.1.1 [110/201] via 10.0.34.1
```

Similarly, the route toward R2 changed to:

```text
O 2.2.2.2 [110/211] via 10.0.34.1
```

This demonstrates **OSPF dynamic convergence** and its ability to select an alternate path after a link failure.

---

# 7. Observe OSPF Hello Messages

Switch Packet Tracer into **Simulation Mode**.

Filter the traffic so that OSPF packets can be observed.

OSPF routers periodically exchange **Hello packets** to discover and maintain neighbor relationships.

The Hello packet contains information such as:

- OSPF version
- Router ID
- Area ID
- Network mask
- Hello interval
- Dead interval
- Router priority
- Designated Router (DR)
- Backup Designated Router (BDR)
- Neighbor list
- Authentication-related information where applicable

The **Neighbor list** is particularly important because it allows a router to identify which OSPF neighbors it has discovered on the interface.

---

# Useful Verification Commands

### Check interface status

```cisco
show ip interface brief
```

### Check OSPF neighbors

```cisco
show ip ospf neighbor
```

### Check OSPF configuration

```cisco
show ip ospf
```

### Check OSPF interfaces

```cisco
show ip ospf interface
```

### Check routing table

```cisco
show ip route
```

### Display OSPF routes only

```cisco
show ip route ospf
```

### Check running configuration

```cisco
show running-config
```

---

# Example R1 Configuration

```cisco
hostname R1

interface Loopback0
 ip address 1.1.1.1 255.255.255.255
 ip ospf 1 area 0

interface GigabitEthernet0/0
 ip address 10.0.12.1 255.255.255.252
 ip ospf 1 area 0
 no shutdown

interface FastEthernet1/0
 ip address 10.0.13.1 255.255.255.252
 ip ospf 1 area 0
 no shutdown

router ospf 1
 log-adjacency-changes
 passive-interface Loopback0
 auto-cost reference-bandwidth 10000
 default-information originate

ip route 0.0.0.0 0.0.0.0 203.0.113.2
```

---

# Example R2 Configuration

```cisco
hostname R2

interface Loopback0
 ip address 2.2.2.2 255.255.255.255
 ip ospf 1 area 0

interface GigabitEthernet0/0
 ip address 10.0.12.2 255.255.255.252
 ip ospf 1 area 0
 no shutdown

interface FastEthernet1/0
 ip address 10.0.24.1 255.255.255.252
 ip ospf 1 area 0
 no shutdown

router ospf 1
 log-adjacency-changes
 passive-interface Loopback0
 auto-cost reference-bandwidth 10000
```

---

# Example R3 Configuration

```cisco
hostname R3

interface Loopback0
 ip address 3.3.3.3 255.255.255.255
 ip ospf 1 area 0

interface FastEthernet1/0
 ip address 10.0.13.2 255.255.255.252
 ip ospf 1 area 0
 no shutdown

interface FastEthernet2/0
 ip address 10.0.34.1 255.255.255.252
 ip ospf 1 area 0
 no shutdown

router ospf 1
 log-adjacency-changes
 passive-interface Loopback0
 auto-cost reference-bandwidth 10000
```

---

# Example R4 Configuration

```cisco
hostname R4

interface Loopback0
 ip address 4.4.4.4 255.255.255.255
 ip ospf 1 area 0

interface GigabitEthernet0/0
 ip address 192.168.4.254 255.255.255.0
 ip ospf 1 area 0
 no shutdown

interface FastEthernet1/0
 ip address 10.0.24.2 255.255.255.252
 ip ospf 1 area 0
 no shutdown

interface FastEthernet2/0
 ip address 10.0.34.2 255.255.255.252
 ip ospf 1 area 0
 no shutdown

router ospf 1
 log-adjacency-changes
 passive-interface Loopback0
 auto-cost reference-bandwidth 10000
```

---

# Key Lessons

1. **OSPF can be enabled directly on interfaces** using `ip ospf <process-id> area <area-id>`.
2. **Loopbacks are commonly configured as passive interfaces** because they do not need to form OSPF neighbor relationships.
3. The OSPF reference bandwidth should be **consistent across all routers**.
4. With a reference bandwidth of `10000 Mbps`, a FastEthernet interface has an OSPF cost of **100**.
5. R1 becomes an **ASBR** when it injects the external default route into OSPF.
6. `default-information originate` advertises a default route into the OSPF domain when the router has a default route.
7. OSPF automatically recalculates paths when a link fails.
8. OSPF Hello packets are used to **discover and maintain neighboring routers**.
9. The OSPF routing table uses administrative distance **110** for OSPF routes.
10. OSPF's ability to maintain alternate paths provides **dynamic network convergence and redundancy**.

---

## Lab Completion Checklist

- [ ] Configure R1 hostname and IP addresses
- [ ] Configure R2 hostname and IP addresses
- [ ] Configure R3 hostname and IP addresses
- [ ] Configure R4 hostname and IP addresses
- [ ] Enable all required interfaces
- [ ] Configure Loopback0 on R1-R4
- [ ] Enable OSPF process 1
- [ ] Place all interfaces in Area 0
- [ ] Configure loopbacks as passive interfaces
- [ ] Configure OSPF reference bandwidth to `10000`
- [ ] Verify OSPF neighbor adjacencies
- [ ] Configure R1's static default route
- [ ] Configure R1 as an OSPF ASBR
- [ ] Advertise the default route using `default-information originate`
- [ ] Verify R4's routing table
- [ ] Test OSPF path selection
- [ ] Shut down an R4 link and observe OSPF convergence
- [ ] Use Packet Tracer Simulation Mode to inspect OSPF Hello packets
- [ ] Save the configurations with `write memory` / `do wr`