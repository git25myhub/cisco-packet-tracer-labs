# Cisco Packet Tracer Lab: Default Route and DNS Configuration

## Lab Objectives

1. Configure a default route to the Internet on R1.
2. Configure PC1, PC2, and PC3 to use **1.1.1.1** as their DNS server.
3. Configure R1 to use **1.1.1.1** as its DNS server.
4. Configure local host entries on R1 for R1, PC1, PC2, and PC3.
5. Verify name resolution by pinging PC1 by hostname from R1.
6. Use **Simulation Mode** to observe DNS and ICMP traffic when PC1 pings `youtube.com`.

---

## Network Addressing

| Device | Interface | IP Address | Subnet Mask | Gateway |
|---|---|---|---|---|
| R1 | G0/0 | 203.0.113.1 | 255.255.255.252 | 203.0.113.2 |
| R1 | G0/1 | 192.168.0.254 | 255.255.255.0 | — |
| PC1 | — | 192.168.0.1 | 255.255.255.0 | 192.168.0.254 |
| PC2 | — | 192.168.0.2 | 255.255.255.0 | 192.168.0.254 |
| PC3 | — | 192.168.0.3 | 255.255.255.0 | 192.168.0.254 |
| Internet/Next Hop | — | 203.0.113.2 | 255.255.255.252 | — |
| DNS Server | — | 1.1.1.1 | — | — |

---

## 1. Configure the Default Route on R1

The first attempt used an incorrect next-hop address:

```text
R1(config)#ip route 0.0.0.0 0.0.0.0 203.0.1.113.2
                                    ^
% Invalid input detected at '^' marker.
```

The correct next-hop address is **203.0.113.2**.

```text
R1(config)#ip route 0.0.0.0 0.0.0.0 203.0.113.2
```

The default route provides a path for traffic destined for networks that are not present in R1's routing table.

### Verify Internet Connectivity

```text
R1(config)#do ping 1.1.1.1
```

Result:

```text
Sending 5, 100-byte ICMP Echos to 1.1.1.1, timeout is 2 seconds:

..!!!

Success rate is 60 percent (3/5)
```

The successful replies confirm that R1 can reach the DNS server through the configured default route.

---

## 2. Configure DNS on the PCs

PC1, PC2, and PC3 should each be configured with:

```text
DNS Server: 1.1.1.1
```

The PCs should also use R1's LAN address as their default gateway:

```text
Default Gateway: 192.168.0.254
```

### PC Addressing

**PC1**

```text
IP Address:      192.168.0.1
Subnet Mask:     255.255.255.0
Default Gateway: 192.168.0.254
DNS Server:      1.1.1.1
```

**PC2**

```text
IP Address:      192.168.0.2
Subnet Mask:     255.255.255.0
Default Gateway: 192.168.0.254
DNS Server:      1.1.1.1
```

**PC3**

```text
IP Address:      192.168.0.3
Subnet Mask:     255.255.255.0
Default Gateway: 192.168.0.254
DNS Server:      1.1.1.1
```

---

## 3. Configure DNS and Host Entries on R1

R1 was configured to use **1.1.1.1** as its DNS server:

```text
R1(config)#ip name-server 1.1.1.1
```

Local hostname entries were then configured:

```text
R1(config)#ip host PC1 192.168.0.1
R1(config)#ip host PC2 192.168.0.2
R1(config)#ip host PC3 192.168.0.3
R1(config)#ip host R1 192.168.0.254
```

These entries allow R1 to resolve the local device names without relying on an external DNS server for those addresses.

### Verify Host Entries

```text
R1(config)#do show hosts
```

Expected entries:

```text
Host                      Port  Flags      Age Type   Address(es)

PC1                       None  (perm, OK)  0   IP      192.168.0.1
PC2                       None  (perm, OK)  0   IP      192.168.0.2
PC3                       None  (perm, OK)  0   IP      192.168.0.3
R1                        None  (perm, OK)  0   IP      192.168.0.254
```

---

## 4. Test Name Resolution from R1

R1 was used to ping PC1 by hostname:

```text
R1(config)#do ping PC1
```

Result:

```text
Sending 5, 100-byte ICMP Echos to 192.168.0.1, timeout is 2 seconds:

.!!!!

Success rate is 80 percent (4/5)
```

This confirms that R1 successfully resolved **PC1** to **192.168.0.1** and was able to reach the device.

---

## 5. Verify the Running Configuration

The important portions of R1's final configuration are:

```text
hostname R1

ip host PC1 192.168.0.1
ip host PC2 192.168.0.2
ip host PC3 192.168.0.3
ip host R1 192.168.0.254
ip name-server 1.1.1.1

interface GigabitEthernet0/0
 ip address 203.0.113.1 255.255.255.252

interface GigabitEthernet0/1
 ip address 192.168.0.254 255.255.255.0

ip classless
ip route 0.0.0.0 0.0.0.0 203.0.113.2
```

The configuration was saved using:

```text
R1(config)#do wr
```

---

## 6. Test `youtube.com` from PC1

**IMPORTANT: Use Simulation Mode for this step.**

From PC1, open the command prompt and run:

```text
C:\>ping youtube.com
```

Observed result:

```text
Pinging 172.217.6.78 with 32 bytes of data:

Request timed out.
Reply from 172.217.6.78: bytes=32 time<1ms TTL=126
Reply from 172.217.6.78: bytes=32 time<1ms TTL=126
Reply from 172.217.6.78: bytes=32 time<1ms TTL=126

Ping statistics for 172.217.6.78:
    Packets: Sent = 4, Received = 3, Lost = 1 (25% loss)
```

The important observation is that `youtube.com` was first resolved to the IP address:

```text
172.217.6.78
```

The PC then sent ICMP echo requests toward that IP address.

---

## 7. Simulation Mode Analysis

When PC1 executes:

```text
ping youtube.com
```

two important processes can be observed.

### Step 1 — DNS Resolution

PC1 does not initially know the IP address of `youtube.com`.

It therefore sends a DNS query to:

```text
1.1.1.1
```

The DNS server responds with an IP address for `youtube.com`.

In this lab, the resolved address was:

```text
172.217.6.78
```

### Step 2 — ICMP Ping

After DNS resolution, PC1 sends ICMP Echo Request packets toward:

```text
172.217.6.78
```

The destination responds with ICMP Echo Reply packets.

### Traffic Flow

The overall communication can be summarized as:

```text
PC1
 |
 | DNS Query: youtube.com
 v
1.1.1.1
 |
 | DNS Response: 172.217.6.78
 v
PC1
 |
 | ICMP Echo Request
 v
Default Gateway (R1)
 |
 | Default Route
 | 0.0.0.0/0 via 203.0.113.2
 v
Internet
 |
 v
172.217.6.78
 |
 | ICMP Echo Reply
 v
PC1
```

---

## Key Commands Used

### Configure Default Route

```text
ip route 0.0.0.0 0.0.0.0 203.0.113.2
```

### Configure DNS Server

```text
ip name-server 1.1.1.1
```

### Configure Host Entries

```text
ip host PC1 192.168.0.1
ip host PC2 192.168.0.2
ip host PC3 192.168.0.3
ip host R1 192.168.0.254
```

### Verify Host Entries

```text
show hosts
```

### Test DNS/Host Resolution

```text
ping PC1
```

### Test Internet Connectivity

```text
ping 1.1.1.1
```

### Save Configuration

```text
write memory
```

or:

```text
do wr
```

---

## Troubleshooting Note

An incorrect next-hop address was initially entered:

```text
203.0.1.113.2
```

This produced:

```text
% Invalid input detected at '^' marker.
```

The correct address was:

```text
203.0.113.2
```

After correcting the address, R1 successfully reached `1.1.1.1`.

---

## Conclusion

This lab demonstrated how to configure a Cisco router with a **default route**, configure **DNS services**, create **local hostname-to-IP mappings**, and verify name resolution.

The final test demonstrated the difference between **DNS resolution** and **ICMP communication**. When PC1 pinged `youtube.com`, DNS first translated the hostname into an IP address. PC1 then used the resulting IP address to generate ICMP traffic. Simulation Mode makes these DNS and ICMP packets visible, allowing the complete communication process to be analyzed.

**Final verification:**

- Default route configured on R1: **Yes**
- R1 DNS server configured as `1.1.1.1`: **Yes**
- PC DNS server configured as `1.1.1.1`: **Yes**
- R1 host entries configured: **Yes**
- R1 successfully pinged PC1 by name: **Yes**
- PC1 successfully resolved `youtube.com`: **Yes**
- ICMP traffic observed to the resolved IP: **Yes**