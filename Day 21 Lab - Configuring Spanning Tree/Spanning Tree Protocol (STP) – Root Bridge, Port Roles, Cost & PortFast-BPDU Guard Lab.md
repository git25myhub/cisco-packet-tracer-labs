# Spanning Tree Protocol (STP) – Root Bridge, Port Roles, Cost & PortFast/BPDU Guard Lab

## Lab Overview

In this lab, you will analyze and manipulate the **Spanning Tree Protocol (STP)** topology using Cisco IOS commands. You will identify the current root bridge and port roles, configure different root bridges for VLAN 1 and VLAN 2, modify STP path costs and port priorities, and configure PortFast and BPDU Guard.

The lab uses **PVST (Per-VLAN Spanning Tree)**, allowing VLAN 1 and VLAN 2 to have different root bridges and STP topologies.

> **Important:** Turn off link lights in Packet Tracer for this lab:  
> **Options > Preferences > Show Link Lights**

---

## Objectives

By the end of this lab, you should be able to:

- Identify the STP root bridge.
- Identify Root, Designated, and Alternate port roles.
- Interpret STP forwarding and blocking states.
- Configure a switch as the primary or secondary root bridge.
- Modify the STP path cost of an interface.
- Modify the STP port priority of an interface.
- Explain how STP selects a root port.
- Configure PortFast on access ports.
- Configure BPDU Guard on edge/access ports.
- Verify STP configuration and operation using the CLI.

---

## Topology

The lab contains four switches:

- **SW1**
- **SW2**
- **SW3**
- **SW4**

The switches are connected using trunk links. VLAN 1 and VLAN 2 are used to demonstrate how PVST can produce different STP topologies for different VLANs.

---

# Part 1 – Identify the Current STP Topology

Use the CLI to examine the current STP topology before making any configuration changes.

### Commands

On each switch, use:

```text
enable
show spanning-tree
```

You can also inspect a specific VLAN:

```text
show spanning-tree vlan 1
show spanning-tree vlan 2
```

### Questions

1. What is the current **root bridge** for VLAN 1?

2. What is the current **root bridge** for VLAN 2?

3. What is the STP role and state of each port on each switch?

Record your results in a table similar to:

| Switch | Port | VLAN | STP Role | State |
|---|---|---|---|---|
| SW1 | Fa0/1 | 1 | | |
| SW1 | Fa0/2 | 1 | | |
| SW1 | Fa0/3 | 1 | | |
| SW2 | Fa0/1 | 1 | | |
| SW2 | Fa0/2 | 1 | | |
| SW2 | Fa0/3 | 1 | | |
| SW3 | Fa0/1 | 1 | | |
| SW3 | Fa0/2 | 1 | | |
| SW3 | Fa0/3 | 1 | | |
| SW4 | Fa0/1 | 1 | | |
| SW4 | Fa0/2 | 1 | | |

Repeat the analysis for VLAN 2 where applicable.

### STP Roles

Pay attention to these roles:

- **Root** – the switch's best path toward the root bridge.
- **Designated** – the forwarding port selected for a network segment.
- **Alternate** – a backup path that is placed into a blocking state.

Common states include:

- **FWD** – Forwarding
- **BLK** – Blocking
- **LRN** – Learning

---

# Part 2 – Configure the Root Bridges

Configure **SW1** as:

- Primary root for **VLAN 1**
- Secondary root for **VLAN 2**

Configure **SW2** as:

- Primary root for **VLAN 2**
- Secondary root for **VLAN 1**

This creates a load-balanced STP design where different switches are preferred as the root for different VLANs.

---

## SW1 Configuration

```text
enable
configure terminal

spanning-tree vlan 1 root primary
spanning-tree vlan 2 root secondary

end
write memory
```

Verify:

```text
show spanning-tree vlan 1
show spanning-tree vlan 2
```

---

## SW2 Configuration

```text
enable
configure terminal

spanning-tree vlan 1 root secondary
spanning-tree vlan 2 root primary

end
write memory
```

Verify:

```text
show spanning-tree vlan 1
show spanning-tree vlan 2
```

### Questions

1. Which switch is now the root bridge for VLAN 1?

2. Which switch is now the root bridge for VLAN 2?

3. What is the STP role/state of each port on each switch now?

4. Compare the topology before and after changing the root bridge.

Record your results.

---

# Part 3 – Change the STP Cost on SW4

Increase the **VLAN 1 STP cost** of SW4's **Fa0/2** interface to `100`.

### Configuration

On SW4:

```text
enable
configure terminal

interface fastethernet 0/2
spanning-tree vlan 1 cost 100

end
write memory
```

Verify:

```text
show spanning-tree vlan 1
```

You should see the increased cost on Fa0/2.

### Question

**Does SW4 select a different root port? Why or why not?**

Consider the following when answering:

- The cost of the available paths to the root bridge.
- The original cost of Fa0/1.
- The new cost of Fa0/2.
- STP's process for selecting the lowest-cost path.

In the observed topology, SW4 selects **Fa0/1 as the Root Port**, while Fa0/2 becomes an **Alternate/Blocking** port with a cost of `100`.

---

# Part 4 – Change the STP Port Priority on SW1

Increase the **VLAN 1 port priority** of SW1's **Fa0/1** interface to `240`.

### Configuration

On SW1:

```text
enable
configure terminal

interface fastethernet 0/1
spanning-tree vlan 1 port-priority 240

end
write memory
```

Verify:

```text
show spanning-tree vlan 1
```

The interface should now display a priority of `240`.

### Question

**Does SW3 select a different root port? Why or why not?**

Explain how STP uses port priority when comparing paths.

Remember that STP does not select a root port based only on port priority. It first considers the root path cost and then uses additional tie-breakers when necessary.

---

# Part 5 – Configure PortFast and BPDU Guard

Configure **PortFast** and **BPDU Guard** on the **Fa0/3** interfaces of SW3 and SW4.

These interfaces are intended to operate as access/edge ports connected to end devices.

---

## SW3 Configuration

```text
enable
configure terminal

interface fastethernet 0/3
switchport mode access
spanning-tree portfast
spanning-tree bpduguard enable

end
write memory
```

Verify:

```text
show running-config
```

You should see:

```text
interface FastEthernet0/3
 switchport mode access
 spanning-tree portfast
 spanning-tree bpduguard enable
```

---

## SW4 Configuration

```text
enable
configure terminal

interface fastethernet 0/3
switchport mode access
spanning-tree portfast
spanning-tree bpduguard enable

end
write memory
```

Verify:

```text
show running-config
```

You should see:

```text
interface FastEthernet0/3
 switchport access vlan 2
 switchport mode access
 spanning-tree portfast
 spanning-tree bpduguard enable
```

---

## PortFast

**PortFast** allows an access port connected to an end device to transition to the forwarding state without waiting through the normal STP listening and learning process.

It should normally be used only on ports connected to end devices such as:

- PCs
- Printers
- Servers
- Other endpoint devices

It should **not** normally be enabled on switch-to-switch links.

---

## BPDU Guard

**BPDU Guard** protects PortFast-enabled ports from receiving Bridge Protocol Data Units (BPDUs).

If a BPDU is received on a BPDU Guard-enabled port, the switch can place the interface into an **err-disabled** state.

This helps prevent an unauthorized switch from being connected to an edge port and potentially affecting the STP topology.

---

# Verification

After completing the configuration, verify the final STP topology.

Use:

```text
show spanning-tree
```

For individual VLANs:

```text
show spanning-tree vlan 1
show spanning-tree vlan 2
```

Verify the configuration:

```text
show running-config
```

For specific interfaces:

```text
show running-config interface fastethernet 0/1
show running-config interface fastethernet 0/2
show running-config interface fastethernet 0/3
```

---

# Final Verification Checklist

- [ ] Identified the original root bridge.
- [ ] Identified the original STP role/state of each port.
- [ ] Configured SW1 as the primary root for VLAN 1.
- [ ] Configured SW1 as the secondary root for VLAN 2.
- [ ] Configured SW2 as the primary root for VLAN 2.
- [ ] Configured SW2 as the secondary root for VLAN 1.
- [ ] Verified the new root bridges.
- [ ] Verified the new STP port roles and states.
- [ ] Changed SW4 Fa0/2 VLAN 1 cost to `100`.
- [ ] Verified whether SW4 changed its root port.
- [ ] Changed SW1 Fa0/1 VLAN 1 port priority to `240`.
- [ ] Verified whether SW3 changed its root port.
- [ ] Configured PortFast on SW3 Fa0/3.
- [ ] Configured BPDU Guard on SW3 Fa0/3.
- [ ] Configured PortFast on SW4 Fa0/3.
- [ ] Configured BPDU Guard on SW4 Fa0/3.
- [ ] Saved the configurations.
- [ ] Verified the final STP topology.

---

# Key Takeaways

This lab demonstrates that STP topology is not static. It can be influenced by several parameters, including:

1. **Root bridge priority** – determines which switch becomes the root bridge.
2. **Root path cost** – determines the best path toward the root bridge.
3. **Port priority** – helps STP select between otherwise comparable paths.
4. **Port role** – identifies whether a port is Root, Designated, or Alternate.
5. **Port state** – identifies whether the port is Forwarding, Blocking, or Learning.
6. **PortFast** – allows endpoint ports to reach forwarding quickly.
7. **BPDU Guard** – protects edge ports from unexpected BPDUs.

The main goal is to understand **why STP chooses a particular port or switch**, rather than simply memorizing the commands used to configure it.