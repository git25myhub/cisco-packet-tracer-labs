# Rapid Spanning Tree Protocol (RSTP) – Port Roles, Link Types & PortFast Lab

## Lab Overview

In this lab, you will examine how **Rapid Spanning Tree Protocol (RSTP)** determines port roles and states. You will identify the root bridge, analyze the interfaces on the root bridge, determine the roles of interfaces on the remaining switches, and manually configure the appropriate RSTP link types.

You will also examine the difference between RSTP's **Point-to-Point** and **Shared** link types and configure PortFast on an edge interface.

> **Important:** Turn off link lights in Packet Tracer for this lab:  
> **Options > Preferences > Show Link Lights**

---

## Objectives

By the end of this lab, you should be able to:

- Identify the RSTP root bridge.
- Identify RSTP port roles and states.
- Explain why the root bridge can have a non-forwarding port.
- Determine RSTP port roles by analyzing the topology.
- Verify predicted port roles using the CLI.
- Identify Point-to-Point and Shared RSTP link types.
- Manually configure RSTP link types.
- Configure PortFast on an edge/access interface.
- Explain why the correct link type matters for RSTP convergence.

---

# Part 1 – Identify the Root Bridge

Use the CLI to determine which switch is currently the **root bridge**.

On each switch, run:

```text
enable
show spanning-tree
```

You can also use:

```text
show spanning-tree vlan 1
```

Look for:

```text
Root ID
```

and the message:

```text
This bridge is the root
```

### Question 1

**Which switch is the root bridge?**

Record your answer:

```text
Root Bridge: __________________
```

---

## Examine the Root Bridge

Once you have identified the root bridge, examine the role and state of every interface.

Use:

```text
show spanning-tree vlan 1
```

The output will contain a table similar to:

```text
Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------
Fa0/3            Back BLK 19        128.3    Shr
Fa0/2            Desg FWD 19        128.2    Shr
Fa0/24           Desg FWD 19        128.24   Shr
Fa0/1            Desg FWD 19        128.1    P2p
```

Record the results for the root bridge:

| Interface | Role | State | Link Type |
|---|---|---|---|
| Fa0/1 | | | |
| Fa0/2 | | | |
| Fa0/3 | | | |
| Fa0/24 | | | |

---

## Question 2 – What Looks Different?

Think about what you have learned regarding the root bridge.

Normally, you may expect the root bridge to have **all of its active ports in the Designated/Forwarding state**.

However, the observed root bridge has:

```text
Fa0/3    Back BLK
```

while other interfaces are Designated/Forwarding.

### Question

**What appears different about the root bridge compared with what you have previously learned about STP?**

Explain why a port on the root bridge can appear as:

```text
Back BLK
```

instead of:

```text
Desg FWD
```

---

## Cause of the Difference

The important clue is the RSTP **link type**:

```text
Type
----
Shr
```

A port displayed as **Shr** is operating as a **shared link**.

RSTP treats shared links differently from point-to-point links. A shared link is typically associated with a half-duplex Ethernet connection or a medium where multiple devices can share the same segment.

This can cause RSTP to use the **Backup** port role.

The RSTP Backup port provides a redundant path to the same shared segment where another port on the switch is already the Designated Port.

### Key Concept

Do not assume that every port on the root bridge must always appear as Designated/Forwarding.

The **Backup** role exists in RSTP and is used for redundancy on shared segments.

---

# Part 2 – Predict the Remaining Port Roles

Before using the CLI, analyze the topology yourself.

For each remaining switch:

- Identify the path toward the root bridge.
- Determine which interface should be the Root Port.
- Determine which interfaces should be Designated Ports.
- Identify any Alternate or Backup ports.
- Determine whether each port should be Forwarding or Blocking.

**Do not use the CLI initially for this step.**

Use the topology diagram and your knowledge of RSTP.

---

## SW2 Prediction

Record your prediction:

| Interface | Predicted Role | Predicted State |
|---|---|---|
| Fa0/1 | | |
| Fa0/2 | | |
| Fa0/23 | | |
| Fa0/24 | | |
| Gi0/1 | | |

Then verify your answer with:

```text
show spanning-tree vlan 1
```

The observed topology shows:

```text
Fa0/1     Root FWD
Fa0/2     Desg FWD
Fa0/23    Desg FWD
Fa0/24    Desg FWD
Gi0/1     Altn BLK
```

---

## SW3 Prediction

Before checking the CLI, determine the expected role/state of each interface.

| Interface | Predicted Role | Predicted State |
|---|---|---|
| Fa0/1 | | |
| Fa0/2 | | |
| Fa0/24 | | |
| Gi0/1 | | |

Then verify:

```text
show spanning-tree vlan 1
```

The observed topology shows:

```text
Fa0/2     Root FWD
Fa0/1     Desg FWD
Gi0/1     Desg FWD
Fa0/24    Desg FWD
```

---

## SW4 Prediction

Determine the expected role/state before checking the CLI.

| Interface | Predicted Role | Predicted State |
|---|---|---|
| Fa0/1 | | |
| Fa0/2 | | |
| Fa0/24 | | |

Then verify:

```text
show spanning-tree vlan 1
```

The observed topology shows:

```text
Fa0/1     Root FWD
Fa0/2     Altn BLK
Fa0/24    Desg FWD
```

---

# Part 3 – Understand RSTP Link Types

RSTP can classify links as:

- **Point-to-Point**
- **Shared**
- **Edge**

The link type affects how RSTP handles convergence and synchronization.

---

## Point-to-Point Links

A point-to-point link normally exists between two switches connected through a full-duplex Ethernet connection.

RSTP can rapidly transition a point-to-point link because there are only two devices participating on the link.

The CLI command is:

```text
spanning-tree link-type point-to-point
```

---

## Shared Links

A shared link is a segment where multiple devices can potentially communicate over the same medium.

The output may display:

```text
Shr
```

RSTP handles these links differently from point-to-point links.

In this lab, SW1 has interfaces that appear as:

```text
Fa0/2    Desg FWD    Shr
Fa0/3    Back BLK    Shr
Fa0/24   Desg FWD    Shr
```

while Fa0/1 appears as:

```text
Fa0/1    Desg FWD    P2p
```

---

# Part 4 – Manually Configure the RSTP Link Types

Manually configure the appropriate RSTP link type on each interface.

For switch-to-switch links that are point-to-point, use:

```text
interface <interface>
spanning-tree link-type point-to-point
```

For example:

```text
interface range fastethernet 0/1 - 2
spanning-tree link-type point-to-point
```

Verify the result with:

```text
show spanning-tree
```

Look at the **Type** column.

---

# Part 5 – SW4 Link-Type Configuration

SW4 has two interfaces participating in the redundant switch topology:

```text
Fa0/1
Fa0/2
```

These are switch-to-switch links and should be treated as **Point-to-Point** links.

Configure:

```text
enable
configure terminal

interface range fastethernet 0/1 - 2
spanning-tree link-type point-to-point

end
write memory
```

Verify:

```text
show spanning-tree vlan 1
```

The interfaces should now display:

```text
P2p
```

---

# Part 6 – What Is the Correct Link Type for SW1 Fa0/24?

This is the key question of the lab.

### Question

**What do you think is the correct RSTP link type for SW1's Fa0/24?**

Before configuring anything, inspect the topology.

Ask yourself:

- Is Fa0/24 connected to another switch?
- Is it connected to an end device?
- Is the interface operating as an access port?
- Should it be treated as an edge port?
- Would configuring it as point-to-point make sense?

If Fa0/24 is connected to an **end device**, the appropriate configuration is **PortFast**, rather than manually forcing a point-to-point link type.

Example:

```text
interface fastethernet 0/24
spanning-tree portfast
```

Verify:

```text
show running-config interface fastethernet 0/24
```

The configuration should show:

```text
spanning-tree portfast
```

> **Important:** PortFast should only be used on interfaces connected to end devices. Do not enable PortFast on a switch-to-switch link.

---

# Part 7 – PortFast and RSTP Edge Ports

An interface connected to an endpoint is considered an **edge port**.

PortFast allows the interface to immediately transition toward forwarding rather than waiting through the normal STP process.

Configure PortFast when the port is connected to devices such as:

- PCs
- Servers
- Printers
- Other endpoint devices

Example:

```text
interface fastethernet 0/24
spanning-tree portfast
```

Verify:

```text
show spanning-tree
```

and:

```text
show running-config interface fastethernet 0/24
```

---

# Verification

After completing the lab, verify all switches.

Use:

```text
show spanning-tree
```

and:

```text
show spanning-tree vlan 1
```

For interface-specific configuration:

```text
show running-config interface fastethernet 0/24
```

For SW4:

```text
show running-config interface range fastethernet 0/1 - 2
```

---

# Final Questions

Answer the following questions in your lab notes:

1. Which switch is the root bridge?

2. What are the roles and states of all interfaces on the root bridge?

3. What appears unusual about the root bridge's port roles?

4. What is the cause of the unusual port role/state?

5. Without using the CLI, what role/state did you predict for each interface on SW2?

6. How accurate were your predictions after verifying with the CLI?

7. What role/state did you predict for each interface on SW3?

8. What role/state did you predict for each interface on SW4?

9. What is the difference between a **Point-to-Point** and **Shared** RSTP link?

10. Why is a switch-to-switch Ethernet link normally treated as Point-to-Point?

11. What is the appropriate configuration for an interface connected to an end device?

12. **What is the correct RSTP link type or edge configuration for SW1 Fa0/24, and why?**

---

# Useful Commands

### Display STP information

```text
show spanning-tree
```

### Display STP for VLAN 1

```text
show spanning-tree vlan 1
```

### Display the running configuration

```text
show running-config
```

### Configure Point-to-Point link type

```text
interface <interface>
spanning-tree link-type point-to-point
```

### Configure PortFast

```text
interface <interface>
spanning-tree portfast
```

### Save configuration

```text
write memory
```

or:

```text
copy running-config startup-config
```

---

# Key Takeaways

This lab demonstrates several important RSTP concepts:

- The **root bridge** does not necessarily have every interface listed as Designated/Forwarding.
- RSTP introduces the **Backup** and **Alternate** port roles.
- The **link type** affects RSTP's behavior and convergence.
- **Point-to-Point** links are normally used between two switches over full-duplex Ethernet.
- **Shared** links can result in different RSTP behavior, including Backup ports.
- **PortFast** should be used on ports connected to end devices.
- A switch-to-switch link should **not** be configured as an edge/PortFast port.
- RSTP troubleshooting requires understanding the **topology, root bridge, path cost, port roles, and link types**, rather than relying only on CLI output.

The goal of this lab is not just to identify the port role shown by `show spanning-tree`, but to be able to **predict why RSTP selected that role before verifying it with the CLI**.