# Cisco HSRP and STP Synchronization Lab

## 📌 Lab Overview

This lab demonstrates how to configure **Hot Standby Router Protocol (HSRP)** on two multilayer switches and synchronize the HSRP roles with **Spanning Tree Protocol (STP)**.

The goal is to provide first-hop gateway redundancy while ensuring that Layer 2 traffic follows the same switch that is actively forwarding Layer 3 traffic.

The design uses **DSW1 and DSW2** as redundant distribution switches.

---

# 🎯 Lab Objectives

Configure HSRP and STP so that:

### VLAN 10

- **DSW1** is the HSRP Active router.
- **DSW2** is the HSRP Standby router.
- **DSW1** is the STP Root Bridge.
- **DSW2** is the STP Secondary Root Bridge.

### VLAN 20

- **DSW2** is the HSRP Active router.
- **DSW1** is the HSRP Standby router.
- **DSW2** is the STP Root Bridge.
- **DSW1** is the STP Secondary Root Bridge.

This arrangement provides **load sharing** across the two distribution switches.

---

# 🖥️ Lab Topology

```text
                    +----------------+
                    |     DSW1       |
                    |                |
                    | VLAN 10:       |
                    | HSRP Active    |
                    | STP Root       |
                    |                |
                    | VLAN 20:       |
                    | HSRP Standby   |
                    | STP Secondary  |
                    +-------+--------+
                            |
                       Trunk Links
                            |
                    +-------+--------+
                    |     DSW2       |
                    |                |
                    | VLAN 10:       |
                    | HSRP Standby   |
                    | STP Secondary  |
                    |                |
                    | VLAN 20:       |
                    | HSRP Active    |
                    | STP Root       |
                    +----------------+
```

---

# 🌐 IP Addressing Plan

| VLAN | DSW1 | DSW2 | HSRP Virtual IP |
|---|---|---|---|
| VLAN 10 | `10.0.10.1/24` | `10.0.10.2/24` | `10.0.10.254` |
| VLAN 20 | `10.0.20.1/24` | `10.0.20.2/24` | `10.0.20.254` |

The HSRP virtual IP is configured as the default gateway for hosts in each VLAN.

---

# 1. Configure STP on DSW1

The STP priorities determine which switch becomes the root bridge for each VLAN.

A lower STP priority is preferred.

For VLAN 10, DSW1 should be the root:

```cisco
DSW1(config)# spanning-tree vlan 10 priority 24576
```

For VLAN 20, DSW1 should be the secondary root:

```cisco
DSW1(config)# spanning-tree vlan 20 priority 28672
```

The resulting configuration is:

```cisco
spanning-tree vlan 10 priority 24576
spanning-tree vlan 20 priority 28672
```

### STP Role on DSW1

| VLAN | Priority | Role |
|---|---:|---|
| VLAN 10 | 24576 | Root |
| VLAN 20 | 28672 | Secondary |

---

# 2. Configure STP on DSW2

Reverse the priorities on DSW2.

For VLAN 20, DSW2 should be the root:

```cisco
DSW2(config)# spanning-tree vlan 20 priority 24576
```

For VLAN 10, DSW2 should be the secondary root:

```cisco
DSW2(config)# spanning-tree vlan 10 priority 28672
```

The resulting configuration is:

```cisco
spanning-tree vlan 20 priority 24576
spanning-tree vlan 10 priority 28672
```

### STP Role on DSW2

| VLAN | Priority | Role |
|---|---:|---|
| VLAN 10 | 28672 | Secondary |
| VLAN 20 | 24576 | Root |

---

# 3. Configure HSRP on DSW1

HSRP provides a redundant default gateway for hosts.

DSW1 must be active for VLAN 10 and standby for VLAN 20.

## VLAN 10 — DSW1 Active

Enter the VLAN 10 interface:

```cisco
DSW1(config)# interface vlan 10
```

Configure the SVI address:

```cisco
DSW1(config-if)# ip address 10.0.10.1 255.255.255.0
```

Enable HSRP version 2:

```cisco
DSW1(config-if)# standby version 2
```

Configure the HSRP virtual IP:

```cisco
DSW1(config-if)# standby 10 ip 10.0.10.254
```

Give DSW1 a higher priority:

```cisco
DSW1(config-if)# standby 10 priority 105
```

Enable preemption:

```cisco
DSW1(config-if)# standby 10 preempt
```

### Result

DSW1 becomes the preferred HSRP Active router for VLAN 10.

---

# 4. Configure HSRP on DSW1 for VLAN 20

DSW1 must be the standby router for VLAN 20.

```cisco
DSW1(config)# interface vlan 20
DSW1(config-if)# ip address 10.0.20.1 255.255.255.0
DSW1(config-if)# standby version 2
DSW1(config-if)# standby 20 ip 10.0.20.254
DSW1(config-if)# standby 20 priority 95
DSW1(config-if)# standby 20 preempt
```

The lower priority of `95` ensures that DSW2, with priority `105`, becomes the HSRP Active router.

---

# 5. Configure HSRP on DSW2

DSW2 must be standby for VLAN 10 and active for VLAN 20.

## VLAN 10 — DSW2 Standby

```cisco
DSW2(config)# interface vlan 10
DSW2(config-if)# ip address 10.0.10.2 255.255.255.0
DSW2(config-if)# standby version 2
DSW2(config-if)# standby 10 ip 10.0.10.254
DSW2(config-if)# standby 10 priority 95
DSW2(config-if)# standby 10 preempt
```

DSW2 has a priority of `95`, so DSW1 with priority `105` becomes active.

---

# 6. Configure HSRP on DSW2 for VLAN 20

DSW2 must be active for VLAN 20.

```cisco
DSW2(config)# interface vlan 20
DSW2(config-if)# ip address 10.0.20.2 255.255.255.0
DSW2(config-if)# standby version 2
DSW2(config-if)# standby 20 ip 10.0.20.254
DSW2(config-if)# standby 20 priority 105
DSW2(config-if)# standby 20 preempt
```

Because DSW2 has the higher priority of `105`, it becomes the HSRP Active router for VLAN 20.

---

# 7. Why HSRP and STP Must Be Synchronized

HSRP and STP perform different jobs:

- **HSRP** determines which switch provides the default gateway.
- **STP** determines which switch is the Layer 2 forwarding root.

If these roles are not aligned, traffic may take an inefficient path.

For example, if DSW1 is the HSRP Active router but DSW2 is the STP Root Bridge for the same VLAN, hosts may send traffic toward DSW1 while Layer 2 forwarding prefers DSW2.

This can result in unnecessary traffic crossing the distribution-switch link.

The recommended design is therefore:

```text
VLAN 10:
HSRP Active  = DSW1
STP Root     = DSW1

VLAN 20:
HSRP Active  = DSW2
STP Root     = DSW2
```

This keeps the Layer 2 and Layer 3 forwarding paths aligned.

---

# 8. Verify HSRP

Use the following command on both switches:

```cisco
show standby brief
```

### Expected DSW1 Output

```text
Interface   Grp  Pri  P  State    Active       Standby       Virtual IP
Vl10        10   105  P  Active   local        10.0.10.2     10.0.10.254
Vl20        20    95  P  Standby  10.0.20.2   local          10.0.20.254
```

The important results are:

- VLAN 10 → **Active**
- VLAN 20 → **Standby**

### Expected DSW2 Output

```text
Interface   Grp  Pri  P  State    Active       Standby       Virtual IP
Vl10        10    95  P  Standby  10.0.10.1   local          10.0.10.254
Vl20        20   105  P  Active   local        10.0.20.1     10.0.20.254
```

The important results are:

- VLAN 10 → **Standby**
- VLAN 20 → **Active**

---

# 9. Verify STP

Check the STP status on DSW1:

```cisco
DSW1# show spanning-tree vlan 10
DSW1# show spanning-tree vlan 20
```

Check DSW2:

```cisco
DSW2# show spanning-tree vlan 10
DSW2# show spanning-tree vlan 20
```

### Expected STP Roles

| VLAN | DSW1 | DSW2 |
|---|---|---|
| VLAN 10 | Root Bridge | Secondary Root |
| VLAN 20 | Secondary Root | Root Bridge |

You can also verify the STP priorities:

```cisco
show spanning-tree vlan 10
show spanning-tree vlan 20
```

The switch with the lower bridge priority should become the root bridge.

---

# 10. Verify the SVI Configuration

On DSW1:

```cisco
show running-config interface vlan 10
show running-config interface vlan 20
```

On DSW2:

```cisco
show running-config interface vlan 10
show running-config interface vlan 20
```

Expected addressing:

### DSW1

```text
VLAN 10: 10.0.10.1/24
VLAN 20: 10.0.20.1/24
```

### DSW2

```text
VLAN 10: 10.0.10.2/24
VLAN 20: 10.0.20.2/24
```

---

# 11. Verify Trunk Interfaces

The distribution switches use trunk interfaces to carry multiple VLANs.

Check the trunk status:

```cisco
show interfaces trunk
```

The relevant interfaces should be operating as trunks and should allow VLAN 10 and VLAN 20.

---

# 12. Test HSRP Failover

One of the most important purposes of HSRP is gateway redundancy.

### Test VLAN 10

Since DSW1 is normally active:

```text
VLAN 10
        |
   DSW1 Active
        |
  10.0.10.254
```

If DSW1 becomes unavailable, DSW2 should take over as the Active HSRP router.

Verify with:

```cisco
show standby brief
```

### Test VLAN 20

DSW2 is normally active:

```text
VLAN 20
        |
   DSW2 Active
        |
  10.0.20.254
```

If DSW2 becomes unavailable, DSW1 should take over.

The `preempt` command ensures that the switch with the higher priority can reclaim the Active role when it becomes available again.

---

# 13. Final HSRP and STP Design

The completed design should look like this:

```text
                    VLAN 10
          +-------------------------+
          |                         |
       DSW1                       DSW2
     HSRP ACTIVE                HSRP STANDBY
     STP ROOT                  STP SECONDARY
     Priority 105               Priority 95
          |                         |
          +-------------------------+

                    VLAN 20
          +-------------------------+
          |                         |
       DSW1                       DSW2
    HSRP STANDBY                HSRP ACTIVE
    STP SECONDARY                 STP ROOT
     Priority 95                Priority 105
          |                         |
          +-------------------------+
```

---

# 📊 Final Configuration Summary

| Feature | VLAN 10 | VLAN 20 |
|---|---|---|
| HSRP Virtual IP | `10.0.10.254` | `10.0.20.254` |
| DSW1 SVI | `10.0.10.1` | `10.0.20.1` |
| DSW2 SVI | `10.0.10.2` | `10.0.20.2` |
| HSRP Active | **DSW1** | **DSW2** |
| HSRP Standby | DSW2 | DSW1 |
| STP Root | **DSW1** | **DSW2** |
| STP Secondary | DSW2 | DSW1 |
| Active HSRP Priority | 105 | 105 |
| Standby HSRP Priority | 95 | 95 |
| HSRP Version | 2 | 2 |
| Preemption | Enabled | Enabled |

---

# 🧪 Verification Checklist

- [x] HSRP version 2 configured on both switches.
- [x] VLAN 10 HSRP virtual IP configured as `10.0.10.254`.
- [x] VLAN 20 HSRP virtual IP configured as `10.0.20.254`.
- [x] DSW1 has HSRP priority 105 for VLAN 10.
- [x] DSW2 has HSRP priority 95 for VLAN 10.
- [x] DSW2 has HSRP priority 105 for VLAN 20.
- [x] DSW1 has HSRP priority 95 for VLAN 20.
- [x] HSRP preemption enabled.
- [x] DSW1 configured as STP root for VLAN 10.
- [x] DSW2 configured as STP root for VLAN 20.
- [x] DSW2 configured as STP secondary for VLAN 10.
- [x] DSW1 configured as STP secondary for VLAN 20.
- [x] HSRP Active and STP Root roles are synchronized.
- [x] HSRP status verified using `show standby brief`.
- [x] STP status verified using `show spanning-tree`.
- [x] Trunk interfaces verified.
- [x] HSRP failover tested.

---

# 🧠 Key Concepts

### HSRP

**Hot Standby Router Protocol** provides a virtual default gateway shared between multiple routers or multilayer switches.

Hosts use the virtual IP address rather than the physical IP address of either distribution switch.

### HSRP Priority

The switch with the highest HSRP priority becomes Active.

In this lab:

```text
105 = Preferred / Active
95  = Standby
```

### HSRP Preemption

The `preempt` command allows a higher-priority switch to regain the Active role after recovering from a failure.

### STP Root Bridge

The STP root bridge is selected using the lowest bridge ID, which is influenced by the configured STP priority.

A lower priority wins.

### Synchronizing HSRP and STP

The most important design principle in this lab is:

> **The HSRP Active switch should also be the STP Root Bridge for the same VLAN.**

This avoids inefficient Layer 2 paths and provides a more predictable and efficient network topology.

## Conclusion

This lab demonstrates how **HSRP and STP can be coordinated to provide gateway redundancy, Layer 2 redundancy, and traffic load sharing**.

DSW1 handles primary forwarding for VLAN 10, while DSW2 handles primary forwarding for VLAN 20. If either distribution switch fails, the other can assume the gateway role through HSRP, providing continued network connectivity.

The final design therefore achieves both **redundancy and efficient traffic forwarding**.