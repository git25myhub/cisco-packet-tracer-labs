# Voice VLANs and Router-on-a-Stick (ROAS)

## Lab Overview

In this lab, you will configure **data and voice VLANs** on SW1 and implement **Router-on-a-Stick (ROAS)** between SW1 and R1.

You will then use **Simulation Mode** in Cisco Packet Tracer to observe how VLAN tagging differs between:

- PC-to-PC communication
- IP phone-to-IP phone communication

> **Note:** Telephony configurations on R1 have been pre-configured and are not required for CCNA configuration purposes.

---

## Lab Objectives

By completing this lab, you will learn how to:

- Configure access ports for a data VLAN.
- Configure a voice VLAN on switch ports.
- Configure an 802.1Q trunk between a switch and router.
- Configure Router-on-a-Stick using router subinterfaces.
- Understand how VLAN tagging works on trunk links.
- Analyze data and voice traffic using Packet Tracer Simulation Mode.

---

## Network Configuration

### VLANs

| VLAN | Purpose | Network |
|---|---|---|
| VLAN 10 | Data | 192.168.10.0/24 |
| VLAN 20 | Voice | 192.168.20.0/24 |

### R1 Subinterfaces

| Interface | VLAN | IP Address |
|---|---:|---|
| Fa0/0.10 | 10 | 192.168.10.1/24 |
| Fa0/0.20 | 20 | 192.168.20.1/24 |

---

# Part 1: Configure SW1 Interfaces

The interfaces connected to the PCs/IP phones need to be configured as access ports.

Configure **GigabitEthernet1/0/2 and GigabitEthernet1/0/3** with:

- Access mode
- Data VLAN 10
- Voice VLAN 20

### Configuration

```cisco
SW1> enable
SW1# configure terminal

SW1(config)# interface range g1/0/2 - 3
SW1(config-if-range)# switchport mode access
SW1(config-if-range)# switchport access vlan 10
SW1(config-if-range)# switchport voice vlan 20
```

If the VLANs do not already exist, IOS will create them when they are assigned to the interfaces.

### Verify

```cisco
SW1# show vlan brief
SW1# show interfaces g1/0/2 switchport
SW1# show interfaces g1/0/3 switchport
```

You should see:

- VLAN 10 as the access VLAN
- VLAN 20 as the voice VLAN

---

# Part 2: Configure Router-on-a-Stick

The connection between **SW1 G1/0/1** and **R1 Fa0/0** will carry both VLAN 10 and VLAN 20 traffic.

## Configure SW1 as a Trunk

```cisco
SW1(config)# interface g1/0/1
SW1(config-if)# switchport mode trunk
SW1(config-if)# switchport trunk allowed vlan 10,20
```

### Important Note

On this switch model, the command:

```cisco
switchport trunk encapsulation dot1q
```

is rejected because the switch only supports 802.1Q encapsulation and does not provide a choice of trunk encapsulation.

Therefore, simply configuring:

```cisco
switchport mode trunk
```

is sufficient.

### Verify

```cisco
SW1# show interfaces trunk
```

You should see VLANs **10 and 20** allowed on the trunk.

---

# Part 3: Configure R1 Router Subinterfaces

First, enable the physical interface connected to SW1.

```cisco
R1> enable
R1# configure terminal

R1(config)# interface fastethernet0/0
R1(config-if)# no shutdown
```

## Configure VLAN 10 Subinterface

```cisco
R1(config)# interface fastethernet0/0.10
R1(config-subif)# encapsulation dot1Q 10
R1(config-subif)# ip address 192.168.10.1 255.255.255.0
```

## Configure VLAN 20 Subinterface

```cisco
R1(config)# interface fastethernet0/0.20
R1(config-subif)# encapsulation dot1Q 20
R1(config-subif)# ip address 192.168.20.1 255.255.255.0
```

### Verify

```cisco
R1# show ip interface brief
R1# show running-config interface fastethernet0/0.10
R1# show running-config interface fastethernet0/0.20
```

The subinterfaces should be operational and have the correct IP addresses.

---

# Part 4: Verify PC Connectivity

PC1 and PC2 should be in **VLAN 10**.

From PC1, ping PC2.

Example:

```text
C:\> ping 192.168.10.12
```

A successful result should look similar to:

```text
Reply from 192.168.10.12: bytes=32 time<1ms TTL=128
Reply from 192.168.10.12: bytes=32 time<1ms TTL=128
Reply from 192.168.10.12: bytes=32 time<1ms TTL=128
Reply from 192.168.10.12: bytes=32 time<1ms TTL=128

Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

You can also verify connectivity between PC1 and the router's VLAN 10 gateway:

```text
C:\> ping 192.168.10.1
```

---

# Part 5: Analyze PC-to-PC Traffic

Switch Packet Tracer to **Simulation Mode**.

From PC1, send a ping to PC2.

Observe the packet as it travels through the network.

### Question

**Is the traffic tagged with a VLAN ID?**

### Answer

**No, not while the traffic is traveling on the access links between the PCs and SW1.**

PC1 and PC2 are connected to access ports assigned to VLAN 10. Frames sent by the PCs are normally **untagged Ethernet frames**.

However, when VLAN 10 traffic travels across the **trunk link between SW1 and R1**, the frame is tagged with an **802.1Q VLAN 10 tag**.

This allows the trunk to distinguish VLAN 10 traffic from VLAN 20 traffic.

---

# Part 6: Analyze IP Phone Traffic

> **Watch the accompanying video before completing this step.**

Switch to **Simulation Mode** and initiate a call from **PH2 to PH1**.

Observe the traffic as it passes through SW1 and R1.

### Question

**Is the traffic tagged with a VLAN ID?**

### Answer

**Yes, when voice traffic crosses the trunk link, it is tagged with VLAN 20.**

The switch ports connected to the phones are configured with:

```cisco
switchport access vlan 10
switchport voice vlan 20
```

This allows the same physical switch port to carry:

- **Untagged data traffic** → VLAN 10
- **Tagged voice traffic** → VLAN 20

The switch uses the voice VLAN configuration to identify and separate IP phone traffic from normal data traffic.

---

# Key Concepts

## Access VLAN vs Voice VLAN

A switch port connected to an IP phone can carry both data and voice traffic.

```cisco
switchport access vlan 10
switchport voice vlan 20
```

| Traffic | VLAN | Typical Frame |
|---|---:|---|
| PC/Data | VLAN 10 | Untagged on access link |
| IP Phone/Voice | VLAN 20 | Voice VLAN tagged |
| Trunk traffic | VLAN 10/20 | 802.1Q tagged |

---

## Router-on-a-Stick

ROAS allows a single physical router interface to route between multiple VLANs by using subinterfaces.

In this lab:

```text
                 802.1Q Trunk
        VLAN 10 + VLAN 20
              │
              │
          SW1 G1/0/1
              │
              │
          R1 Fa0/0
          ├── Fa0/0.10 → VLAN 10 → 192.168.10.1
          └── Fa0/0.20 → VLAN 20 → 192.168.20.1
```

Each router subinterface is associated with a VLAN using:

```cisco
encapsulation dot1Q <VLAN-ID>
```

---

# Final Configuration

## SW1

```cisco
interface GigabitEthernet1/0/1
 switchport trunk allowed vlan 10,20
 switchport mode trunk

interface GigabitEthernet1/0/2
 switchport access vlan 10
 switchport mode access
 switchport voice vlan 20

interface GigabitEthernet1/0/3
 switchport access vlan 10
 switchport mode access
 switchport voice vlan 20
```

## R1

```cisco
interface FastEthernet0/0
 no ip address
 no shutdown

interface FastEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0

interface FastEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
```

---

# Verification Commands

### SW1

```cisco
show vlan brief
show interfaces trunk
show interfaces g1/0/2 switchport
show interfaces g1/0/3 switchport
show running-config
```

### R1

```cisco
show ip interface brief
show running-config
show ip route
```

---

# Lab Questions

1. What VLAN is used for PC/data traffic?

2. What VLAN is used for IP phone/voice traffic?

3. What type of link connects SW1 to R1?

4. Why is `encapsulation dot1Q 10` configured on R1's Fa0/0.10 subinterface?

5. Why is `encapsulation dot1Q 20` configured on R1's Fa0/0.20 subinterface?

6. Is PC-to-PC traffic tagged with a VLAN ID on the access links?

7. Is VLAN 10 traffic tagged when it crosses the SW1-R1 trunk?

8. Is voice traffic tagged with a VLAN ID when it crosses the trunk?

9. Why can an IP phone port carry both data and voice traffic?

---

# Key Takeaways

- **VLAN 10** is used for normal data traffic.
- **VLAN 20** is used for voice traffic.
- SW1's PC/phone ports are configured as **access ports** with a separate voice VLAN.
- The SW1-R1 connection is an **802.1Q trunk**.
- **Router-on-a-Stick** uses router subinterfaces to route traffic between VLANs.
- Data frames from PCs are normally untagged on access links.
- VLAN traffic is **802.1Q tagged across the trunk**.
- Voice traffic uses the configured **voice VLAN (VLAN 20)**.
- Packet Tracer Simulation Mode can be used to observe VLAN tagging and understand how traffic is handled at different points in the network.