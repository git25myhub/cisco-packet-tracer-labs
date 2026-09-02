# Cisco Switching Lab – MAC Address Learning, ARP, and Ping

## 📌 Lab Overview

This lab demonstrates how Ethernet switches learn MAC addresses, how ARP and ICMP traffic travels through a switched network, and how switches use their MAC address tables to forward frames.

At the beginning of the lab:

- Both switches have an empty MAC address table.
- All PCs have empty ARP tables.
- Network traffic is generated using `ping`.
- The switches learn the source MAC addresses from Ethernet frames.
- The learned MAC addresses can then be viewed using Cisco IOS `show` commands.
- Finally, the dynamic MAC addresses are cleared from the switches.

---

# 🎯 Lab Objectives

By completing this lab, you will learn how to:

1. Determine what happens when one PC pings another PC on a switched network.
2. Understand the relationship between ARP, ICMP, and Ethernet frames.
3. Use Packet Tracer Simulation Mode to observe network traffic.
4. Generate traffic to populate switch MAC address tables.
5. Use Cisco IOS commands to identify learned MAC addresses.
6. Understand how switches learn MAC addresses.
7. Clear dynamically learned MAC addresses from a switch.

---

# 🖥️ Network Topology

The lab contains:

```text
PC1 ── SW1 ── SW2 ── PC3
          │      │
         PCs    PCs
```

The exact physical topology depends on the Packet Tracer file used for the lab.

The important concept is that the switches are connected to each other and the PCs communicate through the switches.

---

# 🧠 Initial Network State

At the beginning:

### Switches

Both SW1 and SW2 have empty MAC address tables.

Example:

```text
Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
```

### PCs

All PCs have empty ARP tables.

This means the PCs initially do not know the MAC address associated with the IP address of another PC on the local network.

---

# 1. What Happens When PC1 Pings PC3?

Before PC1 can send an ICMP Echo Request to PC3, PC1 needs to know PC3's MAC address.

Assuming PC1 and PC3 are on the same IP subnet, the process is:

### Step 1 – ARP Request

PC1 sends an **ARP Request** asking:

> Who has PC3's IP address?

The ARP Request is an Ethernet **broadcast**.

Destination MAC:

```text
FFFF.FFFF.FFFF
```

Because it is a broadcast, the switch receiving it floods it out all appropriate ports except the port on which it arrived.

Therefore, the ARP Request can be received by:

- SW1
- SW2
- PC2/other connected hosts
- PC3

All devices in the same Layer 2 broadcast domain receive the broadcast, although only PC3 should respond.

---

### Step 2 – ARP Reply

PC3 recognizes that the ARP Request is asking for its IP address.

PC3 sends an **ARP Reply** back to PC1.

The ARP Reply is normally a **unicast** frame addressed to PC1's MAC address.

As the switches process the frames, they learn the source MAC addresses.

---

### Step 3 – ICMP Echo Request

After PC1 learns PC3's MAC address, PC1 can send the actual ping.

The packet contains:

```text
ICMP Echo Request
```

The Ethernet destination is PC3's MAC address.

Because the switches have learned the destination MAC address, they can forward the frame specifically toward PC3 rather than flooding it.

---

### Step 4 – ICMP Echo Reply

PC3 responds with:

```text
ICMP Echo Reply
```

The switches forward the response back toward PC1.

The successful exchange is:

```text
PC1
 │
 │ ARP Request (Broadcast)
 ▼
Switches
 │
 ▼
PC3
 │
 │ ARP Reply (Unicast)
 ▼
PC1
 │
 │ ICMP Echo Request
 ▼
PC3
 │
 │ ICMP Echo Reply
 ▼
PC1
```

---

# 2. Verify Using Packet Tracer Simulation Mode

Packet Tracer's **Simulation Mode** can be used to observe the individual frames and packets.

### Procedure

1. Switch Packet Tracer from **Realtime** to **Simulation** mode.
2. From PC1, open the Command Prompt.
3. Ping PC3.

Example:

```bash
PC> ping <PC3-IP-address>
```

4. Use the **Capture/Forward** button to move through the packet exchange.
5. Observe the ARP and ICMP packets.
6. Pay attention to:
   - Source MAC address
   - Destination MAC address
   - Source IP address
   - Destination IP address
   - Broadcast vs. unicast traffic
   - Which switch ports receive and forward frames

### Expected Observation

The first ping may involve ARP before ICMP communication begins.

You should observe:

```text
ARP Request
     ↓
ARP Reply
     ↓
ICMP Echo Request
     ↓
ICMP Echo Reply
```

Subsequent pings usually do not need another ARP request because PC1 has learned PC3's MAC address.

---

# 3. Generate Traffic and Allow the Switches to Learn MAC Addresses

Use `ping` between the PCs to generate network traffic.

For example:

```bash
PC1> ping <PC3-IP>
PC2> ping <PC4-IP>
```

You can also ping other PCs on the network.

As Ethernet frames travel through the switches, the switches examine the **source MAC address** of incoming frames.

They then associate that MAC address with the interface on which the frame was received.

For example:

```text
Source MAC: 0001.647b.3119
Received on: Gig0/1
```

The switch learns:

```text
0001.647b.3119 → Gig0/1
```

This information is stored in the switch's MAC address table.

---

# 4. Identify MAC Addresses Using `show` Commands

After generating traffic, use the following command on SW1:

```bash
SW1> enable
SW1# show mac address-table
```

### SW1 Result

Your SW1 output shows:

```text
SW1#show mac address-table
          Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----

   1    0001.647b.3119    DYNAMIC     Gig0/1
   1    0004.9a6e.d870    DYNAMIC     Gig0/1
```

### SW1 Learned Addresses

| MAC Address | Type | VLAN | Port |
|---|---|---:|---|
| `0001.647b.3119` | Dynamic | 1 | Gig0/1 |
| `0004.9a6e.d870` | Dynamic | 1 | Gig0/1 |

Both addresses are learned dynamically through **GigabitEthernet0/1**.

This indicates that the devices associated with these MAC addresses are reachable through SW1's Gig0/1 interface.

---

## SW2 MAC Address Table

On SW2:

```bash
SW2> enable
SW2# show mac address-table
```

Your SW2 output shows:

```text
SW2#show mac address-table
          Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----

   1    0001.647b.3119    DYNAMIC     Fa0/2
   1    0004.9a6e.d870    DYNAMIC     Fa0/1
```

### SW2 Learned Addresses

| MAC Address | Type | VLAN | Port |
|---|---|---:|---|
| `0001.647b.3119` | Dynamic | 1 | Fa0/2 |
| `0004.9a6e.d870` | Dynamic | 1 | Fa0/1 |

SW2 has learned the same two MAC addresses but through different interfaces.

This demonstrates an important switching concept:

> **A switch learns a MAC address based on the interface where it receives a frame from that MAC address.**

---

# 5. Clear Dynamic MAC Addresses

After completing the MAC learning exercise, clear the dynamically learned MAC addresses from each switch.

On SW1:

```bash
SW1# clear mac address-table dynamic
```

On SW2:

```bash
SW2# clear mac address-table dynamic
```

Verify the table:

```bash
SW1# show mac address-table
```

and:

```bash
SW2# show mac address-table
```

The dynamically learned entries should no longer appear.

---

# 🔍 Important Commands

### Display the MAC address table

```bash
show mac address-table
```

### Clear dynamically learned MAC addresses

```bash
clear mac address-table dynamic
```

### Display a more detailed MAC address table

```bash
show mac address-table dynamic
```

### Check ARP information on a PC

On a Packet Tracer PC:

```bash
arp -a
```

---

# 🧠 Key Concepts

## MAC Address Learning

Switches learn MAC addresses automatically.

When a frame arrives, the switch examines the **source MAC address** and records:

```text
MAC Address → Incoming Interface
```

For example:

```text
0001.647b.3119 → Gig0/1
```

---

## MAC Address Table

The MAC address table tells the switch where devices can be reached.

Example:

```text
MAC Address       Port
0001.647b.3119    Gig0/1
0004.9a6e.d870    Gig0/1
```

The switch can use this information to forward frames efficiently.

---

## ARP

ARP maps an IPv4 address to a MAC address.

For example:

```text
192.168.1.3 → 0001.647b.3119
```

The ARP Request is broadcast, while the ARP Reply is normally unicast.

---

## ICMP

`ping` uses ICMP.

The two important ICMP messages are:

```text
ICMP Echo Request
ICMP Echo Reply
```

A successful ping confirms that the destination responded to the ICMP request.

---

# 📊 ARP vs ICMP vs Ethernet

| Protocol/Technology | Purpose |
|---|---|
| ARP | Finds the MAC address associated with an IPv4 address |
| Ethernet | Provides Layer 2 frame delivery |
| MAC Address Table | Allows switches to determine where to forward frames |
| ICMP | Used by `ping` to test IP connectivity |

---

# 🔄 Overall Traffic Flow

The basic communication process is:

```text
PC1 wants to ping PC3
        │
        ▼
Does PC1 know PC3's MAC?
        │
       NO
        │
        ▼
ARP Request (Broadcast)
        │
        ▼
Switch floods broadcast
        │
        ▼
PC3 receives ARP Request
        │
        ▼
ARP Reply (Unicast)
        │
        ▼
PC1 learns PC3's MAC
        │
        ▼
ICMP Echo Request
        │
        ▼
PC3
        │
        ▼
ICMP Echo Reply
        │
        ▼
PC1
```

During this process, the switches learn the source MAC addresses and add them to their MAC address tables.

---

# 📝 Lab Questions and Answers

### Question 1

**If PC1 pings PC3, what messages will be sent over the network and which devices will receive them?**

Initially, PC1 sends an **ARP Request broadcast** to discover PC3's MAC address. The broadcast is flooded through the switches and received by devices within the same Layer 2 broadcast domain.

PC3 responds with an **ARP Reply**, normally sent as a unicast to PC1.

PC1 then sends an **ICMP Echo Request** to PC3, and PC3 responds with an **ICMP Echo Reply**.

---

### Question 2

**How can this be verified?**

Use Packet Tracer's **Simulation Mode** and step through the traffic using **Capture/Forward**.

The ARP and ICMP exchanges can be observed individually.

---

### Question 3

**How do the switches learn the MAC addresses?**

Switches learn MAC addresses by examining the **source MAC address** of incoming Ethernet frames.

They associate the source MAC address with the interface on which the frame was received.

---

### Question 4

**Which command displays the learned MAC addresses?**

```bash
show mac address-table
```

---

### Question 5

**How are the learned MAC addresses classified?**

The addresses shown in the lab are:

```text
DYNAMIC
```

This means the switches learned them automatically from network traffic rather than having them manually configured.

---

### Question 6

**How do you remove dynamically learned MAC addresses?**

Use:

```bash
clear mac address-table dynamic
```

---

# ✅ Lab Verification

### SW1

```text
MAC Address       Type        Port
0001.647b.3119    DYNAMIC     Gig0/1
0004.9a6e.d870    DYNAMIC     Gig0/1
```

### SW2

```text
MAC Address       Type        Port
0001.647b.3119    DYNAMIC     Fa0/2
0004.9a6e.d870    DYNAMIC     Fa0/1
```

These results confirm that the switches successfully learned MAC addresses from network traffic.

---

# ✅ Lab Completion Checklist

- [x] Started with empty MAC address tables.
- [x] Started with empty PC ARP tables.
- [x] Predicted the traffic generated by a PC1-to-PC3 ping.
- [x] Used Packet Tracer Simulation Mode to observe traffic.
- [x] Generated network traffic using `ping`.
- [x] Allowed the switches to dynamically learn MAC addresses.
- [x] Used `show mac address-table` to verify learned addresses.
- [x] Identified the MAC addresses and associated switch ports.
- [x] Cleared dynamic MAC addresses from SW1.
- [x] Cleared dynamic MAC addresses from SW2.

---

# 📚 Conclusion

This lab demonstrates how **ARP, ICMP, Ethernet, and switch MAC address learning work together** during host-to-host communication.

The key takeaway is that a switch does not initially know where every device is located. It learns this information dynamically by examining the **source MAC addresses of frames** entering its interfaces.

Once the MAC address table is populated, the switch can forward unicast traffic directly to the correct interface instead of flooding the frame throughout the network.

The lab also demonstrates why the first ping can involve additional ARP traffic, while subsequent pings can proceed more directly because the required MAC address information has already been learned.