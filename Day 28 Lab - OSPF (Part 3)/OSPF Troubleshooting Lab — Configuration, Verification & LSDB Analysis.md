# OSPF Troubleshooting Lab — Configuration, Verification & LSDB Analysis

## Overview

This lab focuses on **OSPF configuration and troubleshooting** in a multi-router network. The network has been partially pre-configured with IP addressing and OSPF, but several faults have been intentionally introduced.

The objective is to identify and correct OSPF, routing, and default-route problems while verifying the resulting network using Cisco IOS show commands.

---

## Lab Objectives

By completing this lab, you will practice:

- Configuring a new **serial point-to-point connection**.
- Establishing OSPF adjacency between routers.
- Troubleshooting missing OSPF routes.
- Troubleshooting failed OSPF neighbor relationships.
- Configuring and verifying an OSPF default route.
- Troubleshooting end-to-end connectivity to an external network.
- Examining the OSPF **Link-State Database (LSDB)**.
- Identifying different OSPF LSA types.

---

## Starting Conditions

The network has been pre-configured with:

- IP addressing.
- OSPF process configuration.
- OSPF Area 0.
- Existing OSPF adjacencies.
- Internal networks.
- An external network connected through R5.

The following network prefixes are present in the topology:

| Network | Purpose |
|---|---|
| `10.0.1.0/24` | LAN connected to R1 |
| `10.0.2.0/24` | LAN connected to R3 |
| `192.168.12.0/30` | R1–R2 serial link |
| `192.168.34.0/30` | R3–R4 link |
| `192.168.245.0/29` | Shared R2/R4/R5 segment |
| `203.0.113.0/30` | External network between R5 and the external router/server |

OSPF is running as **process ID 1** in **Area 0**.

---

# Tasks

## 1. Configure the New R1–R2 Serial Connection

A new connection has been added between **R1 and R2**.

Configure the serial link using the addressing provided in the topology.

### Requirements

- R1 serial interface:
  - IP address: `192.168.12.1/30`
  - OSPF Area: `0`
  - Clock rate: `128000`

- R2 serial interface:
  - IP address: `192.168.12.2/30`
  - OSPF Area: `0`

### Example Configuration

**R1**

```cisco
R1(config)# interface serial0/0/0
R1(config-if)# ip address 192.168.12.1 255.255.255.252
R1(config-if)# clock rate 128000
R1(config-if)# ip ospf 1 area 0
R1(config-if)# no shutdown
```

**R2**

```cisco
R2(config)# interface serial0/0/0
R2(config-if)# ip address 192.168.12.2 255.255.255.252
R2(config-if)# ip ospf 1 area 0
R2(config-if)# no shutdown
```

### Verification

Check the interface:

```cisco
show ip interface brief
```

Verify the OSPF neighbor relationship:

```cisco
show ip ospf neighbor
```

The R1–R2 adjacency should reach:

```text
FULL
```

Verify the route learned across the new link:

```cisco
show ip route ospf
```

---

# 2. R3 Is the Only Router With a Route to 10.0.2.0/24

Initially, only R3 has a route to the `10.0.2.0/24` network.

### Troubleshooting Question

**Why does only R3 have a route to `10.0.2.0/24`?**

The network is connected directly to R3 through:

```text
R3 G0/0 → 10.0.2.0/24
```

For the other routers to learn this network through OSPF, the `10.0.2.0/24` interface must be advertised into OSPF.

R3's configuration should include the LAN network in OSPF:

```cisco
router ospf 1
 network 10.0.2.254 0.0.0.0 area 0
```

or an equivalent network statement.

The interface can remain passive if hosts are connected to the interface and no OSPF neighbors are required there:

```cisco
router ospf 1
 passive-interface GigabitEthernet0/0
```

### Verification

On R3:

```cisco
show ip ospf interface g0/0
```

On another router:

```cisco
show ip route
```

You should eventually see:

```text
O    10.0.2.0/24
```

For example, R4 learns the network through R3:

```text
O       10.0.2.0/24 [110/2] via 192.168.34.1
```

---

# 3. R2 and R4 Are Not Becoming OSPF Neighbors With R5

R2 and R4 share the `192.168.245.0/29` broadcast network with R5.

However, R5 initially uses different OSPF hello/dead timers:

```text
Hello: 5 seconds
Dead: 20 seconds
```

while R2/R4 use:

```text
Hello: 10 seconds
Dead: 40 seconds
```

OSPF neighbors on the same network must agree on important parameters, including the hello and dead intervals.

### Problem

R5 has:

```text
Hello 5
Dead 20
```

while R2 and R4 have:

```text
Hello 10
Dead 40
```

This prevents the routers from forming OSPF adjacencies.

### Fix

Restore R5's OSPF timers to the default values:

```cisco
R5(config)# interface gigabitEthernet0/0
R5(config-if)# no ip ospf hello-interval
R5(config-if)# no ip ospf dead-interval
```

This restores:

```text
Hello = 10 seconds
Dead = 40 seconds
```

### Verification

Run:

```cisco
show ip ospf neighbor
```

On R5, the expected neighbors are:

```text
192.168.245.2     FULL/DROTHER
192.168.245.1     FULL/BDR
```

The adjacency messages should eventually show:

```text
from LOADING to FULL
```

---

# 4. PC1 and PC2 Cannot Ping 8.8.8.8

The external server is located at:

```text
8.8.8.8
```

The internal routers need a route toward the Internet/external network.

R5 is the edge router and has a connection to:

```text
203.0.113.0/30
```

with R5 using:

```text
203.0.113.1
```

and the external router using:

```text
203.0.113.2
```

### Problem

R5 needs a default route pointing toward the external router:

```text
0.0.0.0/0 → 203.0.113.2
```

Configure:

```cisco
R5(config)# ip route 0.0.0.0 0.0.0.0 203.0.113.2
```

R5 must then advertise the default route through OSPF.

The OSPF configuration includes:

```cisco
router ospf 1
 default-information originate
```

This causes R5 to advertise the default route as an **OSPF external route**.

### Verification on R5

```cisco
show ip route
```

You should see:

```text
Gateway of last resort is 203.0.113.2
S*    0.0.0.0/0 [1/0] via 203.0.113.2
```

### Verification on Internal Routers

On R1, R2, R3, or R4:

```cisco
show ip route
```

The default route should appear as:

```text
O*E2 0.0.0.0/0
```

For example:

```text
O*E2 0.0.0.0/0 [110/1] via 192.168.12.2
```

This means the route is:

- `O` — learned through OSPF.
- `*` — candidate default route.
- `E2` — OSPF external Type 2 route.

### End-to-End Test

From PC1 and PC2:

```text
ping 8.8.8.8
```

The ping should succeed once:

1. Internal OSPF adjacencies are established.
2. R5 has a static default route.
3. R5 originates the default route into OSPF.
4. Internal routers install the OSPF default route.

---

# 5. Examine the OSPF LSDB

Use:

```cisco
show ip ospf database
```

The LSDB contains the OSPF link-state information used by routers to build the shortest-path tree.

The captured LSDB contains the following LSA categories.

## Router LSAs — Type 1

The LSDB contains Router LSAs for:

```text
192.168.12.1
192.168.34.1
192.168.245.1
203.0.113.1
192.168.245.2
```

### Purpose

**Type 1 Router LSAs** are generated by every OSPF router.

They describe the router's:

- OSPF interfaces.
- Connected OSPF links.
- Link costs.
- Neighbor relationships.

---

## Network LSAs — Type 2

The LSDB contains Network LSAs for:

```text
192.168.34.2
192.168.245.3
```

### Purpose

**Type 2 Network LSAs** are generated by the **Designated Router (DR)** on multi-access broadcast networks.

They describe:

- The broadcast network.
- The routers attached to that network.

For example, R5 is the DR on the `192.168.245.0/29` network.

---

## AS External LSA — Type 5

The LSDB contains:

```text
0.0.0.0
```

advertised by:

```text
203.0.113.1
```

This is a **Type 5 AS External LSA**.

### Purpose

Type 5 LSAs are used to advertise routes redistributed into OSPF or external routes originated by an Autonomous System Boundary Router (ASBR).

In this lab, R5 originates the default route:

```text
0.0.0.0/0
```

using:

```cisco
default-information originate
```

The internal routers therefore learn the default route as:

```text
O*E2 0.0.0.0/0
```

---

# LSDB Summary

| LSA Type | Name | Generated By | Purpose |
|---|---|---|---|
| Type 1 | Router LSA | Every OSPF router | Describes router links and OSPF interfaces |
| Type 2 | Network LSA | DR | Describes multi-access networks |
| Type 5 | AS External LSA | ASBR | Advertises external routes into OSPF |

In this lab, the important LSA types observed are:

```text
Type 1 — Router LSAs
Type 2 — Network LSAs
Type 5 — AS External LSAs
```

---

# Useful Verification Commands

Use the following commands throughout the lab.

### Interface Status

```cisco
show ip interface brief
```

### OSPF Configuration

```cisco
show ip protocols
```

### OSPF Neighbors

```cisco
show ip ospf neighbor
```

### OSPF Interface Details

```cisco
show ip ospf interface
```

or:

```cisco
show ip ospf interface gigabitEthernet0/0
```

### OSPF Routes

```cisco
show ip route ospf
```

### Complete Routing Table

```cisco
show ip route
```

### OSPF LSDB

```cisco
show ip ospf database
```

### Save Configuration

```cisco
copy running-config startup-config
```

or:

```cisco
write memory
```

---

# Troubleshooting Workflow

A useful troubleshooting sequence for this lab is:

```text
1. Check physical/interface status
        ↓
2. Check IP addressing
        ↓
3. Check OSPF configuration
        ↓
4. Check OSPF neighbors
        ↓
5. Check OSPF routes
        ↓
6. Check the default route
        ↓
7. Test end-to-end connectivity
        ↓
8. Examine the LSDB
```

When an OSPF neighbor fails to form, check:

```text
- Interface status
- IP addressing
- Area number
- OSPF process
- Hello interval
- Dead interval
- Network type
- Authentication
- Subnet/network compatibility
- Passive-interface configuration
```

---

# Expected Final State

At the end of the lab:

- R1 and R2 form an OSPF adjacency over the serial link.
- `10.0.2.0/24` is reachable through OSPF from the other routers.
- R2 and R4 form OSPF adjacencies with R5.
- R5 has a static default route toward `203.0.113.2`.
- R5 advertises the default route into OSPF.
- Internal routers install an `O*E2` default route.
- PC1 and PC2 can successfully ping `8.8.8.8`.
- The OSPF LSDB contains Type 1, Type 2, and Type 5 LSAs.

---

# Key Lessons

This lab demonstrates several important OSPF troubleshooting concepts:

1. **An interface must participate in OSPF for its network to be advertised.**
2. **OSPF neighbors must agree on critical parameters such as hello and dead intervals.**
3. **A default route must exist on the edge router before `default-information originate` can advertise it.**
4. **OSPF external routes are represented using Type 5 LSAs.**
5. **The DR generates Type 2 LSAs on broadcast multi-access networks.**
6. **The OSPF LSDB provides the topology information used to calculate routes.**
7. **Successful OSPF adjacency does not automatically guarantee end-to-end connectivity; the routing table and default route must also be verified.**

---

## Lab Verification Checklist

- [ ] R1–R2 serial interface configured.
- [ ] Serial clock rate configured as `128000`.
- [ ] R1 and R2 participate in OSPF Area 0.
- [ ] R1 and R2 reach `FULL` OSPF adjacency.
- [ ] `10.0.2.0/24` is advertised by R3.
- [ ] All required routers learn `10.0.2.0/24`.
- [ ] R5 OSPF hello interval is `10` seconds.
- [ ] R5 OSPF dead interval is `40` seconds.
- [ ] R2 and R4 form OSPF neighbors with R5.
- [ ] R5 has a default route via `203.0.113.2`.
- [ ] R5 originates the default route into OSPF.
- [ ] Internal routers learn `0.0.0.0/0` as `O*E2`.
- [ ] PC1 can ping `8.8.8.8`.
- [ ] PC2 can ping `8.8.8.8`.
- [ ] OSPF LSDB has Type 1 LSAs.
- [ ] OSPF LSDB has Type 2 LSAs.
- [ ] OSPF LSDB has a Type 5 LSA for `0.0.0.0/0`.
- [ ] Configurations are saved.

---

## Conclusion

This lab provides practical experience troubleshooting an OSPF network rather than simply configuring OSPF from scratch. The key is to work systematically from **interfaces → neighbors → routes → default routing → end-to-end connectivity → LSDB**.

The final network should have complete internal OSPF reachability and a functional path from the internal PCs through R5 to the external server at `8.8.8.8`.