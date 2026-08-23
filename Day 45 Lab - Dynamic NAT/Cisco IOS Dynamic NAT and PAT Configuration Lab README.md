# Cisco IOS Dynamic NAT & PAT Configuration Lab

## 📌 Lab Overview

This lab demonstrates the configuration and behavior of **Dynamic NAT** and **Port Address Translation (PAT)** on a Cisco router.

The lab begins by configuring Dynamic NAT using a limited pool of two public IP addresses. Three internal PCs then attempt to access an external network. Because only two public addresses are available, the third PC cannot obtain a NAT address while the first two translations are active.

The Dynamic NAT configuration is then replaced with **PAT**, allowing all internal hosts to share R1's single public IP address.

---

## 🎯 Lab Objectives

By completing this lab, you will learn how to:

- Configure NAT inside and outside interfaces.
- Create an access list identifying internal addresses that should be translated.
- Configure a Dynamic NAT pool.
- Translate traffic from the `172.16.0.0/24` network.
- Understand NAT pool address exhaustion.
- Observe what happens when more hosts require translation than available public addresses.
- Clear active NAT translations.
- Replace Dynamic NAT with PAT.
- Configure PAT using R1's public interface address.
- Verify NAT translations.
- Test connectivity from multiple internal hosts.
- Understand how PAT allows many private hosts to share one public IP address.

---

# 🗺️ Network Addressing

| Device | Interface | IP Address | NAT Role |
|---|---|---|---|
| R1 | G0/0 | `203.0.113.1/30` | NAT Outside |
| R1 | G0/1 | `172.16.0.254/24` | NAT Inside |
| PC1 | — | `172.16.0.1` | Inside Local |
| PC2 | — | `172.16.0.2` | Inside Local |
| PC3 | — | `172.16.0.3` | Inside Local |
| External Gateway | — | `203.0.113.2` | Default Gateway |

### Dynamic NAT Pool

```text
Pool Name:       POOL1
Start Address:   100.0.0.1
End Address:     100.0.0.2
Subnet:          100.0.0.0/24
Available IPs:   2
```

### PAT Address

PAT ultimately uses R1's public IP:

```text
203.0.113.1
```

---

# 1. Configure NAT Inside and Outside Interfaces

The first step is to identify which interfaces face the internal and external networks.

## Configure G0/0 as NAT Outside

```cisco
R1> enable
R1# configure terminal
R1(config)# interface g0/0
R1(config-if)# ip nat outside
```

## Configure G0/1 as NAT Inside

```cisco
R1(config-if)# interface g0/1
R1(config-if)# ip nat inside
R1(config-if)# exit
```

The final interface roles are:

```text
G0/0 → NAT OUTSIDE
G0/1 → NAT INSIDE
```

---

# 2. Create the NAT Access List

The internal network that should be translated is:

```text
172.16.0.0/24
```

An extended wildcard mask is used in the access list:

```cisco
R1(config)# access-list 1 permit 172.16.0.0 0.0.0.255
```

This permits all addresses from:

```text
172.16.0.0
through
172.16.0.255
```

Therefore, PC1, PC2, and PC3 are all eligible for NAT.

---

# 3. Create the Dynamic NAT Pool

A Dynamic NAT pool containing two public addresses was configured:

```cisco
R1(config)# ip nat pool POOL1 100.0.0.1 100.0.0.2 netmask 255.255.255.0
```

The pool contains only:

```text
100.0.0.1
100.0.0.2
```

Therefore, there are only **two available public addresses** for translation.

---

# 4. Associate the ACL with the NAT Pool

The access list and NAT pool were connected using:

```cisco
R1(config)# ip nat inside source list 1 pool POOL1
```

The complete Dynamic NAT configuration is therefore:

```cisco
access-list 1 permit 172.16.0.0 0.0.0.255

ip nat pool POOL1 100.0.0.1 100.0.0.2 netmask 255.255.255.0

ip nat inside source list 1 pool POOL1
```

---

# 5. Save the Configuration

The configuration was saved using:

```cisco
R1(config)# do write
```

Expected result:

```text
Building configuration...

[OK]
```

---

# 6. Verify Dynamic NAT Translations

The NAT table was examined using:

```cisco
R1(config)# do show ip nat translations
```

The router showed translations similar to:

```text
Pro  Inside global     Inside local
icmp 100.0.0.1:5       172.16.0.1:5
icmp 100.0.0.2:1       172.16.0.2:1
udp  100.0.0.1:1025     172.16.0.1:1025
udp  100.0.0.2:1025     172.16.0.2:1025
```

This demonstrates that:

```text
PC1 172.16.0.1 → 100.0.0.1
PC2 172.16.0.2 → 100.0.0.2
```

The two available public addresses have been consumed.

---

# 7. Test Dynamic NAT with PC1 and PC2

PC1 and PC2 were instructed to ping:

```text
google.com
```

The traffic generated NAT translations for both hosts.

The NAT table showed:

```text
172.16.0.1 → 100.0.0.1
172.16.0.2 → 100.0.0.2
```

Both hosts were therefore able to communicate with the external network.

---

# 8. Test PC3

PC3 has the internal address:

```text
172.16.0.3
```

PC3 was then instructed to ping:

```text
google.com
```

At this point, both addresses in the NAT pool were already being used:

```text
100.0.0.1 → PC1
100.0.0.2 → PC2
```

There was no third address available for:

```text
PC3 → 172.16.0.3
```

### What happens?

PC3's traffic cannot obtain a Dynamic NAT address from `POOL1`.

Therefore, PC3's Internet connectivity fails while both NAT pool addresses are occupied.

---

# 🧠 Why Does PC3 Fail?

This is one of the most important concepts demonstrated by the lab.

Dynamic NAT uses addresses from a configured pool.

The pool contains only:

```text
100.0.0.1
100.0.0.2
```

But there are three internal hosts:

```text
172.16.0.1
172.16.0.2
172.16.0.3
```

The first two hosts consume the available translations:

```text
PC1 → 100.0.0.1
PC2 → 100.0.0.2
```

When PC3 attempts to communicate externally, R1 has no free public address to assign.

This results in **NAT pool exhaustion**.

### Key lesson

> Dynamic NAT requires enough public addresses to accommodate the number of simultaneous translations that need to be established.

---

# 9. Clear NAT Translation Entries

Before switching to PAT, the active NAT translations were cleared.

The command used was:

```cisco
R1(config)# do clear ip nat translation *
```

The attempted command:

```cisco
do clear ip nat translations *
```

was rejected because the Packet Tracer IOS syntax expects:

```cisco
clear ip nat translation *
```

The correct command successfully cleared the active NAT entries.

---

# 10. Replace Dynamic NAT with PAT

The Dynamic NAT statement was replaced with PAT.

The new configuration uses R1's public interface address:

```text
203.0.113.1
```

The command configured was:

```cisco
R1(config)# ip nat inside source list 1 interface g0/0 overload
```

This enables PAT.

### PAT Configuration

```cisco
ip nat inside source list 1 interface GigabitEthernet0/0 overload
```

The keyword:

```text
overload
```

is what enables Port Address Translation.

---

# 11. Dynamic NAT vs PAT

### Dynamic NAT

```text
172.16.0.1 → 100.0.0.1
172.16.0.2 → 100.0.0.2
```

Each internal host requires a separate public IP address.

### PAT

```text
172.16.0.1 ─┐
172.16.0.2 ─┼──→ 203.0.113.1
172.16.0.3 ─┘
```

Multiple internal hosts can share the same public IP by using different port numbers.

---

# 12. Verify the Final NAT Configuration

The final NAT-related configuration was checked with:

```cisco
R1(config)# do show running-config | include nat
```

The output was:

```text
ip nat outside
ip nat inside
ip nat pool POOL1 100.0.0.1 100.0.0.2 netmask 255.255.255.0
ip nat inside source list 1 interface GigabitEthernet0/0 overload
```

### Important Observation

Although the old NAT pool definition remained in the running configuration:

```text
ip nat pool POOL1 100.0.0.1 100.0.0.2 netmask 255.255.255.0
```

the active NAT rule now uses:

```text
interface GigabitEthernet0/0 overload
```

Therefore, traffic is translated using R1's public address rather than the old pool.

For a clean configuration, the unused pool can also be removed:

```cisco
no ip nat pool POOL1
```

---

# 13. Test PAT from All PCs

After PAT was configured, the PCs were able to ping external destinations.

Example:

```text
C:\>ping google.com
```

PC1:

```text
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)

Minimum = 0ms
Maximum = 43ms
Average = 10ms
```

Another PC:

```text
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)

Minimum = 0ms
Maximum = 10ms
Average = 2ms
```

PC3 also successfully accessed Google after PAT was enabled.

This confirms that PAT solved the public-address limitation encountered with Dynamic NAT.

---

# 14. Verify PAT Translations

The NAT translations were examined using:

```cisco
R1(config)# do show ip nat translations
```

The router showed entries such as:

```text
icmp 203.0.113.1:10
     172.16.0.1:10
     172.217.175.238:10
     172.217.175.238:10

icmp 203.0.113.1:1
     172.16.0.3:1
     172.217.175.238:1
     172.217.175.238:1

icmp 203.0.113.1:5
     172.16.0.2:5
     172.217.175.238:5
     172.217.175.238:5
```

The important observation is that **all three internal hosts are using the same public IP address**:

```text
203.0.113.1
```

The router distinguishes their sessions using different identifiers/port information.

---

# 15. DNS Translations with PAT

The NAT table also showed UDP DNS translations:

```text
udp 203.0.113.1:1024
    172.16.0.2:1026
    8.8.8.8:53
    8.8.8.8:53

udp 203.0.113.1:1025
    172.16.0.3:1026
    8.8.8.8:53
    8.8.8.8:53

udp 203.0.113.1:1026
    172.16.0.1:1026
    8.8.8.8:53
    8.8.8.8:53
```

This demonstrates how PAT allows multiple internal hosts to simultaneously communicate with the DNS server using the same public IP.

---

# 16. Final R1 Configuration

The final relevant configuration was:

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

ip nat inside source list 1 interface GigabitEthernet0/0 overload

ip route 0.0.0.0 0.0.0.0 203.0.113.2

access-list 1 permit 172.16.0.0 0.0.0.255
```

The old Dynamic NAT pool definition was still visible in the provided `show running-config` output:

```cisco
ip nat pool POOL1 100.0.0.1 100.0.0.2 netmask 255.255.255.0
```

However, it was no longer referenced by the active NAT rule.

---

# 🔍 Useful Verification Commands

## Check interfaces

```cisco
show ip interface brief
```

## Check routing

```cisco
show ip route
```

## Check NAT translations

```cisco
show ip nat translations
```

## Check NAT statistics

```cisco
show ip nat statistics
```

## Check NAT configuration

```cisco
show running-config | include nat
```

## Check the access list

```cisco
show access-lists
```

## Clear active NAT translations

```cisco
clear ip nat translation *
```

## Save configuration

```cisco
write
```

or:

```cisco
copy running-config startup-config
```

---

# 📊 Dynamic NAT vs PAT

| Feature | Dynamic NAT | PAT |
|---|---|---|
| Public IP requirement | One per active host/translation | One public IP can serve many hosts |
| Configuration | NAT pool | Interface + `overload` |
| Pool used | Yes | No |
| Port translation | Not normally required | Yes |
| Address conservation | Limited | Excellent |
| Example | `172.16.0.1 → 100.0.0.1` | `172.16.0.1 → 203.0.113.1:port` |
| Three PCs with two public IPs | Third host fails when pool is exhausted | All three can communicate |

---

# 🧠 Key Concepts Learned

## Dynamic NAT

Dynamic NAT dynamically assigns an address from a configured pool to an internal host.

In this lab:

```text
100.0.0.1
100.0.0.2
```

were available.

Because only two addresses existed in the pool, only two hosts could simultaneously obtain translations.

---

## NAT Pool Exhaustion

When all addresses in a Dynamic NAT pool are in use, additional hosts cannot create new translations.

In this lab:

```text
PC1 → 100.0.0.1
PC2 → 100.0.0.2
PC3 → No available address
```

This explains why PC3 could not successfully reach the external network using Dynamic NAT.

---

## PAT

PAT allows multiple internal hosts to share a single public IP address.

In this lab:

```text
PC1 ─┐
PC2 ─┼──→ 203.0.113.1
PC3 ─┘
```

R1 uses different port/translation identifiers to keep the sessions separate.

---

## Access List

The ACL identifies which internal addresses are eligible for NAT:

```cisco
access-list 1 permit 172.16.0.0 0.0.0.255
```

This includes the entire `172.16.0.0/24` network.

---

# 🧪 Lab Results

| Test | Result |
|---|---|
| Configure NAT inside/outside | ✅ Successful |
| Configure `172.16.0.0/24` ACL | ✅ Successful |
| Create Dynamic NAT pool | ✅ Successful |
| PC1 through Dynamic NAT | ✅ Successful |
| PC2 through Dynamic NAT | ✅ Successful |
| PC3 with pool exhausted | ❌ Translation unavailable |
| Clear NAT translations | ✅ Successful |
| Configure PAT | ✅ Successful |
| PC1 through PAT | ✅ Successful |
| PC2 through PAT | ✅ Successful |
| PC3 through PAT | ✅ Successful |
| Verify PAT translations | ✅ Successful |
| All PCs share `203.0.113.1` | ✅ Confirmed |

---

# ❓ Answers to Lab Questions

### 1. What happens to PC3's ping with Dynamic NAT?

PC3 cannot successfully create a NAT translation when PC1 and PC2 have already consumed the two available addresses in the NAT pool.

The pool contains only:

```text
100.0.0.1
100.0.0.2
```

Therefore, there is no available public address for PC3.

---

### 2. Do the pings work after switching to PAT?

**Yes.**

After configuring:

```cisco
ip nat inside source list 1 interface g0/0 overload
```

all three PCs can access the external network.

PAT allows PC1, PC2, and PC3 to share R1's public address:

```text
203.0.113.1
```

---

### 3. What does the NAT table show?

The NAT table shows that all three internal addresses are translated to the same public IP:

```text
172.16.0.1 → 203.0.113.1
172.16.0.2 → 203.0.113.1
172.16.0.3 → 203.0.113.1
```

Different port/translation identifiers allow R1 to distinguish the individual connections.

---

# ✅ Lab Verification Checklist

- [x] Configure G0/0 as `ip nat outside`.
- [x] Configure G0/1 as `ip nat inside`.
- [x] Create ACL permitting `172.16.0.0/24`.
- [x] Create Dynamic NAT pool `POOL1`.
- [x] Configure Dynamic NAT using the pool.
- [x] Test connectivity from PC1.
- [x] Test connectivity from PC2.
- [x] Demonstrate NAT pool exhaustion with PC3.
- [x] Clear NAT translations.
- [x] Replace Dynamic NAT with PAT.
- [x] Use R1's G0/0 address for PAT.
- [x] Enable `overload`.
- [x] Test connectivity from all three PCs.
- [x] Examine PAT translations.
- [x] Verify all PCs can share `203.0.113.1`.
- [x] Save the final configuration.

---

# 🏁 Conclusion

This lab demonstrated the practical difference between **Dynamic NAT and PAT**.

Dynamic NAT successfully translated internal hosts while public addresses were available. However, because the configured pool contained only two addresses, the third PC could not obtain a translation once PC1 and PC2 were using the available addresses.

The configuration was then changed to PAT:

```cisco
ip nat inside source list 1 interface GigabitEthernet0/0 overload
```

With PAT enabled, all three internal PCs successfully accessed the external network while sharing R1's single public address, `203.0.113.1`.

**Key takeaway:** Dynamic NAT requires a pool of public addresses, while PAT conserves public IPv4 addresses by allowing many private hosts to share one public IP through port-based translations.