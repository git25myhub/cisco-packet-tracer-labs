# Cisco Packet Tracer – Simulation Mode & OSI Model Analysis

## Lab Objective

The objective of this lab is to use **Cisco Packet Tracer Simulation Mode** to observe and analyze network traffic as it travels through the network.

The lab focuses on:

- Using Simulation Mode.
- Observing different types of network traffic.
- Identifying the OSI model layers involved in communication.
- Generating Layer 7 traffic using DHCP.
- Analyzing DHCP traffic between a PC and the DHCP server.

---

## Lab Instructions

> **Recommendation:** If you are unfamiliar with these tasks, watch the provided YouTube demonstration before starting the lab.

### Task 1 – Analyze Network Traffic

Use **Simulation Mode** to observe the different types of traffic being sent throughout the network.

While analyzing the packets, determine:

- What type of traffic is being generated?
- Which protocols are being used?
- Which OSI model layers are involved?
- How does the packet move between devices?

---

## Task 2 – Release and Renew PC1's IP Address

On **PC1**, release its current DHCP-assigned IP address and then renew it.

This will generate DHCP traffic that can be observed using Simulation Mode.

The process should involve:

```text
PC1
 │
 │ DHCP Discover
 ▼
Network
 │
 │ DHCP Offer
 ▼
PC1
 │
 │ DHCP Request
 ▼
Network
 │
 │ DHCP ACK
 ▼
PC1
```

---

## Releasing and Renewing the IP Address

Open **PC1** in Packet Tracer.

Navigate to:

```text
Desktop
   ↓
Command Prompt
```

First, check the current IP configuration:

```text
ipconfig
```

Release the current DHCP address:

```text
ipconfig /release
```

Then request a new DHCP address:

```text
ipconfig /renew
```

Finally, verify the new configuration:

```text
ipconfig
```

You should see information such as:

- IP address
- Subnet mask
- Default gateway
- DHCP-assigned configuration

---

# Simulation Mode

Switch Packet Tracer from **Realtime** mode to:

```text
Simulation
```

Then generate the DHCP traffic by releasing and renewing PC1's address.

Use the simulation controls to step through the packets.

Pay attention to:

- Source device
- Destination device
- Protocol
- Packet type
- OSI layer information
- Encapsulation/decapsulation
- Direction of traffic

---

# DHCP Traffic Analysis

When renewing PC1's address, you should observe the DHCP process.

### 1. DHCP Discover

PC1 broadcasts a **DHCP Discover** message to locate a DHCP server.

```text
PC1 → Broadcast → DHCP Server
```

Purpose:

> "Is there a DHCP server available?"

---

### 2. DHCP Offer

The DHCP server responds with an offer.

```text
DHCP Server → PC1
```

The offer contains information such as the proposed IP address and other network configuration parameters.

---

### 3. DHCP Request

PC1 requests the offered configuration.

```text
PC1 → DHCP Server
```

---

### 4. DHCP Acknowledgment

The DHCP server confirms the assignment.

```text
DHCP Server → PC1
```

PC1 can now use the assigned IP configuration.

---

# OSI Model Analysis

When examining packets in Simulation Mode, identify the OSI layers involved.

| OSI Layer | Name | Example |
|---|---|---|
| Layer 7 | Application | DHCP |
| Layer 6 | Presentation | Data formatting |
| Layer 5 | Session | Session management |
| Layer 4 | Transport | UDP |
| Layer 3 | Network | IP |
| Layer 2 | Data Link | Ethernet |
| Layer 1 | Physical | Electrical/optical signals |

For DHCP, the most important layers to recognize are:

```text
Layer 7 – Application
        DHCP

Layer 4 – Transport
        UDP

Layer 3 – Network
        IP

Layer 2 – Data Link
        Ethernet

Layer 1 – Physical
        Physical transmission
```

---

# Important DHCP Ports

DHCP uses **UDP** at the Transport Layer.

The standard ports are:

```text
DHCP Server: UDP 67
DHCP Client: UDP 68
```

Therefore, when analyzing DHCP packets, look for:

```text
UDP
Port 67
Port 68
```

---

# Questions to Answer

## Question 1

**What layers of the OSI model are being used when analyzing the traffic?**

Answer:

The traffic can involve all layers needed to transmit the data, but the key layers visible during normal packet analysis include:

- Layer 7 – Application
- Layer 4 – Transport
- Layer 3 – Network
- Layer 2 – Data Link
- Layer 1 – Physical

---

## Question 2

**What Layer 7 protocol is generated when PC1 releases and renews its IP address?**

Answer:

```text
DHCP – Dynamic Host Configuration Protocol
```

---

## Question 3

**What Transport Layer protocol does DHCP use?**

Answer:

```text
UDP
```

---

## Question 4

**What UDP ports are used by DHCP?**

Answer:

```text
Server: UDP 67
Client: UDP 68
```

---

## Question 5

**Why does PC1 use broadcast traffic when initially requesting an IP address?**

Answer:

PC1 may not yet know the IP address of a DHCP server, so it uses broadcast communication to discover an available DHCP server on the local network.

---

# Useful Commands

Check the current IP configuration:

```text
ipconfig
```

Release the DHCP address:

```text
ipconfig /release
```

Renew the DHCP address:

```text
ipconfig /renew
```

Display detailed IP configuration:

```text
ipconfig /all
```

---

# Verification Checklist

- [ ] Packet Tracer is switched to Simulation Mode.
- [ ] Network traffic is visible in the Event List.
- [ ] Different packet types have been examined.
- [ ] OSI layers have been identified.
- [ ] PC1's current IP address has been checked.
- [ ] PC1's IP address has been released.
- [ ] PC1's IP address has been renewed.
- [ ] DHCP traffic has been captured in Simulation Mode.
- [ ] DHCP Discover has been identified.
- [ ] DHCP Offer has been identified.
- [ ] DHCP Request has been identified.
- [ ] DHCP ACK has been identified.
- [ ] UDP ports 67 and 68 have been identified.

---

# Key Learning Points

This lab demonstrates that network communication does not happen at only one OSI layer.

For example, DHCP involves:

```text
Application Layer
       ↓
      DHCP
       ↓
Transport Layer
       ↓
      UDP
       ↓
Network Layer
       ↓
       IP
       ↓
Data Link Layer
       ↓
    Ethernet
       ↓
Physical Layer
```

Using **Simulation Mode** allows you to see this process step-by-step rather than treating network communication as a single event.

---

# Conclusion

This lab provides practical experience with the **OSI model and packet analysis** in Cisco Packet Tracer.

By releasing and renewing PC1's IP address, you can observe the DHCP process and understand how a Layer 7 application protocol is carried through lower OSI layers.

The most important concepts to remember are:

```text
DHCP
 ↓
Layer 7 – Application

UDP
 ↓
Layer 4 – Transport

IP
 ↓
Layer 3 – Network

Ethernet
 ↓
Layer 2 – Data Link

Physical transmission
 ↓
Layer 1 – Physical
```

Simulation Mode is an important troubleshooting and learning tool because it allows you to follow network traffic **hop-by-hop and protocol-by-protocol**.