# Cisco Packet Tracer Lab — VLANs, Inter-VLAN Routing & Broadcast Domains

## 📌 Lab Overview

This lab demonstrates the configuration of **VLANs and inter-VLAN routing** using a Cisco router and switch.

A single switch is divided into three logical networks:

- **VLAN 10 — ENGINEERING**
- **VLAN 20 — HR**
- **VLAN 30 — SALES**

Each VLAN represents a separate broadcast domain. The router provides the Layer 3 gateway for each VLAN, allowing devices in different VLANs to communicate.

The lab also demonstrates how broadcast traffic behaves within VLANs using **Cisco Packet Tracer Simulation Mode**.

---

# 🎯 Objectives

The main objectives of this lab are to:

1. Configure the correct IP address and subnet mask on each PC.
2. Configure the **last usable address** of each subnet as the PC's default gateway.
3. Create three separate connections between R1 and SW1.
4. Configure one router interface for each VLAN.
5. Assign the router interface IP addresses as the PCs' default gateways.
6. Configure switch interfaces as access ports in the appropriate VLANs.
7. Create and name the VLANs:
   - ENGINEERING
   - HR
   - SALES
8. Test connectivity between PCs.
9. Test broadcast behavior using the subnet broadcast address.
10. Observe broadcast propagation using Packet Tracer's **Simulation Mode**.

---

# 🗺️ Network Design

The topology consists of:

```text
                         R1
                  ┌──────┼──────┐
                  │      │      │
                G0/0   G0/1   G0/2
                  │      │      │
                  └──────┼──────┘
                         SW1
              ┌──────────┼──────────┐
              │          │          │
        ENGINEERING      HR        SALES
          VLAN 10      VLAN 20     VLAN 30
```

Unlike a traditional router-on-a-stick configuration, this lab uses **three physical router interfaces**, with one interface dedicated to each VLAN.

---

# 🔢 IP Addressing Plan

The three VLANs use `/26` subnets.

| VLAN | Department | Network | Subnet Mask | Usable Range | Broadcast | Gateway |
|---|---|---|---|---|---|---|
| 10 | ENGINEERING | `10.0.0.0/26` | `255.255.255.192` | `10.0.0.1 – 10.0.0.62` | `10.0.0.63` | `10.0.0.62` |
| 20 | HR | `10.0.0.64/26` | `255.255.255.192` | `10.0.0.65 – 10.0.0.126` | `10.0.0.127` | `10.0.0.126` |
| 30 | SALES | `10.0.0.128/26` | `255.255.255.192` | `10.0.0.129 – 10.0.0.190` | `10.0.0.191` | `10.0.0.190` |

### Gateway Rule

The lab specifically requires the **last usable address** of each subnet to be used as the default gateway.

Therefore:

- VLAN 10 → `10.0.0.62`
- VLAN 20 → `10.0.0.126`
- VLAN 30 → `10.0.0.190`

---

# 🖥️ R1 Configuration

Each physical interface on R1 acts as the Layer 3 gateway for one VLAN.

## VLAN 10 — ENGINEERING

```cisco
R1(config)#interface GigabitEthernet0/0
R1(config-if)#ip address 10.0.0.62 255.255.255.192
R1(config-if)#no shutdown
```

Gateway:

```text
10.0.0.62/26
```

---

## VLAN 20 — HR

```cisco
R1(config)#interface GigabitEthernet0/1
R1(config-if)#ip address 10.0.0.126 255.255.255.192
R1(config-if)#no shutdown
```

Gateway:

```text
10.0.0.126/26
```

---

## VLAN 30 — SALES

```cisco
R1(config)#interface GigabitEthernet0/2
R1(config-if)#ip address 10.0.0.190 255.255.255.192
R1(config-if)#no shutdown
```

Gateway:

```text
10.0.0.190/26
```

---

# 🔍 R1 Interface Verification

The following command was used to verify the router interfaces:

```cisco
R1#show ip interface brief
```

The resulting configuration showed:

```text
Interface              IP-Address      Status    Protocol

GigabitEthernet0/0     10.0.0.62       up        up
GigabitEthernet0/1     10.0.0.126      up        up
GigabitEthernet0/2     10.0.0.190      up        up
Vlan1                  unassigned      down      down
```

This confirms that all three router interfaces are operational.

---

# 🔀 Switch VLAN Configuration

SW1 was configured with three VLANs.

## VLAN 10 — ENGINEERING

```cisco
SW1(config)#vlan 10
SW1(config-vlan)#name ENGINEERING
```

Assigned interfaces:

```text
GigabitEthernet0/1
FastEthernet3/1
FastEthernet4/1
```

Configuration:

```cisco
SW1(config)#interface range GigabitEthernet0/1,FastEthernet3/1,FastEthernet4/1
SW1(config-if-range)#switchport mode access
SW1(config-if-range)#switchport access vlan 10
```

---

## VLAN 20 — HR

```cisco
SW1(config)#vlan 20
SW1(config-vlan)#name HR
```

Assigned interfaces:

```text
GigabitEthernet1/1
FastEthernet5/1
FastEthernet6/1
```

Configuration:

```cisco
SW1(config)#interface range GigabitEthernet1/1,FastEthernet5/1,FastEthernet6/1
SW1(config-if-range)#switchport mode access
SW1(config-if-range)#switchport access vlan 20
```

---

## VLAN 30 — SALES

```cisco
SW1(config)#vlan 30
SW1(config-vlan)#name SALES
```

Assigned interfaces:

```text
GigabitEthernet2/1
FastEthernet7/1
FastEthernet8/1
```

Configuration:

```cisco
SW1(config)#interface range GigabitEthernet2/1,FastEthernet7/1,FastEthernet8/1
SW1(config-if-range)#switchport mode access
SW1(config-if-range)#switchport access vlan 30
```

---

# 📋 Final VLAN Assignment

The final VLAN configuration was verified with:

```cisco
SW1#show vlan brief
```

The resulting configuration was:

```text
VLAN  Name             Status    Ports

10    ENGINEERING      active    Gig0/1, Fa3/1, Fa4/1

20    HR               active    Gig1/1, Fa5/1, Fa6/1

30    SALES            active    Gig2/1, Fa7/1, Fa8/1
```

This confirms that the switch interfaces are assigned to the correct VLANs.

---

# 🖥️ PC Addressing

Each PC should be configured with an address from its respective subnet.

The **default gateway must always be the last usable address**.

Example configuration:

| Department | Example PC IP | Subnet Mask | Default Gateway |
|---|---|---|---|
| ENGINEERING | `10.0.0.1` | `255.255.255.192` | `10.0.0.62` |
| HR | `10.0.0.65` | `255.255.255.192` | `10.0.0.126` |
| SALES | `10.0.0.129` | `255.255.255.192` | `10.0.0.190` |

Additional PCs in each department must use another valid address from the same subnet.

---

# 🌐 How Inter-VLAN Routing Works

Each VLAN is a separate Layer 2 broadcast domain.

For example:

```text
VLAN 10
10.0.0.0/26
      │
      │
    R1 G0/0
  10.0.0.62
      │
      │
      R1
      │
      │
    R1 G0/1
  10.0.0.126
      │
VLAN 20
10.0.0.64/26
```

When a PC in VLAN 10 wants to communicate with a PC in VLAN 20, the packet is sent to the VLAN 10 gateway:

```text
PC → 10.0.0.62 → R1 → 10.0.0.126 → PC
```

The router performs the Layer 3 forwarding between the two directly connected networks.

No static routes are required because all three networks are **directly connected to R1**.

---

# 📡 Broadcast Domain Testing

One of the important objectives of the lab is to observe how broadcasts behave.

Each VLAN represents a separate broadcast domain.

The broadcast addresses are:

| VLAN | Network | Broadcast Address |
|---|---|---|
| ENGINEERING | `10.0.0.0/26` | `10.0.0.63` |
| HR | `10.0.0.64/26` | `10.0.0.127` |
| SALES | `10.0.0.128/26` | `10.0.0.191` |

For example, from an Engineering PC:

```text
C:\>ping 10.0.0.63
```

This is a ping to the broadcast address of the Engineering subnet.

---

# 🔬 Packet Tracer Simulation Mode

To observe the broadcast behavior:

1. Switch Packet Tracer from **Realtime** to **Simulation** mode.
2. Open the PC's command prompt.
3. Send a ping to the subnet broadcast address.
4. Watch the packet as it travels through the topology.
5. Use the **Capture/Forward** button to examine each step.
6. Observe which PCs receive the broadcast.

### Expected Behavior

A Layer 2 broadcast remains within its VLAN/broadcast domain.

For example:

```text
ENGINEERING VLAN 10
        │
        ├── Engineering PC
        ├── Engineering PC
        └── Engineering PC
```

A broadcast generated in VLAN 10 should be seen by devices belonging to **VLAN 10**, but it should not be forwarded by the router into VLAN 20 or VLAN 30.

Similarly:

```text
VLAN 10  ── Broadcast Domain 1
VLAN 20  ── Broadcast Domain 2
VLAN 30  ── Broadcast Domain 3
```

The router separates these broadcast domains.

---

# 🧪 Connectivity Testing

After configuring the PCs, VLANs, and router interfaces, connectivity can be tested using `ping`.

### Same-VLAN Test

A PC in Engineering should be able to ping another PC in Engineering:

```text
C:\>ping 10.0.0.x
```

The same applies to HR and Sales.

### Inter-VLAN Test

A PC in Engineering should also be able to reach PCs in HR and Sales because R1 provides inter-VLAN routing.

For example:

```text
Engineering PC → HR PC
Engineering PC → Sales PC
HR PC → Sales PC
```

Successful replies confirm that:

- PC addressing is correct.
- Default gateways are correct.
- Switch VLAN assignments are correct.
- R1 interfaces are operational.
- Inter-VLAN routing is functioning.

---

# 💾 Saving the Configuration

The configurations were saved using:

```cisco
R1#write memory
SW1#write memory
```

The devices returned:

```text
Building configuration...
[OK]
```

This ensures the configurations are retained.

---

# 🧠 Key Networking Concepts

This lab demonstrates the following concepts:

- VLAN creation
- VLAN naming
- Access ports
- VLAN membership
- Layer 2 broadcast domains
- Layer 3 routing
- Inter-VLAN routing
- Default gateways
- Subnetting with `/26`
- Last usable IP addressing
- Broadcast addresses
- Router interface configuration
- Switch interface configuration
- Packet Tracer Simulation Mode
- Broadcast traffic analysis

---

# ✅ Lab Outcome

The lab successfully created three separate VLANs on SW1:

- **VLAN 10 — ENGINEERING**
- **VLAN 20 — HR**
- **VLAN 30 — SALES**

R1 was configured with a dedicated physical interface for each VLAN, using the **last usable IP address of each subnet as the default gateway**.

SW1 interfaces were assigned to the correct VLANs, and the configuration was verified using `show vlan brief`.

The design demonstrates that devices within the same VLAN share a broadcast domain, while the router provides communication between different VLANs.

The use of Packet Tracer Simulation Mode provides a visual demonstration of the difference between **unicast communication and broadcast traffic**, and shows how VLANs isolate broadcast domains.