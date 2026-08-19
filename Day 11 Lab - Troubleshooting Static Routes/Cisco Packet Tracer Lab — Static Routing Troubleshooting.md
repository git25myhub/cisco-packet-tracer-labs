# Cisco Packet Tracer Lab — Static Routing Troubleshooting

## 📌 Lab Overview

This lab focuses on **network troubleshooting and static routing**.

PC1 and PC2 were initially unable to communicate with each other. The task was to inspect the router configurations, identify **one misconfiguration on each router**, correct the errors, and verify connectivity.

The topology contains three routers:

```text id="9m7x2p"
PC1
 │
SW1
 │
R1
 │
 │ 192.168.12.0/24
 │
R2
 │
 │ 192.168.13.0/24
 │
R3
 │
SW2
 │
PC2
```

The objective was successfully completed when PC1 could ping PC2.

---

# 🎯 Lab Objectives

The main objectives were to:

- Troubleshoot failed connectivity between two PCs.
- Examine router interfaces using `show ip interface brief`.
- Examine routing tables using `show ip route`.
- Inspect static route configurations.
- Identify incorrect routing information.
- Correct the misconfiguration on R2.
- Correct the misconfiguration on R3.
- Save the corrected configurations.
- Verify end-to-end connectivity using `ping`.

---

# 🗺️ IP Addressing Plan

The topology uses three networks between the routers and two LANs.

| Device | Interface | IP Address | Subnet Mask | Purpose |
|---|---|---|---|---|
| R1 | G0/0 | `192.168.12.1` | `255.255.255.0` | R1–R2 |
| R1 | G0/1 | `192.168.1.254` | `255.255.255.0` | PC1 LAN |
| R2 | G0/0 | `192.168.12.2` | `255.255.255.0` | R1–R2 |
| R2 | G0/1 | `192.168.13.2` | `255.255.255.0` | R2–R3 |
| R3 | G0/0 | `192.168.13.3` | `255.255.255.0` | R2–R3 |
| R3 | G0/1 | `192.168.3.254` | `255.255.255.0` | PC2 LAN |

### PC Addressing

| Device | IP Address | Default Gateway |
|---|---|---|
| PC1 | `192.168.1.1` | `192.168.1.254` |
| PC2 | `192.168.3.1` | `192.168.3.254` |

---

# ❌ Initial Connectivity Problem

The initial test from PC1 was:

```text id="m4xqk8"
C:\>ping 192.168.3.1

Pinging 192.168.3.1 with 32 bytes of data:

Request timed out.
Request timed out.
Request timed out.
Request timed out.

Packets: Sent = 4, Received = 0, Lost = 4 (100% loss)
```

PC1 was able to reach its local gateway:

```text id="lq6q7s"
C:\>ping 192.168.1.254

Reply from 192.168.1.254: bytes=32 time<1ms TTL=255
Reply from 192.168.1.254: bytes=32 time<1ms TTL=255
Reply from 192.168.1.254: bytes=32 time=9ms TTL=255
Reply from 192.168.1.254: bytes=32 time<1ms TTL=255
```

This was an important troubleshooting clue.

It confirmed that:

- PC1 had a valid IP address.
- PC1's subnet mask was correct.
- PC1 could reach its default gateway.
- The problem was beyond the PC1 LAN.

---

# 🔎 Troubleshooting R1

The first step was to examine R1's interfaces:

```cisco id="j8zq7x"
R1#show ip interface brief
```

The interfaces were:

```text id="v2q0u5"
GigabitEthernet0/0     192.168.12.1    up    up
GigabitEthernet0/1     192.168.1.254   up    up
```

The R1 routing table showed:

```text id="x6v1mm"
C    192.168.1.0/24 is directly connected
S    192.168.3.0/24 [1/0] via 192.168.12.2
C    192.168.12.0/24 is directly connected
```

R1 therefore had a route to the PC2 network:

```text id="y5j8fe"
192.168.3.0/24 via 192.168.12.2
```

The next hop, `192.168.12.2`, is the R2 interface directly connected to R1.

This route is correct.

---

# 🔎 Troubleshooting R2

R2's interfaces were checked with:

```cisco id="7w3i9k"
R2#show ip interface brief
```

The result showed:

```text id="x9h2rz"
GigabitEthernet0/0     192.168.12.2    up    up
GigabitEthernet0/1     192.168.13.2    up    up
```

R2 therefore had connectivity to both neighboring routers.

However, the routing table contained:

```text id="d7p3c4"
S    192.168.1.0/24 [1/0] via 192.168.12.1
S    192.168.3.0/24 is directly connected, GigabitEthernet0/1
```

The second route was the problem.

R2 had been configured with:

```cisco id="z9q6p1"
ip route 192.168.3.0 255.255.255.0 GigabitEthernet0/1
```

This tells R2 to forward traffic for the `192.168.3.0/24` network out of G0/1 without specifying the next-hop router.

But R3 is the router responsible for the `192.168.3.0/24` LAN.

The correct next-hop address is:

```text id="f3l5td"
192.168.13.3
```

---

# 🔧 Fixing the R2 Misconfiguration

The incorrect route was removed:

```cisco id="r1d8n4"
R2(config)#no ip route 192.168.3.0 255.255.255.0 GigabitEthernet0/1
```

It was replaced with the correct static route:

```cisco id="4u2x9m"
R2(config)#ip route 192.168.3.0 255.255.255.0 192.168.13.3
```

The correct routing logic is now:

```text id="b3h4k8"
R2 → 192.168.13.3 → R3 → 192.168.3.0/24
```

---

# 🔎 Troubleshooting R3

R3 was then examined.

The R3 configuration contained the following interface:

```text id="e7w4n2"
interface GigabitEthernet0/0
 description ## to R2 ##
 ip address 192.168.23.3 255.255.255.0
```

This was incorrect.

The R2–R3 network is:

```text id="q5c8y1"
192.168.13.0/24
```

R2's address on this link is:

```text id="h6t2v9"
192.168.13.2
```

Therefore, R3 must use an address from the same subnet.

The correct R3 address is:

```text id="k8s1wp"
192.168.13.3/24
```

---

# 🔧 Fixing the R3 Misconfiguration

The incorrect address was replaced with:

```cisco id="x2v7m9"
R3(config)#interface GigabitEthernet0/0
R3(config-if)#ip address 192.168.13.3 255.255.255.0
```

After the correction, R3's routing table showed:

```text id="w4p6r2"
C    192.168.3.0/24 is directly connected, GigabitEthernet0/1
C    192.168.13.0/24 is directly connected, GigabitEthernet0/0
L    192.168.13.3/32 is directly connected, GigabitEthernet0/0
```

R3 also retained its route back to the PC1 LAN:

```text id="n7c5x4"
S    192.168.1.0/24 via 192.168.13.2
```

This provides the return path:

```text id="q9m3z6"
R3 → 192.168.13.2 → R2 → R1 → 192.168.1.0/24
```

---

# 🧭 Final Routing Path

After correcting both errors, traffic from PC1 to PC2 follows this path:

```text id="a6v3s8"
PC1
192.168.1.1
   │
   ▼
R1
192.168.1.254
   │
   │ 192.168.12.0/24
   ▼
R2
192.168.12.2
   │
   │ 192.168.13.0/24
   ▼
R3
192.168.13.3
   │
   ▼
192.168.3.254
   │
   ▼
PC2
192.168.3.1
```

The return path works in the reverse direction.

---

# 📊 Final Static Routing Configuration

## R1

R1 needs a route to the PC2 LAN:

```cisco id="g3k8y2"
ip route 192.168.3.0 255.255.255.0 192.168.12.2
```

## R2

R2 needs routes to both end networks:

```cisco id="t4q9n7"
ip route 192.168.1.0 255.255.255.0 192.168.12.1
ip route 192.168.3.0 255.255.255.0 192.168.13.3
```

## R3

R3 needs a route back to the PC1 LAN:

```cisco id="c8m2v5"
ip route 192.168.1.0 255.255.255.0 192.168.13.2
```

---

# 🧪 Connectivity Verification

After correcting the router configurations, PC1 tested connectivity to PC2 again:

```text id="p6r4x9"
C:\>ping 192.168.3.1
```

The first successful test produced:

```text id="s7k2w1"
Reply from 192.168.3.1: bytes=32 time=13ms TTL=125
Reply from 192.168.3.1: bytes=32 time<1ms TTL=125
```

A subsequent test produced:

```text id="v5n8q3"
Reply from 192.168.3.1: bytes=32 time<1ms TTL=125
Reply from 192.168.3.1: bytes=32 time=1ms TTL=125
Reply from 192.168.3.1: bytes=32 time<1ms TTL=125
Reply from 192.168.3.1: bytes=32 time<1ms TTL=125
```

Final statistics:

```text id="j2x6m8"
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

This confirms successful end-to-end connectivity.

---

# 💾 Saving the Configuration

After correcting the configurations, the changes were saved using:

```cisco id="u8q3p5"
R3#write memory
```

and the configuration returned:

```text id="w6r1z9"
Building configuration...
[OK]
```

The same approach was used for the other routers where necessary.

---

# 🧠 Troubleshooting Lessons

This lab demonstrates an important troubleshooting principle:

> **When a PC can reach its local gateway but cannot reach a remote network, investigate the routing path beyond the local LAN.**

Several commands were particularly useful:

### Check interface status

```cisco id="q4y8m2"
show ip interface brief
```

This confirms:

- IP addressing
- Interface status
- Line protocol status

### Check the routing table

```cisco id="f6k2r8"
show ip route
```

This identifies:

- Connected networks
- Static routes
- Next-hop addresses
- Missing routes
- Incorrect routes

### Check configured static routes

```cisco id="z8n3v5"
show running-config | include ip route
```

This is useful when troubleshooting static routing specifically.

### Test connectivity

```text id="r5c7x1"
ping <destination-ip>
```

---

# 🔴 Misconfigurations Found

| Router | Problem | Incorrect Configuration | Correct Configuration |
|---|---|---|---|
| **R2** | Incorrect static route | `ip route 192.168.3.0 255.255.255.0 GigabitEthernet0/1` | `ip route 192.168.3.0 255.255.255.0 192.168.13.3` |
| **R3** | Incorrect R2-facing IP address | `192.168.23.3/24` | `192.168.13.3/24` |

These two errors prevented reliable routing between the two PC LANs.

---

# ✅ Lab Outcome

The two router misconfigurations were identified and corrected.

The final routing path was:

```text id="e3j7v9"
PC1 → R1 → R2 → R3 → PC2
```

The final ping test achieved:

**4 packets sent, 4 received, 0% packet loss.**

Therefore, the lab was successfully completed.

## Key Takeaways

- A correct IP address alone does not guarantee connectivity.
- Every router must know how to reach the destination network.
- Static routes must use an appropriate next-hop address or exit interface.
- The next-hop router must be reachable through the connected network.
- Return routes are equally important.
- `show ip interface brief` and `show ip route` are essential troubleshooting commands.
- Successful local gateway connectivity does not necessarily mean remote connectivity is working.