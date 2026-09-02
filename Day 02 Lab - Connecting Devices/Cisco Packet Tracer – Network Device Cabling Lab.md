# Cisco Packet Tracer – Network Device Cabling Lab

## Lab Objective

The objective of this lab is to connect network devices together using the **correct cable types** based on the device interfaces and connection labels.

For this practice, assume that **Auto MDI-X is disabled or not supported**. This means that you must manually select the appropriate cable type for each connection.

The lab also provides practice in identifying when to use **copper Ethernet, serial, and fiber-optic connections**.

---

## Lab Instructions

1. Connect all network devices according to the labels provided in the topology.
2. Select the appropriate cable type for every connection.
3. Assume **Auto MDI-X is disabled/not supported**.
4. Verify that the correct interfaces are connected.
5. Check that the physical link comes up successfully.
6. Remember that Packet Tracer does not distinguish between **single-mode and multimode fiber**, but you should still consider which type would be appropriate for the connection.

---

## Cable Types

### 1. Copper Straight-Through

Use a **Copper Straight-Through** cable when connecting different types of Ethernet devices.

Typical examples:

- PC → Switch
- Router → Switch
- Access Point → Switch
- Server → Switch

**Example:**

```text
PC ───────── Straight-Through ───────── Switch
```

---

### 2. Copper Crossover

Use a **Copper Crossover** cable when connecting similar Ethernet device types, particularly when Auto MDI-X is disabled.

Typical examples:

- Switch → Switch
- Router → Router
- PC → PC
- Router → PC

**Example:**

```text
Switch ───────── Crossover ───────── Switch
```

> Modern devices often support Auto MDI-X, which can automatically correct transmit/receive pairs. For this lab, assume that feature is unavailable.

---

### 3. Fiber-Optic Cable

Use **Fiber-Optic** when the labeled interfaces require a fiber connection.

Fiber is commonly used for:

- Switch → Switch uplinks
- Switch → Router uplinks
- Long-distance network connections
- High-speed backbone connections

Packet Tracer does not differentiate between single-mode and multimode fiber, but in a real network:

- **Multimode fiber (MMF)** is commonly used for shorter distances, such as within buildings or data centers.
- **Single-mode fiber (SMF)** is normally used for longer distances, such as between buildings or over carrier networks.

**Example:**

```text
Switch ───────── Fiber ───────── Switch
```

---

### 4. Serial Cable

Serial connections are commonly used for **WAN links between routers** in Packet Tracer.

Depending on the topology, you may need:

- Serial DCE
- Serial DTE

The **DCE side** normally provides clocking for the serial connection.

**Example:**

```text
Router ───── Serial WAN Link ───── Router
```

---

## Cable Selection Quick Reference

| Connection | Cable |
|---|---|
| PC → Switch | Copper Straight-Through |
| PC → Router | Copper Crossover |
| Switch → Switch | Copper Crossover |
| Router → Router | Copper Crossover |
| Switch → Router | Copper Straight-Through |
| PC → PC | Copper Crossover |
| Switch → Switch (Fiber interfaces) | Fiber |
| Router → Router (Serial interfaces) | Serial DCE/DTE |

> Always prioritize the **interface labels and requirements in the topology** over this general reference table.

---

## Physical Layer Verification

After connecting the devices, verify the physical links.

### Check Link Lights

Look at the interfaces in Packet Tracer.

A successful physical connection should eventually show an active link.

Typically:

```text
Green = Link is operational
Amber = Interface is initializing or experiencing a transition
Red/Down = Link is not operational
```

Allow a few seconds for interfaces to transition to an operational state.

---

## Troubleshooting

If a link does not come up, check the following:

### 1. Incorrect Cable

Make sure you selected the correct cable.

For example:

```text
Switch ─ Switch
```

With Auto MDI-X disabled, this normally requires:

```text
Copper Crossover
```

not:

```text
Copper Straight-Through
```

---

### 2. Incorrect Interface

Check that the cable is connected to the interfaces specified by the topology labels.

For example:

```text
R1 G0/0 ───── SW1 G0/1
```

Make sure the cable is actually connected to:

- R1 `G0/0`
- SW1 `G0/1`

---

### 3. Interface Shutdown

A router interface may be administratively down.

Check it with:

```text
show ip interface brief
```

If required, enter:

```text
enable
configure terminal
interface gigabitEthernet 0/0
no shutdown
```

---

### 4. Fiber Connection

If a fiber link is not working:

- Confirm that both interfaces are fiber-capable.
- Confirm that the correct fiber cable is selected.
- Verify that the interfaces are enabled.

---

### 5. Serial Connection

For serial links, verify that:

- The correct DCE/DTE ends are used.
- The interfaces are enabled.
- The DCE side has appropriate clocking if required.

You can identify the DCE side with:

```text
show controllers serial 0/0/0
```

---

## Verification Checklist

Before considering the lab complete, verify:

- [ ] All devices are connected according to the topology labels.
- [ ] Correct cable types are used.
- [ ] Straight-through cables are used where appropriate.
- [ ] Crossover cables are used where appropriate.
- [ ] Fiber connections use fiber-capable interfaces.
- [ ] Serial connections use the correct serial cables.
- [ ] Interfaces are not administratively shut down.
- [ ] Physical link indicators are operational.
- [ ] No cables are connected to the wrong interfaces.

---

## Key Learning Points

This lab reinforces the following networking concepts:

1. Identifying different network interface types.
2. Selecting the correct physical cable.
3. Understanding straight-through vs. crossover Ethernet cables.
4. Understanding fiber-optic connections.
5. Understanding DCE and DTE serial connections.
6. Troubleshooting Layer 1 connectivity.
7. Understanding the importance of interface compatibility.
8. Recognizing that Auto MDI-X can affect cable selection.

---

## Conclusion

The main goal of this lab is to develop confidence in **Layer 1 network connectivity**.

Before troubleshooting IP addresses, routing, VLANs, or other higher-layer technologies, always verify the physical connection first:

```text
Device
   ↓
Correct Interface
   ↓
Correct Cable
   ↓
Physical Link
   ↓
Interface Status
   ↓
Layer 2/Layer 3 Configuration
```

A correctly designed network begins with a correctly connected physical layer.