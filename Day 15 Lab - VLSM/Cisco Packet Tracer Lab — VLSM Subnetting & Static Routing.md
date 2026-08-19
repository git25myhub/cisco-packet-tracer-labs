# Cisco Packet Tracer Lab — VLSM Subnetting & Static Routing

## 📌 Lab Overview

This lab demonstrates how to subnet a `192.168.5.0/24` network using **Variable Length Subnet Masking (VLSM)** to provide sufficient IP addressing for multiple LANs and a point-to-point connection between two routers.

The lab also demonstrates the configuration of **static routes** on Cisco routers so that devices on different LANs can communicate with each other.

### Objectives

- Subnet the `192.168.5.0/24` network using VLSM.
- Allocate appropriate address space for each LAN.
- Allocate a `/30` subnet for the point-to-point R1–R2 connection.
- Assign the **first usable IP address** to the PC in each LAN.
- Assign the **last usable IP address** to the router interface in each LAN.
- Configure R1 and R2 interfaces.
- Configure static routes on both routers.
- Verify connectivity between PCs on different LANs.

---

## 🗺️ Addressing Plan

The `192.168.5.0/24` network was divided into the following subnets:

| Network | Subnet Mask | Usable Host Range | Broadcast | Purpose |
|---|---|---|---|---|
| `192.168.5.0/25` | `255.255.255.128` | `192.168.5.1 – 192.168.5.126` | `192.168.5.127` | R1 LAN 1 |
| `192.168.5.128/26` | `255.255.255.192` | `192.168.5.129 – 192.168.5.190` | `192.168.5.191` | R1 LAN 2 |
| `192.168.5.192/28` | `255.255.255.240` | `192.168.5.193 – 192.168.5.206` | `192.168.5.207` | R2 LAN 1 |
| `192.168.5.208/28` | `255.255.255.240` | `192.168.5.209 – 192.168.5.222` | `192.168.5.223` | R2 LAN 2 |
| `192.168.5.224/30` | `255.255.255.252` | `192.168.5.225 – 192.168.5.226` | `192.168.5.227` | R1–R2 P2P link |

### Host Addressing Convention

For each LAN:

- **PC:** First usable IP address
- **Router:** Last usable IP address

For the point-to-point link:

- **R1:** `192.168.5.225/30`
- **R2:** `192.168.5.226/30`

---

## 🔢 Device IP Addressing

### R1

| Interface | IP Address | Subnet Mask | Network |
|---|---|---|---|
| G0/0 | `192.168.5.190` | `255.255.255.192` | `192.168.5.128/26` |
| G0/1 | `192.168.5.126` | `255.255.255.128` | `192.168.5.0/25` |
| G0/0/0 | `192.168.5.225` | `255.255.255.252` | `192.168.5.224/30` |

### R2

| Interface | IP Address | Subnet Mask | Network |
|---|---|---|---|
| G0/0 | `192.168.5.206` | `255.255.255.240` | `192.168.5.192/28` |
| G0/1 | `192.168.5.222` | `255.255.255.240` | `192.168.5.208/28` |
| G0/0/0 | `192.168.5.226` | `255.255.255.252` | `192.168.5.224/30` |

### PCs

| PC | IP Address | Subnet Mask | Default Gateway |
|---|---|---|---|
| PC on R1 LAN 1 | `192.168.5.1` | `255.255.255.128` | `192.168.5.126` |
| PC on R1 LAN 2 | `192.168.5.129` | `255.255.255.192` | `192.168.5.190` |
| PC on R2 LAN 1 | `192.168.5.193` | `255.255.255.240` | `192.168.5.206` |
| PC on R2 LAN 2 | `192.168.5.209` | `255.255.255.240` | `192.168.5.222` |

---

# ⚙️ Router Configuration

## R1 Configuration

### LAN Interfaces

```cisco
R1(config)#interface gigabitEthernet0/0
R1(config-if)#ip address 192.168.5.190 255.255.255.192
R1(config-if)#no shutdown

R1(config)#interface gigabitEthernet0/1
R1(config-if)#ip address 192.168.5.126 255.255.255.128
R1(config-if)#no shutdown
```

### Point-to-Point Interface

```cisco
R1(config)#interface gigabitEthernet0/0/0
R1(config-if)#ip address 192.168.5.225 255.255.255.252
R1(config-if)#no shutdown
```

### Static Routes

R1 needs routes to the two LANs behind R2:

```cisco
R1(config)#ip route 192.168.5.192 255.255.255.240 192.168.5.226
R1(config)#ip route 192.168.5.208 255.255.255.240 192.168.5.226
```

---

## R2 Configuration

### LAN Interfaces

```cisco
R2(config)#interface gigabitEthernet0/0
R2(config-if)#ip address 192.168.5.206 255.255.255.240
R2(config-if)#no shutdown

R2(config)#interface gigabitEthernet0/1
R2(config-if)#ip address 192.168.5.222 255.255.255.240
R2(config-if)#no shutdown
```

### Point-to-Point Interface

```cisco
R2(config)#interface gigabitEthernet0/0/0
R2(config-if)#ip address 192.168.5.226 255.255.255.252
R2(config-if)#no shutdown
```

### Static Routes

R2 needs routes to the two LANs behind R1:

```cisco
R2(config)#ip route 192.168.5.128 255.255.255.192 192.168.5.225
R2(config)#ip route 192.168.5.0 255.255.255.128 192.168.5.225
```

---

# 🧭 Routing Logic

The topology can be understood as:

```text
                         R1 ↔ R2
                    192.168.5.225
                    192.168.5.226
                         /   \
                        /     \
                 R1 LANs     R2 LANs

192.168.5.0/25                192.168.5.192/28
192.168.5.128/26              192.168.5.208/28
```

R1 is directly connected to:

- `192.168.5.0/25`
- `192.168.5.128/26`
- `192.168.5.224/30`

R2 is directly connected to:

- `192.168.5.192/28`
- `192.168.5.208/28`
- `192.168.5.224/30`

Therefore, each router requires static routes for the networks that are **not directly connected**.

---

# 🔍 Verification

## Verify R1 Routing Table

The following command was used:

```cisco
R1#show ip route
```

R1 correctly showed its directly connected networks and static routes:

```text
C 192.168.5.0/25 is directly connected
C 192.168.5.128/26 is directly connected
S 192.168.5.192/28 [1/0] via 192.168.5.226
S 192.168.5.208/28 [1/0] via 192.168.5.226
C 192.168.5.224/30 is directly connected
```

This confirms that R1 knows how to reach both LANs behind R2.

---

## Verify R2 Routing Table

The following command was used:

```cisco
R2#show ip route
```

R2 correctly showed:

```text
S 192.168.5.0/25 [1/0] via 192.168.5.225
S 192.168.5.128/26 [1/0] via 192.168.5.225
C 192.168.5.192/28 is directly connected
C 192.168.5.208/28 is directly connected
C 192.168.5.224/30 is directly connected
```

This confirms that R2 knows how to reach both LANs behind R1.

---

# 🧪 Connectivity Test

A PC was used to test connectivity to the remote R2 LAN:

```text
C:\>ping 192.168.5.209
```

The first test produced two timeouts followed by successful replies:

```text
Request timed out.
Request timed out.
Reply from 192.168.5.209: bytes=32 time<1ms TTL=126
Reply from 192.168.5.209: bytes=32 time=1ms TTL=126
```

A second ping test produced:

```text
Reply from 192.168.5.209: bytes=32 time=1ms TTL=126
Reply from 192.168.5.209: bytes=32 time<1ms TTL=126
Reply from 192.168.5.209: bytes=32 time<1ms TTL=126
Reply from 192.168.5.209: bytes=32 time<1ms TTL=126
```

### Result

```text
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

The successful `0%` packet loss confirms that end-to-end connectivity between the LANs is working.

The initial timeout is normal in Packet Tracer because the first packet may require ARP resolution before subsequent packets are forwarded successfully.

---

# 💾 Saving the Configuration

Both routers were saved using:

```cisco
R1#write memory
R2#write memory
```

or:

```cisco
R1#do write
R2#do write
```

The routers returned:

```text
Building configuration...
[OK]
```

This confirms that the configurations were successfully saved.

---

# 🧠 Key Concepts Demonstrated

This lab demonstrates several important networking concepts:

- **VLSM subnetting**
- **CIDR notation**
- **Subnet masks**
- **Network, usable host, and broadcast addresses**
- **Point-to-point `/30` networks**
- **Cisco router interface configuration**
- **Static routing**
- **Next-hop addresses**
- **Default gateways**
- **Routing table verification**
- **ICMP ping testing**
- **ARP-based first-hop resolution**
- **End-to-end connectivity troubleshooting**

---

# ✅ Lab Outcome

The `192.168.5.0/24` network was successfully subnetted into appropriately sized networks for the required LANs and the R1–R2 point-to-point connection.

Both routers were configured with their respective LAN and point-to-point interfaces, and static routes were added to provide reachability between remote networks.

Connectivity was successfully verified using ICMP ping, with the final test achieving:

**4 packets sent, 4 received, 0% packet loss.**

This confirms that the configured addressing and static routing are functioning correctly.