# Spanning Tree Protocol (STP) — Root Bridge & Port Roles

## Lab Overview

This lab focuses on **Spanning Tree Protocol (STP)** and how switches determine:

- The **Root Bridge**
- **Root Ports**
- **Designated Ports**
- **Alternate/Non-Designated Ports**
- Forwarding and Blocking states

The objective is to analyze the topology before verifying the results through Cisco IOS CLI commands.

> **Packet Tracer Requirement:**  
> Turn off link lights before beginning the lab:
>
> **Options → Preferences → Show Link Lights**  
> Uncheck **Show Link Lights**.

---

## Objectives

By the end of this lab, you should be able to:

1. Identify the STP Root Bridge.
2. Determine the role of every relevant switch port.
3. Understand why STP places certain ports into a blocking state.
4. Verify STP information using Cisco IOS commands.
5. Interpret the output of `show spanning-tree`.
6. Use STP path cost and bridge priority to understand root-port selection.

---

# Part 1 — Identify the Root Bridge

Before using the CLI, examine the topology and predict which switch should become the **Root Bridge**.

STP elects the Root Bridge using the **lowest Bridge ID**, which is determined by:

1. Bridge priority
2. Extended System ID
3. MAC address

The switch with the lowest Bridge ID becomes the Root Bridge.

### Question

**Which switch is the Root Bridge?**

**Answer:** `SW3`

SW3 has the lowest bridge priority:

```text
Bridge ID Priority: 24577
```

The other switches have higher priorities:

- SW1: `32769`
- SW2: `28673`
- SW4: `32769`

Therefore, **SW3 is elected as the Root Bridge**.

---

# Part 2 — Predict the Port Roles

Before confirming the results using the CLI, identify the role of each port.

Use the following STP terminology:

| STP Role | Meaning |
|---|---|
| **Root** | The port providing the lowest-cost path toward the Root Bridge |
| **Designated** | The forwarding port selected for a network segment |
| **Non-Designated / Alternate** | A redundant path placed into blocking to prevent loops |

## SW1

```text
SW1 F0/1:
SW1 F0/2:
SW1 F0/3:
SW1 F0/4:
```

### Verified Roles

| Port | Role | State |
|---|---|---|
| Fa0/1 | Alternate | Blocking |
| Fa0/2 | Alternate | Blocking |
| Fa0/3 | Alternate | Blocking |
| Fa0/4 | Root | Forwarding |

SW1's **Fa0/4** is the Root Port because it provides the best path toward SW3.

---

## SW2

```text
SW2 F0/1:
SW2 F0/2:
SW2 F0/3:
SW2 G0/1:
```

### Verified Roles

| Port | Role | State |
|---|---|---|
| Fa0/1 | Designated | Forwarding |
| Fa0/2 | Designated | Forwarding |
| Fa0/3 | Alternate | Blocking |
| Gi0/1 | Root | Forwarding |

SW2's **Gi0/1** is the Root Port.

SW2's **Fa0/3** is an Alternate Port and is therefore placed into the blocking state.

---

## SW3

```text
SW3 F0/1:
SW3 F0/2:
SW3 F0/3:
SW3 G0/1:
```

### Verified Roles

| Port | Role | State |
|---|---|---|
| Fa0/1 | Designated | Forwarding |
| Fa0/2 | Designated | Forwarding |
| Fa0/3 | Designated | Forwarding |
| Gi0/1 | Designated | Forwarding |

Because **SW3 is the Root Bridge**, all of its active ports are **Designated Ports** and are in the forwarding state.

---

## SW4

```text
SW4 G0/1:
SW4 G0/2:
```

### Verified Roles

| Port | Role | State |
|---|---|---|
| Gi0/1 | Designated | Forwarding |
| Gi0/2 | Root | Forwarding |

SW4's **Gi0/2** is its Root Port because it provides the path toward SW3.

---

# Part 3 — Complete STP Port-Roles Table

The completed table is:

| Switch | Port | STP Role | State |
|---|---|---|---|
| **SW1** | Fa0/1 | Alternate | Blocking |
| | Fa0/2 | Alternate | Blocking |
| | Fa0/3 | Alternate | Blocking |
| | Fa0/4 | Root | Forwarding |
| **SW2** | Fa0/1 | Designated | Forwarding |
| | Fa0/2 | Designated | Forwarding |
| | Fa0/3 | Alternate | Blocking |
| | Gi0/1 | Root | Forwarding |
| **SW3** | Fa0/1 | Designated | Forwarding |
| | Fa0/2 | Designated | Forwarding |
| | Fa0/3 | Designated | Forwarding |
| | Gi0/1 | Designated | Forwarding |
| **SW4** | Gi0/1 | Designated | Forwarding |
| | Gi0/2 | Root | Forwarding |

---

# Part 4 — Verify Using the CLI

After making your predictions, verify the answers directly on each switch.

Enter privileged EXEC mode:

```text
Switch> enable
Switch#
```

Then use:

```text
show spanning-tree
```

You can also specifically examine VLAN 1:

```text
show spanning-tree vlan 1
```

---

# Part 5 — Verify the Root Bridge

On **SW3**, run:

```text
SW3# show spanning-tree
```

The important section is:

```text
Root ID    Priority    24577
           Address     00E0.F9E6.44A5
           This bridge is the root
```

The line:

```text
This bridge is the root
```

confirms that **SW3 is the Root Bridge**.

SW3 also shows every active port as:

```text
Desg FWD
```

meaning **Designated / Forwarding**.

---

# Part 6 — Verify SW2

On SW2:

```text
SW2# show spanning-tree
```

The relevant output is:

```text
Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Gi0/1            Root FWD 4         128.25   P2p
Fa0/2            Desg FWD 19        128.2    P2p
Fa0/1            Desg FWD 19        128.1    P2p
Fa0/3            Altn BLK 19        128.3    P2p
```

This confirms:

- **Gi0/1** → Root Port
- **Fa0/1** → Designated Port
- **Fa0/2** → Designated Port
- **Fa0/3** → Alternate Port / Blocking

The root path cost is:

```text
Cost 8
```

---

# Part 7 — Verify SW4

On SW4:

```text
SW4# show spanning-tree
```

The output shows:

```text
Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Gi0/1            Desg FWD 4         128.25   P2p
Gi0/2            Root FWD 4         128.26   P2p
```

Therefore:

- **Gi0/1** → Designated / Forwarding
- **Gi0/2** → Root / Forwarding

SW4's Root Port is **Gi0/2**.

---

# Part 8 — Verify SW1

On SW1:

```text
SW1# show spanning-tree
```

The output shows:

```text
Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/4            Root FWD 19        128.4    P2p
Fa0/2            Altn BLK 19        128.2    P2p
Fa0/3            Altn BLK 19        128.3    P2p
Fa0/1            Altn BLK 19        128.1    P2p
```

Therefore:

- **Fa0/4** → Root / Forwarding
- **Fa0/1** → Alternate / Blocking
- **Fa0/2** → Alternate / Blocking
- **Fa0/3** → Alternate / Blocking

---

# Part 9 — Understanding the STP Decision

The topology contains redundant Layer 2 paths. Without STP, these redundant links could create switching loops.

STP prevents loops by allowing only the best path to the Root Bridge to forward traffic.

The general decision process is:

```text
1. Elect Root Bridge
        ↓
2. Each non-root switch selects a Root Port
        ↓
3. Each network segment selects a Designated Port
        ↓
4. Redundant ports become Alternate/Non-Designated
        ↓
5. Alternate ports enter Blocking state
```

In this topology:

```text
                 SW3
              ROOT BRIDGE
             /    |    \
            /     |     \
          SW1    SW2    SW4
                  |
              redundant
               paths
```

The exact forwarding topology is determined by STP path cost.

---

# Part 10 — Useful Verification Commands

### Display STP information

```text
show spanning-tree
```

### Display STP for VLAN 1

```text
show spanning-tree vlan 1
```

### Display detailed STP information

```text
show spanning-tree detail
```

### Display an STP summary

```text
show spanning-tree summary
```

### Display interface information

```text
show interfaces
```

### Display interface status

```text
show interfaces status
```

---

# Key Observations

### Root Bridge

**SW3** is the Root Bridge.

Evidence:

```text
This bridge is the root
```

and:

```text
Priority 24577
Address 00E0.F9E6.44A5
```

### Root Ports

The non-root switches use the following Root Ports:

```text
SW1 → Fa0/4
SW2 → Gi0/1
SW4 → Gi0/2
```

### Alternate/Blocking Ports

The redundant paths are blocked on:

```text
SW1 → Fa0/1
SW1 → Fa0/2
SW1 → Fa0/3

SW2 → Fa0/3
```

### Designated Ports

SW3, as the Root Bridge, has all four active ports designated:

```text
SW3 → Fa0/1
SW3 → Fa0/2
SW3 → Fa0/3
SW3 → Gi0/1
```

SW2 also has:

```text
SW2 → Fa0/1
SW2 → Fa0/2
```

as Designated Ports.

SW4 has:

```text
SW4 → Gi0/1
```

as a Designated Port.

---

# Final Answers

| Switch | F0/1 | F0/2 | F0/3 | F0/4 | G0/1 | G0/2 |
|---|---|---|---|---|---|---|
| **SW1** | Alternate / Blocking | Alternate / Blocking | Alternate / Blocking | Root / Forwarding | — | — |
| **SW2** | Designated / Forwarding | Designated / Forwarding | Alternate / Blocking | — | Root / Forwarding | — |
| **SW3** | Designated / Forwarding | Designated / Forwarding | Designated / Forwarding | — | Designated / Forwarding | — |
| **SW4** | — | — | — | — | Designated / Forwarding | Root / Forwarding |

## Root Bridge

**SW3**

## Main STP Lesson

STP creates a loop-free Layer 2 topology by selecting a **Root Bridge**, choosing the best **Root Port** on each non-root switch, selecting **Designated Ports** for each segment, and placing redundant **Alternate Ports** into a blocking state.