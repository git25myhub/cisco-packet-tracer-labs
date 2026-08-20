# Floating Static Routes and OSPF Failover Lab

## Overview

This lab demonstrates how dynamic routing and floating static routes can be used together to provide a backup path when a primary routing link fails.

The primary path between **R1 and R2** uses **OSPF**, while floating static routes with an administrative distance of **111** are configured as backup routes. Because OSPF has an administrative distance of **110**, the floating static routes remain inactive while the OSPF routes are available.

When the R1-R2 link fails, the OSPF routes are removed from the routing tables and the floating static routes become active.

---

## Lab Objectives

1. Examine the routing tables of R1 and R2.
2. Identify the dynamic routing protocol being used.
3. Determine which routes are used to reach SRV1 and the Internet server `1.1.1.1`.
4. Test connectivity to SRV1 and `1.1.1.1`.
5. Configure floating static routes on R1 and R2.
6. Verify whether the floating static routes enter the routing tables while OSPF is still operational.
7. Shut down the R1-R2 link.
8. Verify that the floating static routes become active.
9. Confirm connectivity from PC1 to SRV1 after the primary link fails.

---

# 1. Examine the Routing Tables

## R1 Routing Table

The initial routing table on R1 shows an OSPF route to the `10.0.2.0/24` network:

```text
O       10.0.2.0/24 [110/2] via 10.0.0.2
```

This tells us that R1 learns the network behind R2 dynamically through **OSPF**.

R1 also has a default static route:

```text
S*      0.0.0.0/0 [1/0] via 203.0.113.9
```

Therefore, traffic destined for networks that do not have a more specific route uses the default route toward the ISP.

---

## R2 Routing Table

R2 similarly learns the `10.0.1.0/24` network through OSPF:

```text
O       10.0.1.0/24 [110/2] via 10.0.0.1
```

R2 also has a default route toward the ISP:

```text
S*      0.0.0.0/0 [1/0] via 203.0.113.13
```

---

## Dynamic Routing Protocol

The dynamic routing protocol used by Enterprise A is:

```text
OSPF
```

This is confirmed by the routing table entries marked with `O` and the router configurations containing:

```cisco
router ospf 1
 network 10.0.0.0 0.255.255.255 area 0
```

---

# 2. Which Route Does PC1 Use to Reach SRV1?

PC1 is located in the `10.0.1.0/24` network, while SRV1 is located in the `10.0.2.0/24` network.

The initial path is:

```text
PC1
  |
  | 10.0.1.0/24
  v
R1
  |
  | OSPF
  | 10.0.0.0/30
  v
R2
  |
  | 10.0.2.0/24
  v
SRV1
```

R1 therefore uses the OSPF route:

```text
O 10.0.2.0/24 [110/2] via 10.0.0.2
```

The administrative distance of OSPF is **110**.

---

# 3. Which Route Does PC1 Use to Reach 1.1.1.1?

The routing table does not contain a specific route to `1.1.1.1`.

Instead, R1 uses its default route:

```text
S* 0.0.0.0/0 [1/0] via 203.0.113.9
```

Therefore, the path is:

```text
PC1
  |
  v
R1
  |
  | Default route
  | 203.0.113.9
  v
ISPBR1
  |
  v
1.1.1.1
```

The ISP router has `1.1.1.1/32` configured on Loopback0:

```text
Loopback0
 ip address 1.1.1.1 255.255.255.255
```

---

# 4. Initial Connectivity Tests

## Ping SRV1

PC1 was able to reach SRV1 at `10.0.2.1`:

```text
C:\>ping 10.0.2.1

Reply from 10.0.2.1: bytes=32 time=1ms TTL=126
Reply from 10.0.2.1: bytes=32 time=1ms TTL=126
Reply from 10.0.2.1: bytes=32 time<1ms TTL=126
```

The result was:

```text
Packets: Sent = 4, Received = 3, Lost = 1 (25% loss)
```

The first packet timed out, followed by three successful replies.

---

## Ping Internet Server

PC1 was also able to reach `1.1.1.1`:

```text
C:\>ping 1.1.1.1

Reply from 1.1.1.1: bytes=32 time<1ms TTL=254
Reply from 1.1.1.1: bytes=32 time<1ms TTL=254
Reply from 1.1.1.1: bytes=32 time=11ms TTL=254
```

The result was:

```text
Packets: Sent = 4, Received = 3, Lost = 1 (25% loss)
```

This confirms that the default route toward the ISP was functioning.

---

# 5. Configure Floating Static Routes

A floating static route is a static route with a higher administrative distance than the primary dynamic route.

Since OSPF uses an administrative distance of **110**, the backup static routes were configured with an administrative distance of **111**.

This makes them less preferred than OSPF.

---

## Floating Static Route on R1

R1 was configured with:

```cisco
ip route 10.0.2.0 255.255.255.0 203.0.113.1 111
```

The route points toward `203.0.113.1` as the backup next hop.

The important part is the administrative distance:

```text
111
```

Since:

```text
OSPF = 110
Floating static = 111
```

OSPF remains preferred while the R1-R2 link is operational.

The configured route was saved to the running configuration:

```text
ip route 10.0.2.0 255.255.255.0 203.0.113.1 111
```

---

## Floating Static Route on R2

R2 was configured with the corresponding backup route:

```cisco
ip route 10.0.1.0 255.255.255.0 203.0.113.5 111
```

This provides a backup path toward the `10.0.1.0/24` network.

---

# 6. Do the Floating Static Routes Enter the Routing Tables Initially?

**No.**

While the R1-R2 link is operational, the OSPF routes remain installed because OSPF has a lower administrative distance.

For example, R1 has:

```text
O 10.0.2.0/24 [110/2] via 10.0.0.2
```

The floating static route has:

```text
S 10.0.2.0/24 [111/0] via 203.0.113.1
```

Because **110 < 111**, OSPF wins.

The same logic applies on R2:

```text
OSPF = 110
Floating static = 111
```

Therefore, the floating static routes are configured but are not installed in the routing table while the OSPF routes are available.

---

# 7. Simulate Failure of the R1-R2 Link

The R1-R2 connection uses:

```text
GigabitEthernet0/2/0
```

The interface was shut down:

```cisco
interface GigabitEthernet0/2/0
 shutdown
```

The shutdown generated an OSPF adjacency failure:

```text
%OSPF-5-ADJCHG:
Nbr ... from FULL to DOWN,
Neighbor Down: Interface down or detached
```

This causes the OSPF route between the two enterprise networks to be removed.

---

# 8. Floating Static Route Becomes Active

After the R1-R2 link was shut down, R1's routing table changed.

The OSPF route disappeared:

```text
O 10.0.2.0/24 via 10.0.0.2
```

and the floating static route became active:

```text
S 10.0.2.0/24 [111/0] via 203.0.113.1
```

This demonstrates the purpose of the floating static route.

R1 automatically switched from the primary OSPF path to the backup static path.

---

## R2 After Failure

R2 experienced the same behavior.

The OSPF route to `10.0.1.0/24` disappeared, and the floating static route became active:

```text
S 10.0.1.0/24 [111/0] via 203.0.113.5
```

Therefore, both routers had a backup route available when the primary R1-R2 connection failed.

---

# 9. Verify Connectivity After Failure

After the primary link was shut down, PC1 initially experienced packet loss:

```text
C:\>ping 10.0.2.1

Request timed out.
Request timed out.
Request timed out.
Reply from 10.0.2.1: bytes=32 time=10ms TTL=124
```

This produced:

```text
Packets: Sent = 4, Received = 1, Lost = 3 (75% loss)
```

The packet loss occurred during the routing transition as the network moved from the OSPF path to the floating static path.

A subsequent test showed full connectivity:

```text
C:\>ping 10.0.2.1

Reply from 10.0.2.1: bytes=32 time<1ms TTL=124
Reply from 10.0.2.1: bytes=32 time<1ms TTL=124
Reply from 10.0.2.1: bytes=32 time<1ms TTL=124
Reply from 10.0.2.1: bytes=32 time=10ms TTL=124
```

Final result:

```text
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

This confirms that the floating static route successfully maintained connectivity to SRV1 after the primary OSPF link failed.

---

# 10. Traceroute After Failover

The traceroute to SRV1 showed the backup path:

```text
C:\>tracert 10.0.2.1

1    10.0.1.254
2    203.0.113.1
3    192.168.1.2
4    203.0.113.6
5    10.0.2.1
```

The important observation is that traffic no longer travels directly from R1 to R2 over the `10.0.0.0/30` link.

Instead, it follows the backup path through the external/ISP-facing network:

```text
R1
 |
 | 203.0.113.1
 v
... intermediate network ...
 |
 | 203.0.113.6
 v
R2
 |
 v
SRV1
```

This confirms that the floating static route is being used after the OSPF path becomes unavailable.

---

# Routing Preference

The lab demonstrates the following route-selection behavior:

```text
Primary:
OSPF
Administrative Distance = 110

Backup:
Floating Static Route
Administrative Distance = 111
```

Therefore:

```text
Normal operation
       |
       v
OSPF AD 110
       |
       v
R1 ───────── R2
```

When the R1-R2 link fails:

```text
OSPF route removed
       |
       v
Floating static AD 111
       |
       v
R1 ── ISP/alternate path ── R2
```

---

# Final Configuration

## R1

```cisco
router ospf 1
 network 10.0.0.0 0.255.255.255 area 0

ip route 0.0.0.0 0.0.0.0 203.0.113.9
ip route 10.0.2.0 255.255.255.0 203.0.113.1 111
```

## R2

```cisco
router ospf 1
 network 10.0.0.0 0.255.255.255 area 0

ip route 0.0.0.0 0.0.0.0 203.0.113.13
ip route 10.0.1.0 255.255.255.0 203.0.113.5 111
```

---

# Verification Commands

Useful commands for this lab include:

### View the routing table

```cisco
show ip route
```

### View OSPF configuration

```cisco
show running-config | section router ospf
```

### View OSPF neighbors

```cisco
show ip ospf neighbor
```

### Verify interface status

```cisco
show ip interface brief
```

### Test connectivity

```cisco
ping 10.0.2.1
ping 1.1.1.1
```

### Trace the path

```cisco
tracert 10.0.2.1
```

---

# Results Summary

| Test/Question | Result |
|---|---|
| Dynamic routing protocol | **OSPF** |
| Primary route from R1 to SRV1 | **OSPF, AD 110** |
| Internet route from PC1 | **Default static route via 203.0.113.9** |
| Floating route R1 | `10.0.2.0/24 via 203.0.113.1`, AD 111 |
| Floating route R2 | `10.0.1.0/24 via 203.0.113.5`, AD 111 |
| Floating routes installed initially? | **No** |
| Why? | OSPF AD 110 is preferred over static AD 111 |
| After R1-R2 failure | **Floating static routes become active** |
| PC1 → SRV1 after failover | **Successful** |
| Final ping result | **4/4, 0% loss** |
| Failover path | Via alternate/ISP-facing network |

---

# Key Takeaways

- **OSPF** provides the primary dynamic route between R1 and R2.
- A normal static route has a lower administrative distance than OSPF, but a **floating static route** is deliberately configured with a higher administrative distance.
- The value **111** makes the static route less preferred than OSPF's **110**.
- The floating static route remains inactive while the OSPF route is available.
- When the R1-R2 link fails, the OSPF adjacency goes down and its route is removed.
- The floating static route is then installed automatically.
- PC1 can continue reaching SRV1 through the backup path.
- The final `ping` confirmed **0% packet loss** after failover.