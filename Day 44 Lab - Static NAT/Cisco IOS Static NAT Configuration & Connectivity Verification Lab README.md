# Cisco IOS Static NAT Configuration & Connectivity Verification

## 📌 Lab Overview

This lab demonstrates how to configure **Static Network Address Translation (NAT)** on a Cisco router and verify connectivity from internal hosts to external networks.

The lab focuses on:

- Verifying connectivity before NAT configuration.
- Configuring inside and outside NAT interfaces.
- Creating static NAT mappings.
- Configuring a default route toward the external network.
- Verifying NAT translations and statistics.
- Testing DNS resolution and Internet connectivity.
- Clearing dynamic NAT translation entries.
- Saving and verifying the router configuration.

---

## 🎯 Objectives

By completing this lab, you should be able to:

1. Identify the inside and outside interfaces used for NAT.
2. Configure Cisco IOS interfaces for NAT.
3. Configure static one-to-one NAT mappings.
4. Configure a default route for external connectivity.
5. Verify static NAT entries using `show ip nat translations`.
6. Monitor NAT statistics using `show ip nat statistics`.
7. Verify that ICMP and DNS traffic is being translated.
8. Clear NAT translation entries.
9. Test connectivity using IP addresses and DNS names.
10. Save and verify the router configuration.

---

## 🗺️ Network Addressing

| Device | Interface | IP Address | Role |
|---|---|---|---|
| R1 | G0/0 | `203.0.113.1/30` | NAT Outside |
| R1 | G0/1 | `172.16.0.254/24` | NAT Inside |
| R1 | G0/2 | Unused | Shutdown |
| R1 | Default Route | `203.0.113.2` | External Gateway |
| Internal Host 1 | — | `172.16.0.1` | Inside Local |
| Internal Host 2 | — | `172.16.0.2` | Inside Local |
| Internal Host 3 | — | `172.16.0.3` | Inside Local |
| NAT Address 1 | — | `100.0.0.1` | Inside Global |
| NAT Address 2 | — | `100.0.0.2` | Inside Global |
| NAT Address 3 | — | `100.0.0.3` | Inside Global |

### Static NAT Mapping

The following translations were configured:

| Inside Local | Inside Global |
|---|---|
| `172.16.0.1` | `100.0.0.1` |
| `172.16.0.2` | `100.0.0.2` |
| `172.16.0.3` | `100.0.0.3` |

This means that:

- Host `172.16.0.1` is represented externally as `100.0.0.1`.
- Host `172.16.0.2` is represented externally as `100.0.0.2`.
- Host `172.16.0.3` is represented externally as `100.0.0.3`.

---

# 1. Initial Connectivity Test

Before configuring NAT, connectivity to the external IP address was tested from the internal network.

```text
C:\>ping 8.8.8.8

Pinging 8.8.8.8 with 32 bytes of data:

Request timed out.
Request timed out.
Request timed out.
Request timed out.

Packets: Sent = 4, Received = 0, Lost = 4 (100% loss)
```

The initial test resulted in **100% packet loss**.

This provided a baseline demonstrating that external connectivity was not yet working correctly.

---

# 2. Configure NAT Interfaces

R1's interfaces were configured to identify which side of the router represents the internal network and which side represents the external network.

### Configure the outside interface

```cisco
R1# configure terminal
R1(config)# interface g0/0
R1(config-if)# ip nat outside
```

### Configure the inside interface

```cisco
R1(config-if)# interface g0/1
R1(config-if)# ip nat inside
R1(config-if)# exit
```

The resulting roles are:

```text
G0/1 → NAT INSIDE
G0/0 → NAT OUTSIDE
```

---

# 3. Configure Static NAT

Static NAT mappings were configured using the following commands:

```cisco
R1(config)# ip nat inside source static 172.16.0.1 100.0.0.1
R1(config)# ip nat inside source static 172.16.0.2 100.0.0.2
R1(config)# ip nat inside source static 172.16.0.3 100.0.0.3
```

Each internal address is permanently mapped to a corresponding global address.

### NAT Translation Table

```text
Inside Local     Inside Global
172.16.0.1  →    100.0.0.1
172.16.0.2  →    100.0.0.2
172.16.0.3  →    100.0.0.3
```

---

# 4. Configure the Default Route

R1 needs a route toward the external network.

The configured default route is:

```cisco
R1(config)# ip route 0.0.0.0 0.0.0.0 203.0.113.2
```

This tells R1 to forward traffic destined for networks that are not present in its routing table to `203.0.113.2`.

---

# 5. Save the Configuration

The configuration was saved using:

```cisco
R1# write
```

Expected output:

```text
Building configuration...

[OK]
```

The equivalent command is:

```cisco
R1# copy running-config startup-config
```

Saving the configuration ensures that the NAT configuration and routing information survive a router reload.

---

# 6. Verify Static NAT Translations

The configured NAT mappings can be viewed with:

```cisco
R1# show ip nat translations
```

Initial output:

```text
Pro  Inside global     Inside local
---  100.0.0.1         172.16.0.1
---  100.0.0.2         172.16.0.2
---  100.0.0.3         172.16.0.3
```

The `---` entries indicate the configured static mappings before traffic creates active protocol translations.

---

# 7. Verify NAT Statistics

NAT statistics were checked using:

```cisco
R1# show ip nat statistics
```

The important output was:

```text
Total translations: 3 (3 static, 0 dynamic, 0 extended)

Outside Interfaces:
GigabitEthernet0/0

Inside Interfaces:
GigabitEthernet0/1

Hits: 0
Misses: 0
```

### Interpretation

The output confirms:

- **3 total translations**
- All **3 translations are static**
- `G0/0` is correctly configured as the outside interface.
- `G0/1` is correctly configured as the inside interface.
- No traffic had yet matched the NAT entries when the statistics were checked.

---

# 8. Test External Connectivity

After configuring NAT, the internal hosts were able to reach external destinations.

A test using Google's DNS server was performed:

```text
C:\>ping 8.8.8.8
```

The first test after configuration produced:

```text
Request timed out.
Reply from 8.8.8.8: bytes=32 time=1ms TTL=126
Reply from 8.8.8.8: bytes=32 time<1ms TTL=126
Reply from 8.8.8.8: bytes=32 time<1ms TTL=126

Packets: Sent = 4, Received = 3, Lost = 1 (25% loss)
```

The initial timeout can occur while ARP/NAT/state information is being established in the simulated network.

---

# 9. Verify DNS Resolution

The next test used a hostname rather than an IP address:

```text
C:\>ping google.com
```

Packet Tracer resolved the hostname to:

```text
172.217.175.238
```

The result was:

```text
Reply from 172.217.175.238: bytes=32 time=10ms TTL=254
Reply from 172.217.175.238: bytes=32 time<1ms TTL=254
Reply from 172.217.175.238: bytes=32 time<1ms TTL=254
Reply from 172.217.175.238: bytes=32 time=1ms TTL=254

Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

This confirms successful connectivity and DNS resolution.

---

# 10. Observe Active NAT Translations

After generating traffic, the NAT table contained protocol-specific translations.

Example:

```text
icmp 100.0.0.1:10
     172.16.0.1:10
     172.217.175.238:10
     172.217.175.238:10
```

Similar entries were generated for:

```text
172.16.0.2 → 100.0.0.2
172.16.0.3 → 100.0.0.3
```

The router also showed DNS traffic:

```text
udp 100.0.0.1:1025
    172.16.0.1:1025
    8.8.8.8:53
    8.8.8.8:53
```

This demonstrates that traffic originating from the internal hosts is being translated through the configured static NAT addresses.

---

# 11. Clear NAT Translation Entries

To remove the active NAT translation entries, the following command was used:

```cisco
R1# clear ip nat translation *
```

After clearing the translations:

```cisco
R1# show ip nat translations
```

The static mappings remained:

```text
Pro  Inside global     Inside local
---  100.0.0.1         172.16.0.1
---  100.0.0.2         172.16.0.2
---  100.0.0.3         172.16.0.3
```

### Important

`clear ip nat translation *` removes active translation entries. It **does not remove the configured static NAT statements**.

---

# 12. Final Connectivity Test

After clearing the active NAT entries, connectivity was tested again:

```text
C:\>ping google.com
```

Result:

```text
Reply from 172.217.175.238: bytes=32 time=11ms TTL=254
Reply from 172.217.175.238: bytes=32 time<1ms TTL=254
Reply from 172.217.175.238: bytes=32 time=49ms TTL=254
Reply from 172.217.175.238: bytes=32 time=10ms TTL=254

Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)

Minimum = 0ms
Maximum = 49ms
Average = 17ms
```

A second test also produced:

```text
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)

Minimum = 0ms
Maximum = 16ms
Average = 6ms
```

The final tests confirm that connectivity remained functional after the NAT table was cleared.

---

# 13. Final R1 Configuration

The relevant final configuration was:

```cisco
interface GigabitEthernet0/0
 ip address 203.0.113.1 255.255.255.252
 ip nat outside
 duplex auto
 speed auto

interface GigabitEthernet0/1
 ip address 172.16.0.254 255.255.255.0
 ip nat inside
 duplex auto
 speed auto

interface GigabitEthernet0/2
 no ip address
 shutdown

ip nat inside source static 172.16.0.1 100.0.0.1
ip nat inside source static 172.16.0.2 100.0.0.2
ip nat inside source static 172.16.0.3 100.0.0.3

ip route 0.0.0.0 0.0.0.0 203.0.113.2
```

---

# 🔍 Verification Commands

The following commands are useful for verifying this lab:

### Check interface configuration

```cisco
show ip interface brief
```

### Check the routing table

```cisco
show ip route
```

### Check NAT translations

```cisco
show ip nat translations
```

### Check NAT statistics

```cisco
show ip nat statistics
```

### Check the complete configuration

```cisco
show running-config
```

### Check connectivity

```text
ping 8.8.8.8
ping google.com
```

---

# 🧠 Key Concepts Learned

## Static NAT

Static NAT creates a permanent one-to-one mapping between an inside local address and an inside global address.

```text
172.16.0.1 ↔ 100.0.0.1
172.16.0.2 ↔ 100.0.0.2
172.16.0.3 ↔ 100.0.0.3
```

## Inside Local

The actual private IP address assigned to an internal host.

Example:

```text
172.16.0.1
```

## Inside Global

The address used to represent the internal host on the external network.

Example:

```text
100.0.0.1
```

## NAT Inside Interface

The interface facing the internal/private network.

```text
G0/1 → ip nat inside
```

## NAT Outside Interface

The interface facing the external network.

```text
G0/0 → ip nat outside
```

## Default Route

The default route provides a path to destinations that are not specifically present in R1's routing table.

```cisco
ip route 0.0.0.0 0.0.0.0 203.0.113.2
```

---

# ✅ Lab Verification Checklist

- [x] Configure R1's inside interface.
- [x] Configure R1's outside interface.
- [x] Configure three static NAT mappings.
- [x] Configure the default route.
- [x] Save the router configuration.
- [x] Verify static NAT translations.
- [x] Verify NAT statistics.
- [x] Test connectivity to `8.8.8.8`.
- [x] Test DNS resolution using `google.com`.
- [x] Observe ICMP NAT translations.
- [x] Observe UDP/DNS NAT translations.
- [x] Clear active NAT translations.
- [x] Retest external connectivity.
- [x] Confirm `0%` packet loss in the final connectivity tests.

---

# 🏁 Conclusion

This lab successfully demonstrated **Static NAT on Cisco IOS**.

R1 was configured to translate three internal addresses from the `172.16.0.0/24` network into corresponding global addresses in the `100.0.0.0/24` range. The router's interfaces were correctly designated as NAT inside and NAT outside, and a default route was configured toward the external gateway.

Verification with `show ip nat translations` demonstrated both the configured static mappings and the protocol-specific translations generated when hosts accessed external destinations.

The final `ping google.com` tests achieved **0% packet loss**, confirming successful end-to-end connectivity and DNS resolution through the NAT configuration.

**Key takeaway:** Static NAT provides a deterministic one-to-one mapping between internal private addresses and externally represented addresses, making it useful when specific internal hosts need consistent external mappings.