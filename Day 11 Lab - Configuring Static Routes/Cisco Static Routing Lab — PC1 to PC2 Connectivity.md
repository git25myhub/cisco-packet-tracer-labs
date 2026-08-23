# Cisco Static Routing Lab — Static Routing Between PCs

## Lab Objective

The objective of this lab is to configure a small routed network from scratch and use **static routes** to provide end-to-end connectivity between PC1 and PC2.

You will configure:

- Hostnames on the routers and PCs
- IP addresses on the router interfaces
- IP addresses on the PCs
- Default gateways on the PCs
- Static routes on the routers
- End-to-end connectivity using `ping`

> **Note:** All devices start with **no pre-configurations**. The switches do not require configuration for this lab.

---

## Network Topology

The lab consists of three routers connected in a triangle:

```text
        192.168.12.0/24
   R1 ---------------- R2
   |                    |
   |                    | 192.168.13.0/24
   |                    |
   |                    R3
   |                    |
 PC1                  PC2
192.168.1.0/24       192.168.3.0/24
```

The routers form three networks:

- `192.168.1.0/24` — PC1 LAN
- `192.168.12.0/24` — R1 ↔ R2
- `192.168.13.0/24` — R2 ↔ R3
- `192.168.3.0/24` — PC2 LAN

---

## Addressing Table

| Device | Interface | IP Address | Subnet Mask | Default Gateway |
|---|---|---|---|---|
| PC1 | NIC | `192.168.1.1` | `255.255.255.0` | `192.168.1.254` |
| R1 | G0/0 | `192.168.12.1` | `255.255.255.0` | N/A |
| R1 | G0/1 | `192.168.1.254` | `255.255.255.0` | N/A |
| R2 | G0/0 | `192.168.12.2` | `255.255.255.0` | N/A |
| R2 | G0/1 | `192.168.13.2` | `255.255.255.0` | N/A |
| R3 | G0/0 | `192.168.13.3` | `255.255.255.0` | N/A |
| R3 | G0/1 | `192.168.3.254` | `255.255.255.0` | N/A |
| PC2 | NIC | `192.168.3.1` | `255.255.255.0` | `192.168.3.254` |

---

# Lab Tasks

## 1. Configure the Routers

Configure the appropriate hostname and IP addresses on R1, R2, and R3.

### R1

```cisco
enable
configure terminal

hostname R1

interface GigabitEthernet0/0
 ip address 192.168.12.1 255.255.255.0
 no shutdown
exit

interface GigabitEthernet0/1
 ip address 192.168.1.254 255.255.255.0
 no shutdown
exit

end
```

Verify the interfaces:

```cisco
show ip interface brief
```

---

### R2

```cisco
enable
configure terminal

hostname R2

interface GigabitEthernet0/0
 ip address 192.168.12.2 255.255.255.0
 no shutdown
exit

interface GigabitEthernet0/1
 ip address 192.168.13.2 255.255.255.0
 no shutdown
exit

end
```

Verify:

```cisco
show ip interface brief
```

---

### R3

```cisco
enable
configure terminal

hostname R3

interface GigabitEthernet0/0
 ip address 192.168.13.3 255.255.255.0
 no shutdown
exit

interface GigabitEthernet0/1
 ip address 192.168.3.254 255.255.255.0
 no shutdown
exit

end
```

Verify:

```cisco
show ip interface brief
```

---

# 2. Configure PC1

Configure PC1 with:

```text
IP Address:      192.168.1.1
Subnet Mask:     255.255.255.0
Default Gateway: 192.168.1.254
```

In Packet Tracer:

**PC1 → Desktop → IP Configuration**

Enter the values above.

Test connectivity to R1:

```text
ping 192.168.1.254
```

The ping should succeed.

---

# 3. Configure PC2

Configure PC2 with:

```text
IP Address:      192.168.3.1
Subnet Mask:     255.255.255.0
Default Gateway: 192.168.3.254
```

In Packet Tracer:

**PC2 → Desktop → IP Configuration**

Enter the values above.

Test connectivity to R3:

```text
ping 192.168.3.254
```

The ping should succeed.

---

# 4. Configure Static Routing

The routers only know about their directly connected networks. Static routes must therefore be configured so that traffic can travel from the PC1 network to the PC2 network.

## R1 Static Route

R1 needs a route to the `192.168.3.0/24` network.

The next-hop router is R2 at `192.168.12.2`.

```cisco
R1# configure terminal
R1(config)# ip route 192.168.3.0 255.255.255.0 192.168.12.2
R1(config)# end
```

Verify:

```cisco
R1# show ip route
```

You should see a static route similar to:

```text
S    192.168.3.0/24 [1/0] via 192.168.12.2
```

---

## R2 Static Routes

R2 needs routes to both end-user LANs.

### Route to PC1's network

```cisco
R2# configure terminal
R2(config)# ip route 192.168.1.0 255.255.255.0 192.168.12.1
```

### Route to PC2's network

```cisco
R2(config)# ip route 192.168.3.0 255.255.255.0 192.168.13.3
R2(config)# end
```

Verify:

```cisco
R2# show ip route
```

Expected static routes:

```text
S    192.168.1.0/24 [1/0] via 192.168.12.1
S    192.168.3.0/24 [1/0] via 192.168.13.3
```

---

## R3 Static Route

R3 needs a route back to the `192.168.1.0/24` network.

The next-hop router is R2 at `192.168.13.2`.

```cisco
R3# configure terminal
R3(config)# ip route 192.168.1.0 255.255.255.0 192.168.13.2
R3(config)# end
```

Verify:

```cisco
R3# show ip route
```

Expected:

```text
S    192.168.1.0/24 [1/0] via 192.168.13.2
```

---

# 5. Verify Routing

Check the routing table on each router:

```cisco
show ip route
```

You should see:

### R1

```text
Connected:
192.168.1.0/24
192.168.12.0/24

Static:
192.168.3.0/24 via 192.168.12.2
```

### R2

```text
Connected:
192.168.12.0/24
192.168.13.0/24

Static:
192.168.1.0/24 via 192.168.12.1
192.168.3.0/24 via 192.168.13.3
```

### R3

```text
Connected:
192.168.13.0/24
192.168.3.0/24

Static:
192.168.1.0/24 via 192.168.13.2
```

---

# 6. Test End-to-End Connectivity

From **PC1**, ping PC2:

```text
ping 192.168.3.1
```

Expected result:

```text
Reply from 192.168.3.1: bytes=32 time<1ms TTL=...
Reply from 192.168.3.1: bytes=32 time<1ms TTL=...
Reply from 192.168.3.1: bytes=32 time<1ms TTL=...
Reply from 192.168.3.1: bytes=32 time<1ms TTL=...
```

The important requirement is:

> **PC1 must successfully ping PC2.**

---

# 7. Additional Verification

Test each hop individually.

From PC1:

```text
ping 192.168.12.2
ping 192.168.13.2
ping 192.168.13.3
ping 192.168.3.254
ping 192.168.3.1
```

You can also use `tracert` from PC1:

```text
tracert 192.168.3.1
```

This should show the path through the routers:

```text
PC1 → R1 → R2 → R3 → PC2
```

---

# 8. Troubleshooting

If PC1 cannot ping PC2, check the following.

### Check PC addressing

On PC1:

```text
IP:      192.168.1.1
Mask:    255.255.255.0
Gateway: 192.168.1.254
```

On PC2:

```text
IP:      192.168.3.1
Mask:    255.255.255.0
Gateway: 192.168.3.254
```

### Check router interfaces

On each router:

```cisco
show ip interface brief
```

All required interfaces should show:

```text
Status: up
Protocol: up
```

If an interface is administratively down:

```cisco
configure terminal
interface GigabitEthernet0/x
no shutdown
end
```

### Check static routes

```cisco
show ip route
```

Confirm that the destination networks and next-hop addresses are correct.

### Check connectivity between routers

From R1:

```cisco
ping 192.168.12.2
```

From R2:

```cisco
ping 192.168.12.1
ping 192.168.13.3
```

From R3:

```cisco
ping 192.168.13.2
```

---

# 9. Save the Configuration

Once the lab is working, save the configurations on all routers.

```cisco
copy running-config startup-config
```

Press **Enter** when prompted for the destination filename.

Alternatively:

```cisco
write memory
```

---

# Expected Final Configuration

## R1

```cisco
hostname R1

interface GigabitEthernet0/0
 ip address 192.168.12.1 255.255.255.0
 no shutdown

interface GigabitEthernet0/1
 ip address 192.168.1.254 255.255.255.0
 no shutdown

ip route 192.168.3.0 255.255.255.0 192.168.12.2
```

## R2

```cisco
hostname R2

interface GigabitEthernet0/0
 ip address 192.168.12.2 255.255.255.0
 no shutdown

interface GigabitEthernet0/1
 ip address 192.168.13.2 255.255.255.0
 no shutdown

ip route 192.168.1.0 255.255.255.0 192.168.12.1
ip route 192.168.3.0 255.255.255.0 192.168.13.3
```

## R3

```cisco
hostname R3

interface GigabitEthernet0/0
 ip address 192.168.13.3 255.255.255.0
 no shutdown

interface GigabitEthernet0/1
 ip address 192.168.3.254 255.255.255.0
 no shutdown

ip route 192.168.1.0 255.255.255.0 192.168.13.2
```

---

# Verification Checklist

- [ ] Configure PC1 IP address and subnet mask
- [ ] Configure PC1 default gateway
- [ ] Configure PC2 IP address and subnet mask
- [ ] Configure PC2 default gateway
- [ ] Configure R1 hostname
- [ ] Configure R2 hostname
- [ ] Configure R3 hostname
- [ ] Configure all required router interfaces
- [ ] Enable all required router interfaces with `no shutdown`
- [ ] Configure R1 static route to `192.168.3.0/24`
- [ ] Configure R2 static route to `192.168.1.0/24`
- [ ] Configure R2 static route to `192.168.3.0/24`
- [ ] Configure R3 static route to `192.168.1.0/24`
- [ ] Verify routing tables with `show ip route`
- [ ] Verify router-to-router connectivity
- [ ] Verify PC1 can ping its default gateway
- [ ] Verify PC2 can ping its default gateway
- [ ] Verify PC1 can successfully ping PC2
- [ ] Save the router configurations

---

## Key Concepts Practiced

This lab provides hands-on practice with:

- Basic Cisco IOS configuration
- Router interface configuration
- IPv4 addressing
- Default gateways
- Directly connected networks
- Static routing
- Next-hop addresses
- Routing table verification
- End-to-end connectivity testing
- `ping` and `tracert`
- Cisco IOS configuration verification
- Saving configurations to NVRAM

## Final Goal

The lab is successfully completed when:

```text
PC1 (192.168.1.1)
       |
      R1
       |
      R2
       |
      R3
       |
PC2 (192.168.3.1)
```

has full end-to-end connectivity and:

```text
PC1> ping 192.168.3.1
```

returns successful replies.