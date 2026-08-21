# EIGRP Unequal-Cost Load Balancing Lab

## Overview

This lab focuses on configuring **EIGRP (Enhanced Interior Gateway Routing Protocol)** across a four-router topology and using EIGRP's **unequal-cost load-balancing** capability.

The lab covers:

- Basic router configuration
- Hostname and IP address configuration
- Loopback interfaces
- EIGRP AS 100
- EIGRP network advertisement
- Passive interfaces
- Disabling automatic summarization
- EIGRP neighbor formation
- EIGRP route verification
- EIGRP variance for unequal-cost load balancing
- Verification of routing toward `192.168.4.0/24`

> **Platform:** Cisco Packet Tracer  
> **Routing Protocol:** EIGRP  
> **EIGRP Autonomous System:** 100

---

## Lab Objectives

By completing this lab, you should be able to:

1. Configure hostnames and IP addresses on Cisco routers.
2. Enable router interfaces.
3. Configure `/32` loopback interfaces.
4. Configure EIGRP AS 100.
5. Advertise connected networks using EIGRP.
6. Disable EIGRP automatic summarization.
7. Configure passive interfaces appropriately.
8. Verify EIGRP neighbor adjacencies.
9. Verify EIGRP-learned routes.
10. Configure EIGRP variance for unequal-cost load balancing.
11. Verify that R1 can use multiple paths toward `192.168.4.0/24`.

---

## Topology

The lab consists of four routers:

```text
                 10.0.12.0/30
          R1 ---------------- R2
          |                    |
          |                    |
10.0.13.0/30              10.0.24.0/30
          |                    |
          |                    |
          R3 ---------------- R4
                 10.0.34.0/30

R4 LAN:
192.168.4.0/24
```

The routers form two paths between R1 and R4:

```text
R1 ---- R2 ---- R4
 \               /
  \---- R3 -----/
```

This topology provides the paths required to demonstrate EIGRP's unequal-cost load-balancing behavior.

---

## IP Addressing

| Device | Interface | IP Address | Subnet Mask | Purpose |
|---|---|---|---|---|
| R1 | Loopback0 | `1.1.1.1` | `255.255.255.255` | Router ID / Loopback |
| R1 | G0/0 | `10.0.12.1` | `255.255.255.252` | R1-R2 |
| R1 | Fa1/0 | `10.0.13.1` | `255.255.255.252` | R1-R3 |
| R2 | Loopback0 | `2.2.2.2` | `255.255.255.255` | Router ID / Loopback |
| R2 | G0/0 | `10.0.12.2` | `255.255.255.252` | R2-R1 |
| R2 | Fa1/0 | `10.0.24.1` | `255.255.255.252` | R2-R4 |
| R3 | Loopback0 | `3.3.3.3` | `255.255.255.255` | Router ID / Loopback |
| R3 | Fa1/0 | `10.0.13.2` | `255.255.255.252` | R3-R1 |
| R3 | Fa2/0 | `10.0.34.1` | `255.255.255.252` | R3-R4 |
| R4 | Loopback0 | `4.4.4.4` | `255.255.255.255` | Router ID / Loopback |
| R4 | G0/0 | `192.168.4.254` | `255.255.255.0` | R4 LAN |
| R4 | Fa1/0 | `10.0.24.2` | `255.255.255.252` | R4-R2 |
| R4 | Fa2/0 | `10.0.34.2` | `255.255.255.252` | R4-R3 |

---

# Part 1 — Configure Hostnames and IP Addresses

Configure the appropriate hostname and IP addresses on every router.

### R1

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

### R2

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

### R3

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

### R4

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

Each router receives a `/32` loopback address:

| Router | Loopback |
|---|---|
| R1 | `1.1.1.1/32` |
| R2 | `2.2.2.2/32` |
| R3 | `3.3.3.3/32` |
| R4 | `4.4.4.4/32` |

Loopback interfaces are useful for providing stable router addresses because they remain operational as long as the router itself is operational.

Verify the loopbacks with:

```cisco
show ip interface brief
```

Expected example:

```text
Loopback0    1.1.1.1    YES manual    up    up
```

---

# Part 3 — Configure EIGRP

Configure **EIGRP AS 100** on every router.

The requirements are:

- EIGRP AS `100`
- Advertise the required interfaces
- Disable automatic summarization
- Enable EIGRP on loopback interfaces
- Configure loopbacks as passive interfaces
- Keep router-to-router interfaces active for EIGRP neighbor formation

---

## R1 EIGRP Configuration

```cisco
router eigrp 100
 network 10.0.12.0 0.0.0.3
 network 10.0.13.0 0.0.0.3
 network 1.1.1.1 0.0.0.0
 no auto-summary
 passive-interface loopback0
```

## R2 EIGRP Configuration

```cisco
router eigrp 100
 network 10.0.12.0 0.0.0.3
 network 10.0.24.0 0.0.0.3
 network 2.2.2.2 0.0.0.0
 no auto-summary
 passive-interface loopback0
```

## R3 EIGRP Configuration

```cisco
router eigrp 100
 network 10.0.13.0 0.0.0.3
 network 10.0.34.0 0.0.0.3
 network 3.3.3.3 0.0.0.0
 no auto-summary
 passive-interface loopback0
```

## R4 EIGRP Configuration

```cisco
router eigrp 100
 network 10.0.24.0 0.0.0.3
 network 10.0.34.0 0.0.0.3
 network 192.168.4.0 0.0.0.255
 network 4.4.4.4 0.0.0.0
 no auto-summary
 passive-interface loopback0
 passive-interface gigabitEthernet0/0
```

R4's `G0/0` is configured as passive because it connects to the LAN rather than another EIGRP router.

---

# Part 4 — Verify EIGRP

After configuring EIGRP, verify that the expected neighbor relationships have formed.

Use:

```cisco
show ip eigrp neighbors
```

On R1, the expected neighbors are:

```text
IP-EIGRP neighbors for process 100

H   Address         Interface
0   10.0.12.2       Gig0/0
1   10.0.13.2       Fa1/0
```

This confirms that R1 has formed EIGRP adjacencies with R2 and R3.

---

## Verify EIGRP Configuration

Use:

```cisco
show ip protocols
```

You should see:

```text
Routing Protocol is "eigrp 100"
EIGRP maximum hopcount 100
EIGRP maximum metric variance 1
Automatic network summarization is not in effect
Maximum path: 4
```

You should also see the appropriate passive interfaces.

For example, R3 should show:

```text
Passive Interface(s):

    Loopback0
```

R4 should show:

```text
Passive Interface(s):

    GigabitEthernet0/0
    Loopback0
```

---

# Part 5 — Verify the Routing Table

Use:

```cisco
show ip route
```

or:

```cisco
show ip route eigrp
```

On R1, EIGRP-learned routes should appear with the `D` code.

Example:

```text
D       2.2.2.2/32 [90/130816] via 10.0.12.2
D       3.3.3.3/32 [90/156160] via 10.0.13.2
D       4.4.4.4/32 [90/156416] via 10.0.12.2
D       10.0.24.0 [90/28416] via 10.0.12.2
D       10.0.34.0 [90/30720] via 10.0.13.2
D       192.168.4.0/24 [90/28672] via 10.0.12.2
```

The important destination for this lab is:

```text
192.168.4.0/24
```

---

# Part 6 — Configure Unequal-Cost Load Balancing

EIGRP normally performs **equal-cost load balancing**.

One of EIGRP's advantages is that it can also perform **unequal-cost load balancing** using the `variance` command.

The lab requires R1 to perform unequal-cost load balancing toward:

```text
192.168.4.0/24
```

Configure:

```cisco
R1(config)# router eigrp 100
R1(config-router)# variance 1
```

The required EIGRP variance is:

```text
Variance = 1
```

The lab also specifies:

```text
EIGRP maximum hopcount = 100
Maximum metric variance = 1
Maximum paths = 4
```

Verify the configuration with:

```cisco
show ip protocols
```

Expected output should include:

```text
EIGRP maximum hopcount 100
EIGRP maximum metric variance 1
Maximum path: 4
```

---

# Understanding EIGRP Variance

The `variance` command determines which feasible paths EIGRP can install in the routing table.

The basic concept is:

```text
Feasible Path Metric <= Best Path Metric × Variance
```

For example, with:

```text
Best path metric = 1000
Variance = 2
```

EIGRP can potentially install a feasible path with a metric of up to:

```text
1000 × 2 = 2000
```

However, **variance alone does not make an arbitrary route usable**. The alternate route must satisfy EIGRP's **feasibility condition**.

This prevents routing loops while allowing EIGRP to use paths with different metrics.

---

# Part 7 — Verify Unequal-Cost Load Balancing

On R1, check the route to the destination network:

```cisco
show ip route 192.168.4.0
```

You can also use:

```cisco
show ip route eigrp
```

The goal is to verify whether R1 has installed multiple valid EIGRP paths toward:

```text
192.168.4.0/24
```

You can also inspect the EIGRP topology table:

```cisco
show ip eigrp topology
```

For the specific network:

```cisco
show ip eigrp topology 192.168.4.0 255.255.255.0
```

This helps identify:

- Successor
- Feasible successor
- Reported distance
- Feasible distance
- Path metrics

---

# Part 8 — Connectivity Testing

Test connectivity between the router loopbacks.

From R1:

```cisco
ping 2.2.2.2
ping 3.3.3.3
ping 4.4.4.4
```

Test the R4 LAN:

```cisco
ping 192.168.4.254
```

You should receive successful replies once EIGRP has converged.

You can also use:

```cisco
traceroute 192.168.4.254
```

to observe the path selected by the routing table.

---

# Useful Verification Commands

### Interface status

```cisco
show ip interface brief
```

### Running configuration

```cisco
show running-config
```

### EIGRP neighbors

```cisco
show ip eigrp neighbors
```

### EIGRP configuration

```cisco
show ip protocols
```

### EIGRP topology

```cisco
show ip eigrp topology
```

### Routing table

```cisco
show ip route
```

### EIGRP routes only

```cisco
show ip route eigrp
```

### Specific destination

```cisco
show ip route 192.168.4.0
```

### Connectivity

```cisco
ping <destination-ip>
```

### Path testing

```cisco
traceroute <destination-ip>
```

---

# Troubleshooting

## EIGRP Neighbor Not Forming

Check:

```cisco
show ip interface brief
show ip eigrp neighbors
show ip protocols
```

Verify that:

- The interfaces are `up/up`.
- IP addresses are correctly configured.
- Both routers use EIGRP AS `100`.
- The connected networks are included in EIGRP.
- The router-to-router interface is not passive.
- The subnet masks match.

---

## Loopback Not Appearing in the Routing Table

Verify:

```cisco
show ip interface brief
```

Then check:

```cisco
show ip protocols
```

The loopback should be included in the EIGRP network statements.

For example:

```cisco
network 1.1.1.1 0.0.0.0
```

The loopback should be passive, but **passive does not mean the network is excluded from EIGRP**. It means EIGRP advertises the network without sending EIGRP hello packets through that interface.

---

## R4 LAN Not Being Advertised

On R4, verify:

```cisco
show ip protocols
```

Make sure:

```cisco
network 192.168.4.0 0.0.0.255
```

is configured.

Because `G0/0` is a LAN-facing interface, it can remain passive:

```cisco
passive-interface gigabitEthernet0/0
```

---

## Unequal-Cost Path Not Appearing

Check:

```cisco
show ip eigrp topology 192.168.4.0 255.255.255.0
```

Then verify:

```cisco
show ip protocols
```

Make sure the configured variance is correct:

```cisco
variance 1
```

Remember that an alternate route must satisfy EIGRP's feasibility condition before it can be used as a feasible successor. Simply configuring a variance value does not automatically make every available path eligible for load balancing.

---

# Key Concepts Learned

### EIGRP AS

All routers in this lab participate in:

```text
EIGRP AS 100
```

Routers must use the same EIGRP autonomous-system number to form neighbor relationships.

### Passive Interfaces

Passive interfaces allow a network to be advertised through EIGRP without sending EIGRP hello packets through that interface.

Typical examples in this lab:

```text
Loopback0
R4 GigabitEthernet0/0
```

### EIGRP Administrative Distance

EIGRP uses:

```text
Internal EIGRP: 90
External EIGRP: 170
```

### EIGRP Maximum Paths

The default maximum number of paths shown in the lab is:

```text
4
```

### EIGRP Variance

Variance enables EIGRP to consider unequal-metric paths for load balancing when the alternate paths meet EIGRP's feasibility requirements.

---

# Final Verification Checklist

- [ ] Hostnames configured on R1, R2, R3 and R4
- [ ] All required IP addresses configured
- [ ] Router interfaces enabled with `no shutdown`
- [ ] Loopback0 configured on every router
- [ ] R1 loopback = `1.1.1.1/32`
- [ ] R2 loopback = `2.2.2.2/32`
- [ ] R3 loopback = `3.3.3.3/32`
- [ ] R4 loopback = `4.4.4.4/32`
- [ ] EIGRP AS 100 configured
- [ ] Required networks advertised
- [ ] EIGRP automatic summarization disabled
- [ ] Loopback interfaces configured as passive
- [ ] R4 LAN interface configured as passive
- [ ] EIGRP neighbors established
- [ ] EIGRP routes installed
- [ ] EIGRP variance configured on R1
- [ ] `192.168.4.0/24` learned by R1
- [ ] Connectivity tests successful
- [ ] Configuration saved with `write memory` / `do wr`

---

## Conclusion

This lab demonstrates how EIGRP dynamically exchanges routing information between multiple routers and how its metric calculation can be used to support **unequal-cost load balancing**.

The most important verification commands are:

```cisco
show ip eigrp neighbors
show ip protocols
show ip eigrp topology
show ip route eigrp
show ip route 192.168.4.0
```

Successful completion should result in a converged EIGRP topology where R1 has learned the `192.168.4.0/24` network through the available paths and can use EIGRP's variance mechanism where the topology and feasibility condition permit unequal-cost load balancing.