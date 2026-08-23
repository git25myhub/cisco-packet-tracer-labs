# Cisco Packet Tracer Lab — MAC Addresses and Layer 2 Frame Forwarding

## Lab Objective

The objective of this lab is to understand how **Ethernet source and destination MAC addresses change as a frame travels through switches and routers**.

You will use:

- Cisco IOS CLI
- Packet Tracer **Simulation Mode**
- `ping`
- ARP tables
- MAC address tables
- Interface MAC addresses

The lab focuses on an important networking concept:

> **Switches forward Ethernet frames based on MAC addresses, while routers remove the incoming Ethernet frame and create a new Ethernet frame for the next network segment.**

Because routers separate Layer 2 broadcast domains, the source and destination MAC addresses can change at every routed hop.

---

# Network Topology

The lab contains multiple switches, routers, and PCs.

The traffic paths examined in this lab are:

### PC1 → PC4

```text
PC1 → SW1 → R1 → R2 → R3 → SW2 → PC4
```

### PC1 → PC3

```text
PC1 → SW1 → PC3
```

### PC4 → PC1

```text
PC4 → SW2 → R3 → R2 → R1 → SW1 → PC1
```

> **Important:** Use the actual interface names and MAC addresses shown in your Packet Tracer topology. MAC addresses are device-specific and should not be assumed.

---

# Learning Objectives

By completing this lab, you should be able to:

- Identify source and destination MAC addresses in Ethernet frames.
- Determine the MAC address of a Cisco router interface.
- Determine the MAC address of a PC.
- Understand how switches forward frames.
- Understand how routers rewrite Layer 2 headers.
- Use ARP to determine next-hop MAC addresses.
- Use a switch MAC address table to determine forwarding behavior.
- Use Packet Tracer Simulation Mode to inspect Ethernet frames.
- Distinguish between Layer 2 and Layer 3 forwarding.

---

# Part 1 — PC1 Pings PC4

The first task is to follow the traffic from PC1 to PC4.

The path is:

```text
PC1
 ↓
SW1
 ↓
R1
 ↓
R2
 ↓
R3
 ↓
SW2
 ↓
PC4
```

You must identify the **source and destination MAC addresses** at each segment.

---

# 1. Complete the Initial Ping

Before entering Simulation Mode, perform a normal ping from PC1 to PC4.

From PC1:

```text
ping <PC4-IP-address>
```

For example:

```text
ping 192.168.4.1
```

Use the actual PC4 IP address from the topology.

This initial ping allows the devices to complete the necessary:

- ARP resolution
- MAC address learning
- Switch MAC table population

> **Important:** Do not immediately enter Simulation Mode before performing the first successful ping. The ARP process can make the frame flow harder to analyze.

---

# 2. Find the MAC Addresses

Before analyzing the frames, collect the MAC addresses of the relevant devices and interfaces.

## PC1

On PC1:

```text
ipconfig /all
```

Record the MAC address of PC1's network adapter.

Example:

```text
PC1 NIC MAC: XX:XX:XX:XX:XX:XX
```

---

## PC4

On PC4:

```text
ipconfig /all
```

Record:

```text
PC4 NIC MAC: XX:XX:XX:XX:XX:XX
```

---

# 3. Find Router Interface MAC Addresses

On Cisco routers, use:

```cisco
show interfaces
```

or:

```cisco
show interfaces gigabitEthernet 0/0
```

Look for:

```text
Hardware is ...
address is XXXX.XXXX.XXXX
```

Record the MAC address for every router interface involved in the path.

For example:

```text
R1 G0/0 MAC: XX:XX:XX:XX:XX:XX
R1 G0/1 MAC: XX:XX:XX:XX:XX:XX

R2 G0/0 MAC: XX:XX:XX:XX:XX:XX
R2 G0/1 MAC: XX:XX:XX:XX:XX:XX

R3 G0/0 MAC: XX:XX:XX:XX:XX:XX
R3 G0/1 MAC: XX:XX:XX:XX:XX:XX
```

Use the interfaces that actually participate in your topology.

---

# 4. Check ARP Tables

On the routers:

```cisco
show arp
```

On PCs:

```text
arp -a
```

ARP maps an IPv4 address to a MAC address.

For example:

```text
IP Address       MAC Address
192.168.1.254    XXXX.XXXX.XXXX
```

This helps determine which MAC address should be used as the **destination MAC for the next hop**.

---

# 5. Check the Switch MAC Address Table

On SW1:

```cisco
show mac address-table
```

On SW2:

```cisco
show mac address-table
```

You can also use:

```cisco
show mac address-table dynamic
```

The table tells you which MAC addresses have been learned on which switch ports.

Example:

```text
Vlan    Mac Address       Type       Ports
----    -----------       --------   -----
1       aaaa.bbbb.cccc    DYNAMIC    Fa0/1
1       dddd.eeee.ffff    DYNAMIC    Fa0/24
```

---

# 6. Analyze PC1 → PC4

Now determine the Ethernet source and destination MAC addresses at each point.

## A. PC1 → SW1 Segment

At the first segment, the frame originates from PC1.

Therefore:

```text
Source MAC:      PC1 NIC MAC
Destination MAC: R1 interface MAC connected to SW1
```

The destination is **not PC4's MAC address**.

PC1 is sending the frame to its default gateway, so it uses the MAC address of the router interface on the local LAN.

### Record Your Answer

```text
A. PC1 → SW1

Source MAC:
Destination MAC:
```

---

# 7. SW1 → R1 Segment

SW1 forwards the frame toward R1.

The switch does **not** change the Ethernet source or destination MAC addresses.

Therefore:

```text
Source MAC:      PC1 NIC MAC
Destination MAC: R1 interface MAC connected to SW1
```

### Record Your Answer

```text
B. SW1 → R1

Source MAC:
Destination MAC:
```

> **Key concept:** A Layer 2 switch normally forwards the Ethernet frame without changing its source or destination MAC addresses.

---

# 8. R1 → R2 Segment

When R1 receives the frame, it removes the incoming Layer 2 Ethernet header.

R1 examines the destination IP address and determines that the packet must be forwarded toward R2.

R1 then creates a **new Ethernet frame** on its outgoing interface.

Therefore:

```text
Source MAC:      R1 outgoing interface MAC
Destination MAC: R2 incoming interface MAC
```

### Record Your Answer

```text
C. R1 → R2

Source MAC:
Destination MAC:
```

---

# 9. R2 → R3 Segment

R2 receives the frame from R1.

The Ethernet header from the previous segment is removed.

R2 then creates a new Ethernet frame for the next network.

Therefore:

```text
Source MAC:      R2 outgoing interface MAC
Destination MAC: R3 incoming interface MAC
```

### Record Your Answer

```text
D. R2 → R3

Source MAC:
Destination MAC:
```

---

# 10. R3 → SW2 Segment

R3 receives the packet and determines that PC4 is on a directly connected network.

R3 uses ARP to determine PC4's MAC address.

R3 creates a new Ethernet frame:

```text
Source MAC:      R3 interface MAC connected to SW2
Destination MAC: PC4 NIC MAC
```

### Record Your Answer

```text
E. R3 → SW2

Source MAC:
Destination MAC:
```

---

# 11. SW2 → PC4 Segment

SW2 receives the frame and forwards it toward PC4.

The switch does not rewrite the Ethernet source or destination MAC addresses.

Therefore:

```text
Source MAC:      R3 interface MAC connected to SW2
Destination MAC: PC4 NIC MAC
```

### Record Your Answer

```text
F. SW2 → PC4

Source MAC:
Destination MAC:
```

---

# PC1 → PC4 Summary

Your completed table should follow this pattern:

| Segment | Source MAC | Destination MAC |
|---|---|---|
| A. PC1 → SW1 | PC1 NIC | R1 interface toward PC1 |
| B. SW1 → R1 | PC1 NIC | R1 interface toward PC1 |
| C. R1 → R2 | R1 interface toward R2 | R2 interface toward R1 |
| D. R2 → R3 | R2 interface toward R3 | R3 interface toward R2 |
| E. R3 → SW2 | R3 interface toward PC4 | PC4 NIC |
| F. SW2 → PC4 | R3 interface toward PC4 | PC4 NIC |

Replace the descriptions with the **actual MAC addresses and interface names** from your Packet Tracer topology.

---

# 12. Verify Using Simulation Mode

After completing your initial ping:

1. Switch Packet Tracer to **Simulation Mode**.
2. Clear the event list if necessary.
3. Filter the traffic to show ICMP.
4. From PC1, send a ping to PC4.
5. Step through the packets one event at a time.
6. Open the PDU details at each hop.
7. Inspect the Ethernet information.

Look for:

```text
Source MAC Address
Destination MAC Address
```

Compare what Packet Tracer displays with your predicted answers.

You should observe that the MAC addresses change whenever the packet passes through a router.

---

# Part 2 — PC1 Pings PC3

Now analyze communication between PC1 and PC3.

The path is:

```text
PC1 → SW1 → PC3
```

Because PC1 and PC3 are on the same local network, no router is required.

---

# 13. Perform the Initial Ping

From PC1:

```text
ping <PC3-IP-address>
```

For example:

```text
ping 192.168.1.3
```

Use the actual PC3 IP address.

Perform the ping once before entering Simulation Mode so that ARP and MAC learning are already complete.

---

# 14. Analyze PC1 → SW1

PC1 determines that PC3 is on the same local subnet.

Therefore, PC1 uses ARP to learn PC3's MAC address.

The Ethernet frame is:

```text
Source MAC:      PC1 NIC MAC
Destination MAC: PC3 NIC MAC
```

### Record Your Answer

```text
A. PC1 → SW1

Source MAC:
Destination MAC:
```

---

# 15. Analyze SW1 → PC3

SW1 has learned the location of PC3's MAC address.

The switch forwards the frame to the port connected to PC3.

The switch does not change the MAC addresses.

Therefore:

```text
Source MAC:      PC1 NIC MAC
Destination MAC: PC3 NIC MAC
```

### Record Your Answer

```text
B. SW1 → PC3

Source MAC:
Destination MAC:
```

---

# PC1 → PC3 Summary

| Segment | Source MAC | Destination MAC |
|---|---|---|
| A. PC1 → SW1 | PC1 NIC | PC3 NIC |
| B. SW1 → PC3 | PC1 NIC | PC3 NIC |

### Important Observation

Unlike the PC1 → PC4 path, there is **no router between PC1 and PC3**.

Therefore, the source and destination MAC addresses remain the same throughout the entire path.

---

# 16. Verify PC1 → PC3 in Simulation Mode

Switch to **Simulation Mode**.

Generate another ping:

```text
ping <PC3-IP-address>
```

Step through the packet.

Open the PDU information and inspect:

```text
Ethernet II
Source MAC Address
Destination MAC Address
```

Confirm that:

```text
PC1 MAC → PC3 MAC
```

remains unchanged from PC1 to PC3.

---

# Part 3 — PC4 Pings PC1

Now reverse the direction of the communication.

The path is:

```text
PC4 → SW2 → R3 → R2 → R1 → SW1 → PC1
```

This demonstrates that MAC addresses are rewritten at every routed hop in the reverse direction as well.

---

# 17. Perform the Initial Ping

From PC4:

```text
ping <PC1-IP-address>
```

For example:

```text
ping 192.168.1.1
```

Use the actual PC1 IP address.

Perform the initial ping before entering Simulation Mode to ensure ARP tables and switch MAC tables are populated.

---

# 18. Analyze PC4 → SW2

PC4 is sending traffic to a remote network.

Therefore, PC4 sends the frame to its default gateway, which is R3.

The Ethernet frame is:

```text
Source MAC:      PC4 NIC MAC
Destination MAC: R3 interface connected to PC4 LAN
```

### Record Your Answer

```text
A. PC4 → SW2

Source MAC:
Destination MAC:
```

---

# 19. Analyze SW2 → R3

SW2 forwards the frame to R3.

The switch does not change the Ethernet header.

Therefore:

```text
Source MAC:      PC4 NIC MAC
Destination MAC: R3 interface connected to PC4 LAN
```

### Record Your Answer

```text
B. SW2 → R3

Source MAC:
Destination MAC:
```

---

# 20. Analyze R3 → R2

R3 removes the incoming Ethernet header and creates a new frame toward R2.

Therefore:

```text
Source MAC:      R3 interface toward R2
Destination MAC: R2 interface toward R3
```

### Record Your Answer

```text
C. R3 → R2

Source MAC:
Destination MAC:
```

---

# 21. Analyze R2 → R1

R2 receives the packet and creates another Ethernet frame for the next hop.

Therefore:

```text
Source MAC:      R2 interface toward R1
Destination MAC: R1 interface toward R2
```

### Record Your Answer

```text
D. R2 → R1

Source MAC:
Destination MAC:
```

---

# 22. Analyze R1 → SW1

R1 determines that PC1 is on its directly connected LAN.

R1 uses ARP to determine PC1's MAC address.

The new Ethernet frame is:

```text
Source MAC:      R1 interface connected to SW1
Destination MAC: PC1 NIC MAC
```

### Record Your Answer

```text
E. R1 → SW1

Source MAC:
Destination MAC:
```

---

# 23. Analyze SW1 → PC1

SW1 forwards the frame to PC1.

The switch does not change the MAC addresses.

Therefore:

```text
Source MAC:      R1 interface connected to PC1 LAN
Destination MAC: PC1 NIC MAC
```

### Record Your Answer

```text
F. SW1 → PC1

Source MAC:
Destination MAC:
```

---

# PC4 → PC1 Summary

| Segment | Source MAC | Destination MAC |
|---|---|---|
| A. PC4 → SW2 | PC4 NIC | R3 interface toward PC4 |
| B. SW2 → R3 | PC4 NIC | R3 interface toward PC4 |
| C. R3 → R2 | R3 interface toward R2 | R2 interface toward R3 |
| D. R2 → R1 | R2 interface toward R1 | R1 interface toward R2 |
| E. R1 → SW1 | R1 interface toward PC1 | PC1 NIC |
| F. SW1 → PC1 | R1 interface toward PC1 | PC1 NIC |

Replace each description with the actual MAC address from your topology.

---

# 24. Verify PC4 → PC1 in Simulation Mode

Enter Simulation Mode after completing the initial ping.

Generate:

```text
PC4 → PC1
```

Step through every event and inspect the Ethernet header.

Pay particular attention to the router hops.

You should see:

```text
PC4 MAC → R3 MAC
```

then:

```text
R3 MAC → R2 MAC
```

then:

```text
R2 MAC → R1 MAC
```

and finally:

```text
R1 MAC → PC1 MAC
```

This demonstrates that the Layer 2 frame is recreated at every router hop.

---

# Important Concepts

## 1. Switches Do Not Normally Change MAC Addresses

When a switch forwards a frame:

```text
Source MAC: PC1
Destination MAC: PC3
```

the same addresses remain in the frame as it travels through the switch.

The switch simply determines which port should receive the frame.

---

## 2. Routers Change the Layer 2 Header

When a router receives an Ethernet frame, it does not simply forward that same Ethernet frame out another interface.

Instead, the router:

1. Receives the Ethernet frame.
2. Removes the Layer 2 header.
3. Examines the destination IP address.
4. Determines the next hop.
5. Creates a new Layer 2 frame.
6. Uses its outgoing interface MAC as the source.
7. Uses the next-hop device's MAC as the destination.
8. Sends the new frame.

---

## 3. IP Addresses Usually Remain End-to-End

For the ICMP packet traveling from PC1 to PC4:

```text
Source IP:      PC1
Destination IP: PC4
```

These IP addresses remain associated with the packet throughout the routed path, apart from normal IP header fields such as TTL being modified by routers.

The MAC addresses, however, are rewritten at every routed hop.

---

# Layer 2 vs Layer 3 View

For PC1 → PC4:

```text
                     Layer 2
PC1 ───── SW1 ───── R1 ───── R2 ───── R3 ───── SW2 ───── PC4
                   │         │         │
                   └─────────┴─────────┴─────────
                         Layer 3 routing
```

The important distinction is:

```text
Switch:
Forward frame based on MAC address

Router:
Route packet based on IP address
and create a new Layer 2 frame
```

---

# Useful Cisco Commands

## Display Interface MAC Addresses

```cisco
show interfaces
```

or:

```cisco
show interfaces gigabitEthernet 0/0
```

---

## Display ARP Table

```cisco
show arp
```

or:

```cisco
show ip arp
```

---

## Display MAC Address Table

```cisco
show mac address-table
```

---

## Display Interface Status

```cisco
show ip interface brief
```

---

## Test Connectivity

```cisco
ping <destination-ip>
```

---

## Trace the Layer 3 Path

From a PC:

```text
tracert <destination-ip>
```

---

# Final Answer Template

Complete the following table with the actual MAC addresses from your Packet Tracer topology.

## PC1 → PC4

| Point | Source MAC | Destination MAC |
|---|---|---|
| A. PC1 → SW1 | | |
| B. SW1 → R1 | | |
| C. R1 → R2 | | |
| D. R2 → R3 | | |
| E. R3 → SW2 | | |
| F. SW2 → PC4 | | |

## PC1 → PC3

| Point | Source MAC | Destination MAC |
|---|---|---|
| A. PC1 → SW1 | | |
| B. SW1 → PC3 | | |

## PC4 → PC1

| Point | Source MAC | Destination MAC |
|---|---|---|
| A. PC4 → SW2 | | |
| B. SW2 → R3 | | |
| C. R3 → R2 | | |
| D. R2 → R1 | | |
| E. R1 → SW1 | | |
| F. SW1 → PC1 | | |

---

# Verification Checklist

- [ ] Perform an initial PC1 → PC4 ping.
- [ ] Allow ARP to complete.
- [ ] Allow switches to learn MAC addresses.
- [ ] Record PC1's MAC address.
- [ ] Record PC4's MAC address.
- [ ] Record the MAC address of each router interface.
- [ ] Check ARP tables using `show arp`.
- [ ] Check switch MAC tables using `show mac address-table`.
- [ ] Analyze PC1 → SW1.
- [ ] Analyze SW1 → R1.
- [ ] Analyze R1 → R2.
- [ ] Analyze R2 → R3.
- [ ] Analyze R3 → SW2.
- [ ] Analyze SW2 → PC4.
- [ ] Verify PC1 → PC4 using Simulation Mode.
- [ ] Perform an initial PC1 → PC3 ping.
- [ ] Analyze PC1 → SW1.
- [ ] Analyze SW1 → PC3.
- [ ] Verify PC1 → PC3 using Simulation Mode.
- [ ] Perform an initial PC4 → PC1 ping.
- [ ] Analyze the reverse path.
- [ ] Verify PC4 → PC1 using Simulation Mode.
- [ ] Record the final MAC-address answers.

---

# Key Takeaway

The most important lesson from this lab is:

> **MAC addresses are local to each Ethernet segment. Switches forward frames without changing the MAC addresses, while routers replace the Layer 2 header when forwarding traffic between networks.**

For a routed path such as:

```text
PC1 → SW1 → R1 → R2 → R3 → SW2 → PC4
```

the MAC addresses are effectively rebuilt at each router:

```text
PC1 → R1
R1  → R2
R2  → R3
R3  → PC4
```

This is why PC1 does **not** use PC4's MAC address when sending traffic to a remote network. PC1 only needs the MAC address of its **local default gateway**.