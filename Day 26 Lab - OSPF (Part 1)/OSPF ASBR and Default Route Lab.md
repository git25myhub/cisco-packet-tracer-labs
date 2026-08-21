# OSPF ASBR and Default Route Lab

## Overview

This lab focuses on configuring **OSPF (Open Shortest Path First)** across a four-router topology and configuring **R1 as an ASBR (Autonomous System Boundary Router)** that injects a default route into the OSPF domain.

The lab covers:

- Basic router configuration
- Hostname and IP address configuration
- Loopback interfaces
- OSPF configuration
- OSPF passive interfaces
- OSPF neighbor relationships
- OSPF area 0
- Configuring an ASBR
- Default route injection
- OSPF routing-table verification
- Understanding how internal OSPF routers learn a default route

> **Platform:** Cisco Packet Tracer  
> **Routing Protocol:** OSPF  
> **OSPF Area:** Area 0  
> **OSPF Process IDs:** 1–4 as used on the individual routers

---

## Lab Objectives

By completing this lab, you should be able to:

1. Configure hostnames and IP addresses on the routers.
2. Enable the required router interfaces.
3. Configure loopback interfaces using `/32` addresses.
4. Configure OSPF on router interfaces.
5. Advertise loopback interfaces through OSPF.
6. Configure passive interfaces where appropriate.
7. Prevent OSPF from running on R1's Internet-facing interface.
8. Configure R1 as an OSPF ASBR.
9. Inject a default route into the OSPF domain.
10. Verify OSPF neighbor adjacencies.
11. Examine OSPF-learned routes.
12. Determine which default route is installed on R2, R3, and R4.

---

# Topology

The lab uses four routers connected in a redundant topology:

```text
                         Internet
                            |
                         ISPR1
                            |
                    203.0.113.0/30
                            |
                           R1
                         /    \
                        /      \
               10.0.12.0/30  10.0.13.0/30
                    /            \
                   R2            R3
                    \            /
                     \          /
                  10.0.24.0/30  10.0.34.0/30
                       \        /
                         \    /
                           R4
                           |
                    192.168.4.0/24
```

R1, R2, R3, and R4 participate in **OSPF Area 0**.

ISPR1 is outside the OSPF domain and is **not configured as part of this lab**.

---

# IP Addressing

| Device | Interface | IP Address | Subnet Mask | Purpose |
|---|---|---|---|---|
| R1 | Loopback0 | `1.1.1.1` | `255.255.255.255` | R1 Loopback |
| R1 | G0/0 | `10.0.12.1` | `255.255.255.252` | R1-R2 |
| R1 | Fa1/0 | `10.0.13.1` | `255.255.255.252` | R1-R3 |
| R1 | Internet link | `203.0.113.1` | `/30` | R1-ISPR1 |
| R2 | Loopback0 | `2.2.2.2` | `255.255.255.255` | R2 Loopback |
| R2 | G0/0 | `10.0.12.2` | `255.255.255.252` | R2-R1 |
| R2 | Fa1/0 | `10.0.24.1` | `255.255.255.252` | R2-R4 |
| R3 | Loopback0 | `3.3.3.3` | `255.255.255.255` | R3 Loopback |
| R3 | Fa1/0 | `10.0.13.2` | `255.255.255.252` | R3-R1 |
| R3 | Fa2/0 | `10.0.34.1` | `255.255.255.252` | R3-R4 |
| R4 | Loopback0 | `4.4.4.4` | `255.255.255.255` | R4 Loopback |
| R4 | G0/0 | `192.168.4.254` | `255.255.255.0` | R4 LAN |
| R4 | Fa1/0 | `10.0.24.2` | `255.255.255.252` | R4-R2 |
| R4 | Fa2/0 | `10.0.34.2` | `255.255.255.252` | R4-R3 |

> **Note:** ISPR1 does not need to be configured for this lab.

---

# Part 1 — Configure Hostnames and IP Addresses

Configure the appropriate hostname and IP addresses on each router.

## R1

```cisco
enable
configure terminal

hostname R1

interface loopback0
 ip address 1.1.1.1 255.255.255.255

interface gigabitEthernet0/0
 ip address 10.0.12.1 255.255.255.252
 no shutdown

interface fastEthernet1/0
 ip address 10.0.13.1 255.255.255.252
 no shutdown
```

Configure R1's Internet-facing interface according to the topology if required by the supplied Packet Tracer file.

The important requirement is that **OSPF must not be enabled on R1's Internet link**.

---

## R2

```cisco
enable
configure terminal

hostname R2

interface loopback0
 ip address 2.2.2.2 255.255.255.255

interface gigabitEthernet0/0
 ip address 10.0.12.2 255.255.255.252
 no shutdown

interface fastEthernet1/0
 ip address 10.0.24.1 255.255.255.252
 no shutdown
```

---

## R3

```cisco
enable
configure terminal

hostname R3

interface loopback0
 ip address 3.3.3.3 255.255.255.255

interface fastEthernet1/0
 ip address 10.0.13.2 255.255.255.252
 no shutdown

interface fastEthernet2/0
 ip address 10.0.34.1 255.255.255.252
 no shutdown
```

---

## R4

```cisco
enable
configure terminal

hostname R4

interface loopback0
 ip address 4.4.4.4 255.255.255.255

interface gigabitEthernet0/0
 ip address 192.168.4.254 255.255.255.0
 no shutdown

interface fastEthernet1/0
 ip address 10.0.24.2 255.255.255.252
 no shutdown

interface fastEthernet2/0
 ip address 10.0.34.2 255.255.255.252
 no shutdown
```

---

# Part 2 — Configure Loopback Interfaces

Configure a `/32` loopback interface on every router.

| Router | Loopback Address |
|---|---|
| R1 | `1.1.1.1/32` |
| R2 | `2.2.2.2/32` |
| R3 | `3.3.3.3/32` |
| R4 | `4.4.4.4/32` |

Verify the interfaces:

```cisco
show ip interface brief
```

The loopback should appear as:

```text
Loopback0    1.1.1.1    YES manual    up    up
```

The loopback addresses will also be advertised through OSPF.

---

# Part 3 — Configure OSPF

Configure OSPF on each router.

All routers should participate in:

```text
Area 0
```

The lab examples use different OSPF process IDs on the routers:

```text
R1 → OSPF process 1
R2 → OSPF process 2
R3 → OSPF process 3
R4 → OSPF process 4
```

> OSPF process IDs are locally significant. Different process IDs can still form OSPF neighbor relationships as long as the routers agree on the relevant OSPF network parameters.

---

## R1 OSPF Configuration

R1 should advertise its internal OSPF interfaces and loopback.

Do **not** enable OSPF on the Internet-facing interface.

```cisco
router ospf 1
 passive-interface loopback0
 network 10.0.12.0 0.0.0.3 area 0
 network 10.0.13.0 0.0.0.3 area 0
 network 1.1.1.1 0.0.0.0 area 0
```

The important point is that the Internet link is deliberately excluded from the OSPF configuration.

---

## R2 OSPF Configuration

```cisco
router ospf 2
 passive-interface loopback0
 network 10.0.0.0 0.0.255.255 area 0
 network 2.2.2.2 0.0.0.0 area 0
```

The `10.0.0.0 0.0.255.255` wildcard matches R2's OSPF-facing interfaces.

---

## R3 OSPF Configuration

```cisco
router ospf 3
 passive-interface loopback0
 network 10.0.13.2 0.0.0.0 area 0
 network 10.0.34.1 0.0.0.0 area 0
 network 3.3.3.3 0.0.0.0 area 0
```

---

## R4 OSPF Configuration

R4 has two router-to-router interfaces and one LAN interface.

The LAN interface should be passive because there is no OSPF neighbor on the LAN.

```cisco
router ospf 4
 passive-interface gigabitEthernet0/0
 passive-interface loopback0
 network 0.0.0.0 255.255.255.255 area 0
```

Alternatively, more specific network statements can be used to control exactly which interfaces participate in OSPF.

---

# Part 4 — Configure Passive Interfaces

Passive interfaces are used where OSPF should advertise the connected network but should not form OSPF neighbor relationships.

In this lab:

### R1

```cisco
passive-interface loopback0
```

### R2

```cisco
passive-interface loopback0
```

### R3

```cisco
passive-interface loopback0
```

### R4

```cisco
passive-interface loopback0
passive-interface gigabitEthernet0/0
```

R4's `G0/0` is passive because it connects to the `192.168.4.0/24` LAN rather than another OSPF router.

---

# Part 5 — Verify OSPF Neighbor Relationships

Use:

```cisco
show ip ospf neighbor
```

On R1, the expected neighbors are:

```text
Neighbor ID     Pri   State           Dead Time   Address         Interface

2.2.2.2           1   FULL/DR         00:00:31    10.0.12.2       GigabitEthernet0/0
3.3.3.3           1   FULL/DR         00:00:32    10.0.13.2       FastEthernet1/0
```

This confirms that R1 has successfully established OSPF adjacencies with R2 and R3.

The important state is:

```text
FULL
```

A `FULL` adjacency indicates that the routers have synchronized their relevant OSPF link-state databases.

---

# Part 6 — Verify OSPF Interfaces

Use:

```cisco
show ip ospf interface
```

For example, R1's `G0/0` should show:

```text
GigabitEthernet0/0 is up, line protocol is up
  Internet address is 10.0.12.1/30, Area 0
  Process ID 1, Router ID 1.1.1.1
```

R1's `Fa1/0` should similarly show:

```text
FastEthernet1/0 is up, line protocol is up
  Internet address is 10.0.13.1/30, Area 0
```

R1's loopback should appear as:

```text
Loopback0 is up, line protocol is up
  Internet address is 1.1.1.1/32, Area 0
```

The loopback interface is treated as a stub host by OSPF.

---

# Part 7 — Configure R1 as an ASBR

R1 connects the internal OSPF domain to an external network.

Therefore, R1 acts as an:

> **ASBR — Autonomous System Boundary Router**

First, R1 needs a default route pointing toward the Internet/ISPR1.

The lab configuration uses:

```cisco
ip route 0.0.0.0 0.0.0.0 203.0.113.2
```

This creates a default static route on R1.

Verify it with:

```cisco
show ip route
```

R1 should have:

```text
S* 0.0.0.0/0 via 203.0.113.2
```

---

# Part 8 — Advertise the Default Route into OSPF

Once R1 has a default route, configure OSPF to advertise it to the rest of the OSPF domain.

On R1:

```cisco
router ospf 1
 default-information originate
```

This tells R1 to originate a default route into OSPF.

The final R1 OSPF configuration should resemble:

```cisco
router ospf 1
 passive-interface loopback0
 network 10.0.12.0 0.0.0.3 area 0
 network 10.0.13.0 0.0.0.3 area 0
 network 1.1.1.1 0.0.0.0 area 0
 default-information originate
```

The complete R1 routing configuration includes:

```cisco
ip route 0.0.0.0 0.0.0.0 203.0.113.2
```

and:

```cisco
default-information originate
```

---

# Understanding `default-information originate`

The command:

```cisco
default-information originate
```

allows R1 to advertise a default route into the OSPF domain.

The basic flow is:

```text
ISPR1
  |
  | Default route
  |
 R1
  |
  | OSPF default route
  |
  +-------- R2
  |
  +-------- R3
           |
           R4
```

R1 learns or has a default route toward the external network and then advertises that default route through OSPF.

---

# Part 9 — Verify the OSPF Database

Use:

```cisco
show ip ospf database
```

The OSPF database should contain router LSAs for:

```text
1.1.1.1
2.2.2.2
3.3.3.3
4.4.4.4
```

Example:

```text
Link ID         ADV Router
4.4.4.4         4.4.4.4
2.2.2.2         2.2.2.2
3.3.3.3         3.3.3.3
1.1.1.1         1.1.1.1
```

The presence of all four router IDs confirms that the OSPF domain has information about each participating router.

---

# Part 10 — Check the Routing Tables

The lab asks:

> What default route(s) were added to R2, R3, and R4?

Check each router with:

```cisco
show ip route
```

or specifically:

```cisco
show ip route 0.0.0.0
```

---

## R2

After R1 advertises the default route, R2 should learn an OSPF default route through R1.

It will appear with the `O*E2` code:

```text
O*E2 0.0.0.0/0 [110/1] via 10.0.12.1
```

The exact metric display may vary depending on the Packet Tracer topology and configuration.

---

## R3

R3 should also learn the default route through R1:

```text
O*E2 0.0.0.0/0 [110/1] via 10.0.13.1
```

Again, the exact metric may vary.

---

## R4

R4 has two possible paths toward R1:

```text
R4 → R2 → R1
```

and:

```text
R4 → R3 → R1
```

Therefore, depending on the OSPF costs, R4 can potentially install multiple equal-cost paths to the default route.

Check with:

```cisco
show ip route 0.0.0.0
```

The route will be identified as an OSPF external Type 2 route:

```text
O*E2 0.0.0.0/0
```

---

# Why Is the Route `O*E2`?

The route code can be broken down as follows:

```text
O*E2
│││
││└── OSPF External Type 2
│└── Candidate default route
└── OSPF
```

### `O`

The route was learned through OSPF.

### `*`

The route is a candidate default route.

### `E2`

The route is an **OSPF external Type 2** route.

R1 is redistributing the concept of the external/default route into the OSPF domain, making R1 an ASBR.

---

# Important Observation

The default route is **not learned through EIGRP or static routing on R2/R3/R4**.

Instead, R2, R3, and R4 learn it through OSPF.

The path is:

```text
R2 → R1 → Internet
```

```text
R3 → R1 → Internet
```

and from R4:

```text
R4 → R2 → R1 → Internet
```

or:

```text
R4 → R3 → R1 → Internet
```

depending on OSPF's shortest-path calculation.

---

# Part 11 — Verify Connectivity

Test the loopback interfaces from R1:

```cisco
ping 2.2.2.2
ping 3.3.3.3
ping 4.4.4.4
```

From R2:

```cisco
ping 1.1.1.1
ping 3.3.3.3
ping 4.4.4.4
```

From R3:

```cisco
ping 1.1.1.1
ping 2.2.2.2
ping 4.4.4.4
```

From R4:

```cisco
ping 1.1.1.1
ping 2.2.2.2
ping 3.3.3.3
```

---

# Useful Verification Commands

### Interface status

```cisco
show ip interface brief
```

### OSPF neighbors

```cisco
show ip ospf neighbor
```

### OSPF interfaces

```cisco
show ip ospf interface
```

### OSPF configuration

```cisco
show ip protocols
```

### OSPF database

```cisco
show ip ospf database
```

### Complete routing table

```cisco
show ip route
```

### OSPF routes

```cisco
show ip route ospf
```

### Default route

```cisco
show ip route 0.0.0.0
```

### Running configuration

```cisco
show running-config
```

### Connectivity test

```cisco
ping <destination-ip>
```

### Trace the selected path

```cisco
traceroute <destination-ip>
```

---

# Troubleshooting

## OSPF Neighbor Is Not Forming

Check:

```cisco
show ip ospf neighbor
show ip ospf interface
show ip interface brief
```

Verify:

- The interface is `up/up`.
- The correct IP address is configured.
- Both interfaces are in Area 0.
- The interface is not passive.
- The OSPF network statement matches the interface.
- OSPF is enabled on the correct interface.

---

## R1 Does Not Have a Default Route

Check:

```cisco
show ip route
```

R1 should have a default route similar to:

```text
S* 0.0.0.0/0 via 203.0.113.2
```

If it is missing, configure:

```cisco
ip route 0.0.0.0 0.0.0.0 203.0.113.2
```

---

## R2/R3/R4 Do Not Learn the Default Route

First check R1:

```cisco
show ip route 0.0.0.0
```

Then:

```cisco
show running-config | section router ospf
```

Make sure R1 has:

```cisco
default-information originate
```

Also verify that R1 has an actual default route.

The standard `default-information originate` command requires R1 to have a default route in its routing table before advertising it.

---

## R1 Forms an OSPF Neighbor on the Internet Link

This should **not** happen.

The lab specifically requires:

> Do not enable OSPF on R1's Internet link.

Check:

```cisco
show ip ospf interface
```

The Internet-facing interface should not appear as an OSPF interface.

Also check:

```cisco
show running-config
```

and ensure there is no OSPF network statement matching the Internet link.

---

# Configuration Save

Save the configuration on every router:

```cisco
write memory
```

or:

```cisco
copy running-config startup-config
```

Verify that the configuration has been saved by checking:

```cisco
show startup-config
```

---

# Final Verification Checklist

- [ ] R1 hostname configured
- [ ] R2 hostname configured
- [ ] R3 hostname configured
- [ ] R4 hostname configured
- [ ] Correct IP addresses configured
- [ ] Required router interfaces enabled
- [ ] R1 Loopback0 = `1.1.1.1/32`
- [ ] R2 Loopback0 = `2.2.2.2/32`
- [ ] R3 Loopback0 = `3.3.3.3/32`
- [ ] R4 Loopback0 = `4.4.4.4/32`
- [ ] OSPF configured on R1
- [ ] OSPF configured on R2
- [ ] OSPF configured on R3
- [ ] OSPF configured on R4
- [ ] All internal OSPF interfaces are in Area 0
- [ ] Loopback interfaces configured as passive
- [ ] R4 LAN interface configured as passive
- [ ] R1 Internet link excluded from OSPF
- [ ] R1 has a default static route
- [ ] R1 configured as an ASBR
- [ ] `default-information originate` configured on R1
- [ ] OSPF neighbors reach `FULL` state
- [ ] All four router IDs appear in the OSPF database
- [ ] R2 learns an OSPF default route
- [ ] R3 learns an OSPF default route
- [ ] R4 learns an OSPF default route
- [ ] Connectivity tests are successful
- [ ] Configurations saved

---

# Lab Questions

## 1. What default route was added to R2?

R2 learns an OSPF external default route from R1:

```text
O*E2 0.0.0.0/0 via 10.0.12.1
```

## 2. What default route was added to R3?

R3 learns the default route through R1:

```text
O*E2 0.0.0.0/0 via 10.0.13.1
```

## 3. What default route was added to R4?

R4 learns the default route through the OSPF topology. Because R4 has paths through both R2 and R3, OSPF may install multiple equal-cost paths depending on the configured interface costs.

The route is identified as:

```text
O*E2 0.0.0.0/0
```

The exact next-hop entries should be confirmed with:

```cisco
show ip route 0.0.0.0
```

---

# Key Concepts Learned

### OSPF Area 0

All four internal routers participate in the OSPF backbone area:

```text
Area 0
```

### Passive Interfaces

Passive interfaces advertise their connected networks through OSPF but do not form OSPF neighbor relationships.

### ASBR

R1 is an **Autonomous System Boundary Router** because it connects the OSPF routing domain to an external network.

### Default Route Origination

R1 uses:

```cisco
default-information originate
```

to advertise its default route into OSPF.

### OSPF External Type 2

The default route learned by the internal routers is represented as:

```text
O*E2
```

The `E2` indicates an OSPF external Type 2 route.

---

# Conclusion

This lab demonstrates how OSPF can be used to build a dynamic routing domain while allowing one router to provide connectivity to an external network.

R1 acts as the **ASBR**, maintains the default route toward the Internet, and advertises that default route into OSPF.

The resulting routing flow is:

```text
                    Internet
                       |
                     ISPR1
                       |
                  Default Route
                       |
                      R1
                    /    \
                   /      \
                 R2        R3
                   \      /
                    \    /
                      R4
```

The key configuration on R1 is:

```cisco
ip route 0.0.0.0 0.0.0.0 203.0.113.2

router ospf 1
 default-information originate
```

And the key verification command is:

```cisco
show ip route 0.0.0.0
```

On R2, R3, and R4, the expected OSPF-learned default route is identified by:

```text
O*E2 0.0.0.0/0
```