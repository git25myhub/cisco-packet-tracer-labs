# Day 01 – Packet Tracer Introduction Lab

## Overview

This lab introduces the basics of building and working with network topologies in **Cisco Packet Tracer**.

The objective is to recreate the network diagram demonstrated at approximately **16:40 in the Day 1 video**. The topology represents two branch offices connected through the Internet, with firewalls positioned between the branch routers and the Internet.

The completed topology should closely resemble the following structure:

```text
                         INTERNET
                            |
                         ATTACKER
                            |
                 ┌──────────┴──────────┐
                 |                     |
              FW1                     FW2
                 |                     |
                R1                     R2
                 |                     |
                SW1                    SW2
              /    \                 /    \
            PC1    PC2             SRV1   SRV2
        New York Branch          Tokyo Branch
```

> **Note:** The exact visual placement of devices should follow the diagram shown in the Day 1 video and the provided reference topology.

---

## Lab Objectives

By completing this lab, you will:

- Become familiar with the Cisco Packet Tracer workspace.
- Create a network topology from a reference diagram.
- Select and place different Cisco network devices.
- Connect network devices together.
- Use Packet Tracer's **Automatically Choose Connection Type** feature.
- Organize devices into logical branch locations.
- Understand the basic role of routers, switches, firewalls, PCs, servers, and an attacker host.
- Verify that the physical topology matches the required design.

---

## Required Devices

Build the topology using the following devices:

| Device | Quantity | Purpose |
|---|---:|---|
| Cisco 2911 Router | 2 | Branch routers |
| Cisco 2960 Switch | 2 | LAN switches |
| Cisco ASA 5505 Firewall | 2 | Branch firewalls |
| PC | 2 | End-user devices |
| Server | 2 | Branch servers |
| Laptop | 1 | Attacker host |
| Internet/Cloud | 1 | WAN/Internet representation |

### Device Naming

Use the following names to make the topology easy to identify:

**New York Branch**

- `PC1`
- `PC2`
- `SW1`
- `R1`
- `FW1`

**Tokyo Branch**

- `R2`
- `FW2`
- `SW2`
- `SRV1`
- `SRV2`

**Internet**

- `The Internet`
- `ATTACKER`

---

# 1. Create the Packet Tracer Workspace

1. Open **Cisco Packet Tracer**.
2. Create a new Packet Tracer project.
3. Select the **Logical** workspace.
4. Arrange the workspace so there is enough room for both branch networks and the Internet connection.
5. Recreate the general layout shown in the Day 1 video.

A recommended arrangement is:

```text
New York Branch                              Tokyo Branch

PC2 ──┐                                  ┌── SRV1
      ├── SW1 ── R1 ── FW1 ── INTERNET ── FW2 ── R2 ── SW2
PC1 ──┘                                  └── SRV2
                                  |
                              ATTACKER
```

---

# 2. Add the Routers

Add **two Cisco 2911 routers** to the workspace.

### Router 1

Place the first router on the New York side.

Rename it:

```text
R1
```

### Router 2

Place the second router on the Tokyo side.

Rename it:

```text
R2
```

The routers should be positioned between their respective switches and firewalls.

```text
SW1 ─── R1 ─── FW1
```

and

```text
FW2 ─── R2 ─── SW2
```

---

# 3. Add the Switches

Add **two Cisco 2960 switches**.

### Switch 1

Place it in the New York branch and rename it:

```text
SW1
```

Connect the New York PCs to this switch.

### Switch 2

Place it in the Tokyo branch and rename it:

```text
SW2
```

Connect the Tokyo servers to this switch.

The LAN portions should look like:

```text
       PC1
        |
        |
PC2 ── SW1 ── R1
```

and:

```text
       SRV1
        |
        |
SRV2 ─ SW2 ─ R2
```

---

# 4. Add the Firewalls

Add **two Cisco ASA 5505 firewalls**.

### Firewall 1

Place it between `R1` and the Internet.

Rename it:

```text
FW1
```

The connection should follow:

```text
SW1 → R1 → FW1 → Internet
```

### Firewall 2

Place it between the Internet and `R2`.

Rename it:

```text
FW2
```

The connection should follow:

```text
Internet → FW2 → R2 → SW2
```

The final WAN path should therefore be:

```text
R1 → FW1 → The Internet → FW2 → R2
```

---

# 5. Add the PCs

Add **two PCs** to represent users in the New York branch.

Rename them:

```text
PC1
PC2
```

Connect both PCs to `SW1`.

The topology should be:

```text
PC1 ──┐
      |
      SW1 ── R1
      |
PC2 ──┘
```

Position the PCs on the left side of the topology.

---

# 6. Add the Servers

Add **two servers** to represent services in the Tokyo branch.

Rename them:

```text
SRV1
SRV2
```

Connect both servers to `SW2`.

The topology should be:

```text
SRV1 ─┐
      |
      SW2 ── R2
      |
SRV2 ─┘
```

Position the servers on the right side of the topology.

---

# 7. Add the Attacker Laptop

Add a **Laptop** to represent an attacker on the Internet.

Rename the device:

```text
ATTACKER
```

Place it below the Internet device, similar to the reference diagram.

The attacker should be connected to the Internet:

```text
             The Internet
                  |
                  |
              ATTACKER
```

The attacker is included in the topology to represent a host that may later be used for security-related exercises.

---

# 8. Add the Internet Device

Add the appropriate **Internet/Cloud representation** available in Packet Tracer.

Rename it:

```text
The Internet
```

Place it in the center of the topology.

The Internet should provide the central connection between the New York and Tokyo branches:

```text
R1 ── FW1 ── The Internet ── FW2 ── R2
                         |
                     ATTACKER
```

---

# 9. Connect the Devices

For this lab, use Packet Tracer's:

> **Automatically Choose Connection Type**

function when creating connections.

### How to use it

1. Select the **Connections** tool.
2. Choose **Automatically Choose Connection Type**.
3. Click the first device.
4. Select the appropriate interface.
5. Click the second device.
6. Select the appropriate interface if prompted.
7. Repeat until all required connections are complete.

Do **not** manually select copper straight-through, crossover, fiber, serial, or other cable types unless specifically instructed by the lab.

The purpose of this exercise is to become familiar with Packet Tracer's automatic connection functionality.

---

# 10. Required Connections

Create the following logical connections.

### New York Branch

```text
PC1  ── SW1
PC2  ── SW1
SW1  ── R1
R1   ── FW1
```

### Internet/WAN

```text
FW1  ── The Internet
The Internet ── FW2
The Internet ── ATTACKER
```

### Tokyo Branch

```text
FW2  ── R2
R2   ── SW2
SW2  ── SRV1
SW2  ── SRV2
```

---

# 11. Final Topology

When finished, your topology should have the following overall structure:

```text
                    NEW YORK                         TOKYO
                    BRANCH                          BRANCH

                 PC2
                  |
                  |
                 SW1
                /   \
             PC1     R1
                      |
                     FW1
                      |
                      |
                THE INTERNET
                   /     \
                  /       \
             ATTACKER     FW2
                           |
                          R2
                           |
                          SW2
                         /   \
                      SRV1   SRV2
```

A more linear representation is:

```text
PC1 ─┐
     ├── SW1 ── R1 ── FW1 ── The Internet ── FW2 ── R2 ── SW2 ──┬── SRV1
PC2 ─┘                                                            └── SRV2
                                    |
                                ATTACKER
```

---

# 12. Organize the Topology

Arrange the devices so the diagram is easy to read.

Recommended layout:

```text
┌─────────────────────┐                         ┌─────────────────────┐
│    New York Branch  │                         │     Tokyo Branch    │
│                     │                         │                     │
│ PC2                 │                         │                 SRV1│
│  \                  │                         │                /    │
│   SW1 ── R1 ── FW1 ─┼──── The Internet ─────┼─ FW2 ── R2 ── SW2   │
│  /                  │             |           │                \    │
│ PC1                 │         ATTACKER        │                 SRV2│
└─────────────────────┘                         └─────────────────────┘
```

Use device labels consistently so the topology is easy to understand.

---

# 13. Verification Checklist

Before considering the lab complete, verify the following:

- [ ] Two Cisco 2911 routers are present.
- [ ] Two Cisco 2960 switches are present.
- [ ] Two Cisco 5505 firewalls are present.
- [ ] Two PCs are present.
- [ ] Two servers are present.
- [ ] One laptop is present as the attacker.
- [ ] An Internet/cloud device is present.
- [ ] The first branch is labeled **New York Branch**.
- [ ] The second branch is labeled **Tokyo Branch**.
- [ ] The routers are named `R1` and `R2`.
- [ ] The switches are named `SW1` and `SW2`.
- [ ] The firewalls are named `FW1` and `FW2`.
- [ ] The PCs are named `PC1` and `PC2`.
- [ ] The servers are named `SRV1` and `SRV2`.
- [ ] The attacker is named `ATTACKER`.
- [ ] The Internet device is named `The Internet`.
- [ ] PC1 is connected to SW1.
- [ ] PC2 is connected to SW1.
- [ ] SW1 is connected to R1.
- [ ] R1 is connected to FW1.
- [ ] FW1 is connected to the Internet.
- [ ] The Internet is connected to FW2.
- [ ] The Internet is connected to the attacker laptop.
- [ ] FW2 is connected to R2.
- [ ] R2 is connected to SW2.
- [ ] SRV1 is connected to SW2.
- [ ] SRV2 is connected to SW2.
- [ ] Connections were created using **Automatically Choose Connection Type**.
- [ ] The topology generally matches the Day 1 video at approximately **16:40**.

---

# 14. Important Notes

This is primarily a **topology-building exercise**. The lab instructions do not require IP addressing, routing configuration, firewall configuration, or end-to-end connectivity testing.

Therefore, at this stage:

- Do not configure IP addresses unless instructed later.
- Do not configure routing protocols.
- Do not configure firewall policies.
- Do not configure DHCP.
- Do not configure NAT.
- Do not configure DNS.
- Do not configure server services.

The focus is on correctly creating and connecting the physical/logical topology in Packet Tracer.

---

# 15. Expected Result

At the end of the lab, the Packet Tracer workspace should contain:

**New York Branch**

```text
PC1
PC2
 |
SW1
 |
R1
 |
FW1
```

**Internet**

```text
FW1 ── The Internet ── FW2
              |
          ATTACKER
```

**Tokyo Branch**

```text
FW2
 |
R2
 |
SW2
 |
SRV1
SRV2
```

The completed topology should visually resemble the reference diagram provided in the lab instructions, with the New York branch on the left, the Tokyo branch on the right, the Internet in the center, and the attacker laptop below the Internet connection.

---

## Lab Completion

The lab is complete when the entire topology has been recreated using the specified devices, all devices have been correctly named and connected, and the final Packet Tracer workspace closely matches the topology demonstrated in the Day 1 video.

**Lab focus:** Packet Tracer fundamentals, device selection, topology creation, device naming, and automatic cable selection.