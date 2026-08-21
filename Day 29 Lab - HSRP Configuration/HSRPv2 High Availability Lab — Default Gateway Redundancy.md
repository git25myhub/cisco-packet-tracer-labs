# HSRPv2 High Availability Lab — Default Gateway Redundancy

## Overview

This lab introduces **Hot Standby Router Protocol version 2 (HSRPv2)** and demonstrates how HSRP provides a highly available default gateway for end devices.

PC1 and PC2 initially use a physical router interface as their default gateway. The lab replaces this dependency with an **HSRP Virtual IP (VIP)** shared between R1 and R2.

The exercise demonstrates:

- Identifying the existing default gateway.
- Configuring HSRPv2.
- Controlling HSRP active/standby router selection using priority.
- Enabling HSRP preemption.
- Using an HSRP virtual IP as the PC default gateway.
- Understanding the virtual MAC address.
- Testing gateway redundancy during router failure.
- Verifying automatic failover and recovery.

---

# Topology and Addressing

The LAN uses the `10.0.1.0/24` network.

| Device | Interface | IP Address | Role |
|---|---|---|---|
| R1 | G0/0 | `10.0.1.253/24` | HSRP Active |
| R2 | G0/0 | `10.0.1.252/24` | HSRP Standby |
| PC1/PC2 | FastEthernet0 | `10.0.1.x/24` | End devices |
| HSRP VIP | — | `10.0.1.254` | Virtual Default Gateway |

The HSRP configuration uses:

```text
HSRP Group: 1
HSRP Version: 2
Virtual IP: 10.0.1.254
```

The external server is:

```text
8.8.8.8
```

---

# 1. Test the Existing Default Gateway

Before configuring HSRP, determine how the PCs currently reach the external server.

## Test Connectivity

From PC1 or PC2:

```text
ping 8.8.8.8
```

The ping should succeed.

Example result:

```text
Reply from 8.8.8.8: bytes=32 time=14ms TTL=254
Reply from 8.8.8.8: bytes=32 time=1ms TTL=254
Reply from 8.8.8.8: bytes=32 time<1ms TTL=254
Reply from 8.8.8.8: bytes=32 time=13ms TTL=254

Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

## Check the PC Configuration

Run:

```text
ipconfig
```

The PC configuration shows:

```text
IPv4 Address:     10.0.1.2
Subnet Mask:      255.255.255.0
Default Gateway:  10.0.1.253
```

### Current Default Gateway

The default gateway is:

```text
10.0.1.253
```

This is the **physical IP address of R1**.

## Verify With Traceroute

Run:

```text
tracert 8.8.8.8
```

The first hop is:

```text
10.0.1.253
```

This confirms that the PCs are currently forwarding traffic through R1.

### Problem With This Design

If the PC uses R1's physical IP address as its default gateway and R1 fails, the PC loses its gateway even if R2 is still operational.

HSRP solves this problem by providing a **virtual gateway** shared between R1 and R2.

---

# 2. Configure HSRPv2 on R1 and R2

The objective is to configure:

- HSRP version 2.
- HSRP group 1.
- Virtual IP `10.0.1.254`.
- R1 with a priority **higher than the default 100**.
- R2 with a priority **lower than the default 100**.
- HSRP preemption enabled.

The final priority values used in this lab are:

| Router | Priority | HSRP Role |
|---|---:|---|
| R1 | `200` | Active |
| R2 | `50` | Standby |

The default HSRP priority is:

```text
100
```

Therefore:

- R1 priority `200` → preferred router.
- R2 priority `50` → backup router.

---

## Configure R1

Enter:

```cisco
R1# configure terminal
R1(config)# interface gigabitEthernet0/0
```

Enable HSRPv2:

```cisco
R1(config-if)# standby version 2
```

Configure the virtual IP:

```cisco
R1(config-if)# standby 1 ip 10.0.1.254
```

Increase R1's priority:

```cisco
R1(config-if)# standby 1 priority 200
```

Enable preemption:

```cisco
R1(config-if)# standby 1 preempt
```

Save the configuration:

```cisco
R1(config-if)# do write memory
```

---

## Configure R2

Enter:

```cisco
R2# configure terminal
R2(config)# interface gigabitEthernet0/0
```

Enable HSRPv2:

```cisco
R2(config-if)# standby version 2
```

Configure the same virtual IP:

```cisco
R2(config-if)# standby 1 ip 10.0.1.254
```

Lower R2's priority:

```cisco
R2(config-if)# standby 1 priority 50
```

Save the configuration:

```cisco
R2(config-if)# do write memory
```

R2 does not require the `preempt` command for this lab because R1 is intended to be the preferred active router.

---

# 3. Verify HSRP

On R1 and R2, use:

```cisco
show standby
```

A shortened command can also be used:

```cisco
show standby brief
```

## Expected R1 State

R1 should show:

```text
State is Active
Virtual IP address is 10.0.1.254
Priority 200
Active router is local
Standby router is 10.0.1.252
```

## Expected R2 State

R2 should show:

```text
State is Standby
Virtual IP address is 10.0.1.254
Priority 50
Active router is 10.0.1.253
Standby router is local
```

The resulting HSRP relationship is:

```text
              HSRP Group 1
             Virtual IP
             10.0.1.254
                  |
          +-------+-------+
          |               |
       R1 Active       R2 Standby
    10.0.1.253        10.0.1.252
      Priority 200      Priority 50
```

---

# 4. Configure the VIP as the PC Default Gateway

The PCs should no longer use R1's physical address:

```text
10.0.1.253
```

Instead, configure the HSRP virtual IP:

```text
10.0.1.254
```

as the default gateway.

For PC1/PC2, the final addressing should resemble:

```text
IP Address:       10.0.1.x
Subnet Mask:      255.255.255.0
Default Gateway:  10.0.1.254
```

The important change is:

```text
Before:
PC → 10.0.1.253

After:
PC → 10.0.1.254
```

The PCs now depend on the **HSRP virtual gateway**, rather than a specific physical router.

---

# 5. Test Connectivity Through the HSRP VIP

From PC1:

```text
ping 8.8.8.8
```

From PC2:

```text
ping 8.8.8.8
```

Both should successfully reach the external server.

Verify the path:

```text
tracert 8.8.8.8
```

The first hop should now be:

```text
10.0.1.254
```

Example:

```text
Tracing route to 8.8.8.8 over a maximum of 30 hops:

  1   0 ms   0 ms   0 ms   10.0.1.254
  2   0 ms   0 ms   0 ms   8.8.8.8

Trace complete.
```

The important observation is that the PC does not see R1's physical address as the gateway. It uses the HSRP VIP.

---

# 6. Examine the PC ARP Table

On the PC, run:

```text
arp -a
```

Example output:

```text
Internet Address      Physical Address      Type
10.0.1.253            00d0.585b.7501        dynamic
10.0.1.254            0000.0c9f.f001        dynamic
```

The VIP:

```text
10.0.1.254
```

is mapped to:

```text
0000.0c9f.f001
```

### HSRP Virtual MAC Address

With HSRPv2, HSRP group 1 uses:

```text
0000.0C9F.F001
```

The general HSRPv2 virtual MAC format is:

```text
0000.0C9F.Fxxx
```

where the final hexadecimal portion represents the HSRP group.

Therefore:

```text
VIP:          10.0.1.254
Virtual MAC:  0000.0C9F.F001
HSRP Group:   1
```

This is a critical concept: **the virtual IP is associated with a virtual MAC address**, allowing the active router to change without requiring the PCs to change their default gateway.

---

# 7. Shut Down R1 and Test Failover

Before shutting down R1, **save its configuration**:

```cisco
R1# write memory
```

or:

```cisco
R1# copy running-config startup-config
```

Then turn off R1 in Packet Tracer.

R1 will no longer be available to provide the HSRP active gateway.

---

## Observe R2

On R2:

```cisco
show standby
```

R2 should transition from:

```text
Standby
```

to:

```text
Active
```

The HSRP virtual IP remains:

```text
10.0.1.254
```

The PCs therefore do not need to change their gateway configuration.

---

# 8. Test Connectivity During R1 Failure

From PC1:

```text
ping 8.8.8.8
```

You may see one packet timeout during the transition while HSRP detects the failure and R2 takes over.

For example:

```text
Request timed out.

Reply from 8.8.8.8: bytes=32 time=13ms TTL=254
Reply from 8.8.8.8: bytes=32 time=12ms TTL=254
Reply from 8.8.8.8: bytes=32 time<1ms TTL=254
```

This is expected during failover.

Run:

```text
tracert 8.8.8.8
```

The first hop should still be:

```text
10.0.1.254
```

### Is R2 Used as the Default Gateway?

**Yes.**

However, the PC still uses:

```text
10.0.1.254
```

as its configured gateway.

It does **not** change the gateway to:

```text
10.0.1.252
```

Instead, R2 takes ownership of the same HSRP virtual IP and virtual MAC.

This is the main benefit of HSRP.

---

# 9. Turn R1 Back On

Turn R1 back on and allow it to finish booting.

After R1 becomes operational, check:

```cisco
R1# show standby
```

Because R1 has:

```text
Priority: 200
Preemption: enabled
```

it should reclaim the active role.

R2 has:

```text
Priority: 50
```

so it should return to the standby role.

---

# 10. Verify HSRP Preemption

After R1 has returned, check both routers.

### R1

Expected:

```text
State is Active
Priority 200
Active router is local
```

### R2

Expected:

```text
State is Standby
Priority 50
Active router is 10.0.1.253
```

### Does R1 Become Active Again?

**Yes.**

R1 becomes active again because:

1. R1 has the higher priority (`200`).
2. R1 has HSRP preemption enabled.
3. R2 has the lower priority (`50`).

Without preemption, a recovering R1 would not necessarily reclaim the active role automatically.

---

# HSRP Failover Process

The complete failover process is:

```text
                 Normal Operation

                  VIP 10.0.1.254
                         |
             +-----------+-----------+
             |                       |
       R1 — ACTIVE             R2 — STANDBY
       Priority 200             Priority 50


                  R1 Fails
                     X
                     |
                     v
             +-----------+ 
             | R2 ACTIVE|
             | Priority 50
             +-----------+
                    |
              VIP remains
              10.0.1.254


                  R1 Returns
                     |
                     v
             R1 preempts R2
                     |
             +-------+-------+
             |               |
       R1 — ACTIVE       R2 — STANDBY
       Priority 200      Priority 50
```

The PCs continue using:

```text
10.0.1.254
```

throughout the entire process.

---

# Important HSRP Concepts

## Virtual IP

The HSRP virtual IP is:

```text
10.0.1.254
```

This address is used as the default gateway by the PCs.

---

## Physical Router Addresses

R1:

```text
10.0.1.253
```

R2:

```text
10.0.1.252
```

These addresses identify the individual routers.

---

## Virtual MAC

HSRPv2 Group 1 uses:

```text
0000.0C9F.F001
```

The virtual MAC allows the active router to change while maintaining Layer 2 gateway reachability.

---

## HSRP Priority

The default priority is:

```text
100
```

This lab configures:

```text
R1 = 200
R2 = 50
```

The higher priority wins the active role.

---

## Preemption

R1 uses:

```cisco
standby 1 preempt
```

This allows R1 to automatically reclaim the active role after recovering from a failure.

---

# Verification Commands

## Check HSRP Status

```cisco
show standby
```

## Check HSRP Summary

```cisco
show standby brief
```

## Check Interface Configuration

```cisco
show running-config interface gigabitEthernet0/0
```

## Check IP Configuration

```cisco
show ip interface brief
```

## Check Routing Table

```cisco
show ip route
```

## Check PC IP Configuration

```text
ipconfig
```

## Check PC ARP Table

```text
arp -a
```

## Test Connectivity

```text
ping 8.8.8.8
```

## Verify Gateway Path

```text
tracert 8.8.8.8
```

---

# Expected Results

| Test | Expected Result |
|---|---|
| Initial PC gateway | `10.0.1.253` |
| HSRP VIP | `10.0.1.254` |
| R1 physical IP | `10.0.1.253` |
| R2 physical IP | `10.0.1.252` |
| HSRP version | Version 2 |
| HSRP group | 1 |
| R1 priority | 200 |
| R2 priority | 50 |
| Initial active router | R1 |
| Initial standby router | R2 |
| HSRP virtual MAC | `0000.0C9F.F001` |
| PC gateway after HSRP | `10.0.1.254` |
| Gateway during R1 failure | R2 |
| R1 after recovery | Active |
| R2 after R1 recovery | Standby |
| PC → 8.8.8.8 | Successful |

---

# Troubleshooting Notes

If HSRP does not form correctly, check:

```text
- Both routers are connected to the same LAN.
- Both interfaces are up/up.
- Both routers use HSRP group 1.
- Both routers use HSRPv2.
- Both routers use VIP 10.0.1.254.
- R1 and R2 are in the same subnet.
- HSRP priorities are configured correctly.
- Preemption is configured on R1.
```

Use:

```cisco
show standby
```

as the primary HSRP troubleshooting command.

If the PCs cannot reach the external server after changing the gateway, verify:

```text
1. PC default gateway = 10.0.1.254
2. R1/R2 HSRP state
3. Routing table on the active router
4. OSPF/default route
5. ARP table on the PC
6. Connectivity to 8.8.8.8
```

---

# Lab Verification Checklist

- [ ] Verify the original PC default gateway.
- [ ] Confirm the original gateway is `10.0.1.253`.
- [ ] Ping `8.8.8.8` from PC1.
- [ ] Ping `8.8.8.8` from PC2.
- [ ] Configure HSRPv2 on R1.
- [ ] Configure HSRPv2 on R2.
- [ ] Configure VIP `10.0.1.254`.
- [ ] Set R1 priority to `200`.
- [ ] Set R2 priority to `50`.
- [ ] Enable preemption on R1.
- [ ] Verify R1 is HSRP Active.
- [ ] Verify R2 is HSRP Standby.
- [ ] Change PC1/PC2 default gateway to `10.0.1.254`.
- [ ] Ping `8.8.8.8` from both PCs.
- [ ] Check the PC ARP table.
- [ ] Verify VIP maps to `0000.0C9F.F001`.
- [ ] Save R1 configuration.
- [ ] Turn off R1.
- [ ] Verify R2 becomes Active.
- [ ] Ping `8.8.8.8` during the failure.
- [ ] Verify the first traceroute hop remains `10.0.1.254`.
- [ ] Turn R1 back on.
- [ ] Verify R1 becomes Active again.
- [ ] Verify R2 returns to Standby.

---

# Key Takeaways

This lab demonstrates that HSRP provides **first-hop redundancy** by separating the PC's default gateway from a specific physical router.

Instead of configuring:

```text
PC → R1 (10.0.1.253)
```

the PCs use:

```text
PC → HSRP VIP (10.0.1.254)
              |
       +------+------+
       |             |
      R1             R2
    Active         Standby
```

When R1 fails, R2 takes over the virtual gateway. When R1 returns, its higher priority and preemption configuration allow it to become active again.

The most important observation is that **the PC's default gateway never changes**. The IP `10.0.1.254` remains the gateway throughout normal operation, failover, and recovery.