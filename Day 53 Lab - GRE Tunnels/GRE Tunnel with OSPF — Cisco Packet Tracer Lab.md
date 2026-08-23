# GRE Tunnel with OSPF — Cisco Packet Tracer Lab

## 📌 Lab Overview

In this lab, a **GRE (Generic Routing Encapsulation) tunnel** is configured between **R1 and R2** to provide a logical point-to-point connection across the routed network.

After establishing the GRE tunnel, **OSPF** is configured on the tunnel interfaces and LAN interfaces so that PC1 and PC2 can communicate across the routers.

### Objectives

- Configure a GRE tunnel between R1 and R2.
- Assign IP addresses to the GRE tunnel interfaces.
- Configure static default routes to provide reachability to the GRE tunnel endpoints.
- Configure OSPF Area 0 across the GRE tunnel.
- Advertise the LAN networks through OSPF.
- Verify OSPF neighbor adjacency.
- Verify that PC1 and PC2 can communicate.

---

## 🗺️ Network Addressing

| Device | Interface | IP Address | Purpose |
|---|---|---|---|
| R1 | G0/0 | `10.0.1.1/24` | PC1 LAN |
| R1 | G0/0/0 | `100.0.0.2/30` | GRE source/WAN |
| R1 | Tunnel0 | `192.168.1.1/30` | GRE tunnel |
| R2 | G0/0 | `10.0.2.1/24` | PC2 LAN |
| R2 | G0/0/0 | `200.0.0.2/30` | GRE source/WAN |
| R2 | Tunnel0 | `192.168.1.2/30` | GRE tunnel |

### GRE Tunnel

```text
R1 Tunnel0: 192.168.1.1/30
        |
        | GRE Tunnel
        |
R2 Tunnel0: 192.168.1.2/30
```

### Routing

```text
R1 default route → 100.0.0.1
R2 default route → 200.0.0.1

OSPF Area 0:
10.0.1.0/24 ←→ 10.0.2.0/24
```

---

## 1. Configure GRE Tunnel on R1

Configure Tunnel0 with an IP address and specify the physical interface as the tunnel source and R2's WAN address as the tunnel destination.

```cisco
R1(config)#interface Tunnel0
R1(config-if)#ip address 192.168.1.1 255.255.255.252
R1(config-if)#tunnel source GigabitEthernet0/0/0
R1(config-if)#tunnel destination 200.0.0.2
```

The tunnel configuration should result in:

```text
Tunnel0
IP Address: 192.168.1.1/30
Source: GigabitEthernet0/0/0
Destination: 200.0.0.2
```

---

## 2. Configure GRE Tunnel on R2

Configure Tunnel0 on R2 using the opposite tunnel IP address.

```cisco
R2(config)#interface Tunnel0
R2(config-if)#ip address 192.168.1.2 255.255.255.252
R2(config-if)#tunnel source GigabitEthernet0/0/0
R2(config-if)#tunnel destination 100.0.0.2
```

The tunnel endpoints are now:

```text
R1: 192.168.1.1
R2: 192.168.1.2
```

---

## 3. Configure Default Routes

The routers must be able to reach each other's GRE tunnel destination through the underlying WAN network.

### R1

```cisco
R1(config)#ip route 0.0.0.0 0.0.0.0 100.0.0.1
```

### R2

```cisco
R2(config)#ip route 0.0.0.0 0.0.0.0 200.0.0.1
```

Verify the routing table:

```cisco
R1#show ip route
R2#show ip route
```

---

## 4. Verify the GRE Tunnel

Before configuring OSPF, verify that the GRE tunnel is operational.

### R1

```cisco
R1#ping 192.168.1.2
```

Expected result:

```text
Success rate should be high/100% once the tunnel is established.
```

### R2

```cisco
R2#ping 192.168.1.1
```

The first ping may fail while the tunnel is being established. Subsequent pings should succeed.

Check the tunnel interface:

```cisco
R1#show ip interface brief
R2#show ip interface brief
```

The tunnel should eventually show:

```text
Tunnel0    192.168.1.x    YES    manual    up    up
```

---

## 5. Configure OSPF on R1

Create OSPF process 1 and advertise both the R1 LAN network and GRE tunnel address into Area 0.

```cisco
R1(config)#router ospf 1
R1(config-router)#network 192.168.1.1 0.0.0.0 area 0
R1(config-router)#network 10.0.1.1 0.0.0.0 area 0
R1(config-router)#passive-interface GigabitEthernet0/0
```

The LAN interface is configured as passive because R1 should advertise the LAN network but should not form an OSPF neighbor relationship with a PC.

---

## 6. Configure OSPF on R2

Configure the same OSPF process and Area 0 on R2.

```cisco
R2(config)#router ospf 1
R2(config-router)#network 192.168.1.2 0.0.0.0 area 0
R2(config-router)#network 10.0.2.1 0.0.0.0 area 0
R2(config-router)#passive-interface GigabitEthernet0/0
```

OSPF should now form a neighbor relationship between R1 and R2 across Tunnel0.

---

## 7. Verify OSPF Neighbor Adjacency

On R1:

```cisco
R1#show ip ospf neighbor
```

On R2:

```cisco
R2#show ip ospf neighbor
```

The neighbor should reach the **FULL** state.

The lab output showed:

```text
%OSPF-5-ADJCHG: Process 1, Nbr 200.0.0.2 on Tunnel0
from LOADING to FULL, Loading Done
```

This confirms that the OSPF adjacency was successfully established across the GRE tunnel.

---

## 8. Verify OSPF Routes

On R1:

```cisco
R1#show ip route
```

R1 should learn the R2 LAN through OSPF:

```text
O 10.0.2.0/24 via 192.168.1.2, Tunnel0
```

On R2:

```cisco
R2#show ip route
```

R2 should learn the R1 LAN through OSPF:

```text
O 10.0.1.0/24 via 192.168.1.1, Tunnel0
```

The `O` routing code indicates that the route was learned through OSPF.

---

## 9. Test End-to-End Connectivity

From PC1, ping the PC2 address:

```text
C:\>ping 10.0.2.100
```

Expected result:

```text
Reply from 10.0.2.100
Reply from 10.0.2.100
Reply from 10.0.2.100
```

The first packet may time out while ARP and routing information are being resolved. Subsequent packets should succeed.

A successful ping confirms the complete path:

```text
PC1
 |
 | 10.0.1.0/24
 |
R1
 |
 | GRE Tunnel
 | 192.168.1.0/30
 |
R2
 |
 | 10.0.2.0/24
 |
PC2
```

---

## 🔍 Verification Commands

### Check Interfaces

```cisco
show ip interface brief
```

### Check GRE Tunnel

```cisco
show interface Tunnel0
```

### Check Routing Table

```cisco
show ip route
```

### Check OSPF Neighbors

```cisco
show ip ospf neighbor
```

### Check OSPF Configuration

```cisco
show ip protocols
```

### Check OSPF Routes

```cisco
show ip route ospf
```

### Test GRE Connectivity

```cisco
ping 192.168.1.1
ping 192.168.1.2
```

### Test LAN Connectivity

```text
PC1> ping 10.0.2.100
PC2> ping 10.0.1.100
```

---

## 💾 Save the Configuration

Save the configuration on both routers:

```cisco
R1#copy running-config startup-config
R2#copy running-config startup-config
```

or:

```cisco
R1#do write
R2#do write
```

---

## 🧠 Key Concepts Learned

### GRE

GRE creates a **logical tunnel interface** between two routers. It allows Layer 3 traffic, including routing protocols such as OSPF, to travel through the tunnel.

### OSPF over GRE

OSPF treats the GRE tunnel as a logical point-to-point connection. This allows R1 and R2 to exchange routes without requiring OSPF to operate directly over the underlying WAN network.

### Passive Interface

```cisco
passive-interface GigabitEthernet0/0
```

allows the LAN network to be advertised through OSPF while preventing OSPF neighbor formation on the LAN interface.

### Important Troubleshooting Point

A GRE tunnel depends on **underlying IP reachability between the tunnel source and destination**.

For this lab:

```text
R1 tunnel destination: 200.0.0.2
R2 tunnel destination: 100.0.0.2
```

Therefore, the routers must first be able to reach these addresses through the WAN network before the GRE tunnel can come up.

---

## ✅ Lab Completion Checklist

- [x] Configure GRE Tunnel0 on R1.
- [x] Configure GRE Tunnel0 on R2.
- [x] Configure tunnel IP addresses `192.168.1.1/30` and `192.168.1.2/30`.
- [x] Configure GRE source and destination addresses.
- [x] Configure default routes for GRE endpoint reachability.
- [x] Verify Tunnel0 is up/up.
- [x] Configure OSPF process 1.
- [x] Advertise the GRE tunnel interfaces in OSPF Area 0.
- [x] Advertise both LAN networks through OSPF.
- [x] Configure the LAN interfaces as passive OSPF interfaces.
- [x] Verify the OSPF neighbor reaches FULL state.
- [x] Verify OSPF-learned routes.
- [x] Ping PC2 from PC1 successfully.

---

## 🎯 Final Result

The lab successfully demonstrates how **GRE and OSPF can work together** to connect two remote LANs.

R1 and R2 establish a GRE tunnel using their WAN interfaces, then form an OSPF adjacency across the tunnel. OSPF dynamically exchanges the LAN routes, allowing **PC1 and PC2 to communicate across the GRE tunnel**.