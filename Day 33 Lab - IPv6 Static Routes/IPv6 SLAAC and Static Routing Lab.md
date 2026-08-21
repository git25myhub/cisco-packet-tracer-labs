# IPv6 SLAAC and Static Routing Lab

## Lab Overview

In this lab, IPv6 connectivity is configured across a three-router topology using **SLAAC** for end hosts and **static IPv6 routes** for inter-network communication.

The IPv6 addresses on the router interfaces have already been configured. The serial links use **link-local IPv6 addresses only**, requiring link-local next-hop addresses when configuring static routes.

The primary path between the LANs uses the direct **R1–R3 GigabitEthernet link**, while the path through **R2** is configured as a backup using a higher administrative distance.

---

## Objectives

By completing this lab, you will:

1. Enable IPv6 routing on all routers.
2. Use SLAAC to automatically configure IPv6 addresses on the PCs.
3. Identify the IPv6 addresses assigned to the PCs.
4. Configure static IPv6 routes between the LANs.
5. Configure a backup static route through R2 using administrative distance.
6. Verify IPv6 connectivity using `ping`, `tracert`, and routing-table commands.
7. Observe how IPv6 uses link-local addresses as next-hop addresses.

---

## Topology

The topology consists of:

```text
                    Serial
              FE80::/10 only
            ┌─────────────────┐
            │                 │
            ▼                 ▼
        ┌───────┐         ┌───────┐
        │  R1   │─────────│  R2   │
        └───┬───┘  Serial └───┬───┘
            │                  │
            │                  │
       G0/1 │                  │ Serial
            │                  │
            │              ┌───┴───┐
            └──────────────│  R3   │
                 G0/1      └───┬───┘
                              │
                           G0/0
                              │
                             PC2
```

### IPv6 Networks

| Network | Purpose |
|---|---|
| `2001:DB8:0:1::/64` | PC1 LAN / R1 G0/0 |
| `2001:DB8:0:13::/64` | R1–R3 GigabitEthernet link |
| `2001:DB8:0:3::/64` | PC2 LAN / R3 G0/0 |
| Serial links | Link-local IPv6 addressing only |

---

## Preconfigured Router Addresses

### R1

| Interface | IPv6 Address |
|---|---|
| G0/0 | `2001:DB8:0:1::1/64` |
| G0/1 | `2001:DB8:0:13::1/64` |
| S0/0/0 | `FE80::202:4AFF:FE23:E201` |

### R2

| Interface | IPv6 Address |
|---|---|
| S0/0/0 | `FE80::20B:BEFF:FED7:4901` |
| S0/0/1 | `FE80::20B:BEFF:FED7:4901` |

> R2 uses link-local addresses only on its serial interfaces.

### R3

| Interface | IPv6 Address |
|---|---|
| G0/0 | `2001:DB8:0:3::1/64` |
| G0/1 | `2001:DB8:0:13::2/64` |
| S0/0/0 | `FE80::290:2BFF:FECC:A101` |

---

## Part 1 — Enable IPv6 Routing

IPv6 routing must be enabled on **R1, R2, and R3**.

### R1

```cisco
enable
configure terminal
ipv6 unicast-routing
end
```

### R2

```cisco
enable
configure terminal
ipv6 unicast-routing
end
```

### R3

```cisco
enable
configure terminal
ipv6 unicast-routing
end
```

### Verification

Run:

```cisco
show running-config | include ipv6 unicast-routing
```

Expected:

```text
ipv6 unicast-routing
```

---

## Part 2 — Configure the PCs Using SLAAC

IPv6 hosts can automatically configure their global IPv6 addresses using **SLAAC (Stateless Address Autoconfiguration)**.

On each PC:

1. Open **Desktop**.
2. Select **IP Configuration**.
3. Select **IPv6**.
4. Select **Auto Config**.

The PC should receive:

- A global IPv6 address based on the advertised `/64` prefix.
- A link-local address.
- The router's link-local address as the default gateway.

### PC2 Address

The lab output shows PC2 received:

```text
IPv6 Address:
2001:DB8:0:3:240:BFF:FE69:9B18
```

Link-local address:

```text
FE80::240:BFF:FE69:9B18
```

Default gateway:

```text
FE80::290:2BFF:FECC:A101
```

This demonstrates SLAAC using the network prefix:

```text
2001:DB8:0:3::/64
```

and an automatically generated interface ID.

### Record the PC Addresses

Use the PC command prompt:

```text
ipconfig
```

Record the IPv6 addresses:

| Device | IPv6 Address | Default Gateway |
|---|---|---|
| PC1 | `____________________________` | `____________________________` |
| PC2 | `2001:DB8:0:3:240:BFF:FE69:9B18` | `FE80::290:2BFF:FECC:A101` |

---

## Part 3 — Configure Static IPv6 Routes

The goal is to allow:

```text
PC1  <------>  PC2
```

to communicate.

There are two possible paths:

### Primary Path

```text
PC1 → R1 → R3 → PC2
```

This path uses the direct R1–R3 GigabitEthernet connection.

### Backup Path

```text
PC1 → R1 → R2 → R3 → PC2
```

The R2 path should only be used when the primary R1–R3 path is unavailable.

To accomplish this, configure the backup static routes with an **administrative distance of 5**.

---

## R1 Static Routes

### Primary Route

R1 needs a route to the PC2 LAN:

```cisco
ipv6 route 2001:DB8:0:3::/64 GigabitEthernet0/1 2001:DB8:0:13::2
```

This sends traffic directly to R3.

### Backup Route

Configure the R2 path with a higher administrative distance:

```cisco
ipv6 route 2001:DB8:0:3::/64 Serial0/0/0 FE80::20B:BEFF:FED7:4901 5
```

The `5` at the end gives this route an administrative distance of 5.

---

## R2 Static Routes

R2 requires routes to both LANs.

### Route to PC1 LAN

```cisco
ipv6 route 2001:DB8:0:1::/64 Serial0/0/0 FE80::202:4AFF:FE23:E201
```

### Route to PC2 LAN

```cisco
ipv6 route 2001:DB8:0:3::/64 Serial0/0/1 FE80::290:2BFF:FECC:A101
```

Because the serial links use only link-local addresses, the link-local next-hop address must be specified.

---

## R3 Static Routes

### Primary Route

R3 needs a route to the PC1 LAN through R1:

```cisco
ipv6 route 2001:DB8:0:1::/64 GigabitEthernet0/1 2001:DB8:0:13::1
```

### Backup Route

The serial path through R2 is configured with administrative distance 5:

```cisco
ipv6 route 2001:DB8:0:1::/64 Serial0/0/0 FE80::20B:BEFF:FED7:4901 5
```

---

## Why the Backup Route Works

The routing table on R1 should contain two routes to:

```text
2001:DB8:0:3::/64
```

The primary route has the default administrative distance:

```text
[1/0]
```

The backup route has:

```text
[5/0]
```

Because **lower administrative distance is preferred**, R1 selects the direct R1–R3 route under normal conditions.

If the direct route disappears, the route with administrative distance 5 becomes the preferred available route.

The same principle is used on R3 for reaching:

```text
2001:DB8:0:1::/64
```

---

## Verify IPv6 Routing

On each router, use:

```cisco
show ipv6 route
```

On R1, the expected static routes include:

```text
S   2001:DB8:0:3::/64 [1/0]
    via 2001:DB8:0:13::2, GigabitEthernet0/1
```

and the backup route:

```text
S   2001:DB8:0:3::/64 [5/0]
    via FE80::20B:BEFF:FED7:4901, Serial0/0/0
```

The important point is that the `[1/0]` route is preferred over `[5/0]`.

---

## Verify Interfaces

Run:

```cisco
show ipv6 interface brief
```

Example R1 output:

```text
GigabitEthernet0/0    [up/up]
    FE80::202:4AFF:FE23:E201
    2001:DB8:0:1::1

GigabitEthernet0/1    [up/up]
    FE80::202:4AFF:FE23:E202
    2001:DB8:0:13::1

Serial0/0/0           [up/up]
    FE80::202:4AFF:FE23:E201
```

Check that the required interfaces are:

```text
[up/up]
```

---

## Connectivity Testing

From PC1, ping the IPv6 address of PC2:

```text
ping 2001:DB8:0:3:240:BFF:FE69:9B18
```

Expected result:

```text
Reply from 2001:DB8:0:3:240:BFF:FE69:9B18
```

A successful test should result in:

```text
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

---

## Trace the Primary Path

From PC1:

```text
tracert 2001:DB8:0:3:240:BFF:FE69:9B18
```

Under normal conditions, the path should be:

```text
1   2001:DB8:0:1::1
2   2001:DB8:0:13::2
3   2001:DB8:0:3:240:BFF:FE69:9B18
```

This confirms that traffic is using:

```text
PC1 → R1 → R3 → PC2
```

instead of the R2 backup path.

---

## Test the Backup Path

To verify the backup route, temporarily disable the primary R1–R3 link.

On R1:

```cisco
configure terminal
interface GigabitEthernet0/1
shutdown
end
```

Then test connectivity again from PC1:

```text
ping 2001:DB8:0:3:240:BFF:FE69:9B18
```

Run:

```text
tracert 2001:DB8:0:3:240:BFF:FE69:9B18
```

The path should now use R2:

```text
PC1 → R1 → R2 → R3 → PC2
```

The trace may show:

```text
1   2001:DB8:0:1::1
2   * * *
3   2001:DB8:0:3::1
4   PC2
```

The `* * *` does not necessarily indicate a routing failure. A router may simply not respond to the traceroute probes while still forwarding the traffic correctly.

After testing, restore the primary link:

```cisco
configure terminal
interface GigabitEthernet0/1
no shutdown
end
```

Verify that the primary route is installed again:

```cisco
show ipv6 route
```

---

## Important IPv6 Concepts

### 1. IPv6 Routing Must Be Enabled

The command:

```cisco
ipv6 unicast-routing
```

allows the router to forward IPv6 packets between interfaces.

Without it, the router can have IPv6 addresses but will not perform IPv6 routing.

### 2. SLAAC

SLAAC allows hosts to automatically generate their IPv6 addresses from router advertisements.

The router advertises the network prefix, such as:

```text
2001:DB8:0:3::/64
```

The PC then generates its interface ID.

### 3. Link-Local Next Hops

IPv6 routers can use link-local addresses as next-hop addresses.

For example:

```cisco
ipv6 route 2001:DB8:0:3::/64 Serial0/0/0 FE80::20B:BEFF:FED7:4901
```

The `FE80::...` address identifies the next-hop router on the directly connected serial link.

### 4. Administrative Distance

Administrative distance determines which route is preferred when multiple routes exist.

In this lab:

```text
Primary route:  AD 1
Backup route:   AD 5
```

Therefore:

```text
1 < 5
```

and the primary route is preferred.

---

## Useful Verification Commands

### Show IPv6 interfaces

```cisco
show ipv6 interface brief
```

### Show IPv6 routing table

```cisco
show ipv6 route
```

### Show static IPv6 routes in the configuration

```cisco
show running-config | include ipv6 route
```

### Test router-to-router connectivity

```cisco
ping ipv6 2001:DB8:0:13::2
```

### Test host connectivity

```text
ping <PC2-IPv6-address>
```

### Trace the IPv6 path

```text
tracert <PC2-IPv6-address>
```

### Save the configuration

```cisco
write memory
```

or:

```cisco
copy running-config startup-config
```

---

## Verification Checklist

- [ ] IPv6 routing is enabled on R1.
- [ ] IPv6 routing is enabled on R2.
- [ ] IPv6 routing is enabled on R3.
- [ ] PC1 has obtained an IPv6 address using SLAAC.
- [ ] PC2 has obtained an IPv6 address using SLAAC.
- [ ] PC default gateways are automatically configured.
- [ ] R1 has a primary static route to the PC2 LAN.
- [ ] R1 has a backup static route through R2.
- [ ] R2 has routes to both LANs.
- [ ] R3 has a primary static route to the PC1 LAN.
- [ ] R3 has a backup static route through R2.
- [ ] The primary route has a lower administrative distance.
- [ ] PC1 can ping PC2.
- [ ] The normal traceroute uses R1 → R3.
- [ ] The backup traceroute uses R1 → R2 → R3 after the primary link fails.
- [ ] The primary link is restored after testing.
- [ ] Configurations are saved.

---

## Expected Final Result

The completed topology should provide reliable IPv6 connectivity between PC1 and PC2.

Under normal conditions:

```text
PC1
 │
 ▼
 R1
 │
 │ Primary
 ▼
 R3
 │
 ▼
PC2
```

If the direct R1–R3 connection fails:

```text
PC1
 │
 ▼
 R1
 │
 ▼
 R2
 │
 ▼
 R3
 │
 ▼
PC2
```

This lab demonstrates **IPv6 SLAAC, link-local next-hop addressing, static routing, administrative distance, route preference, and IPv6 failover**.