# Extended ACLs — Network Traffic Filtering Lab

## 📌 Lab Overview

This lab demonstrates the configuration and verification of **Extended Access Control Lists (ACLs)** on a Cisco router using Cisco Packet Tracer.

The objective is to implement specific network security policies that restrict selected traffic based on:

- Source network
- Destination host
- Protocol
- Destination port/service

Extended ACLs provide more granular traffic control than standard ACLs because they can filter traffic based on IP addresses, protocols, and TCP/UDP ports.

---

## 🎯 Lab Objectives

Configure extended ACLs on **R1** to enforce the following network policies:

1. Hosts in `172.16.2.0/24` must **not communicate with PC1**.
2. Hosts in `172.16.1.0/24` must **not access the DNS service on SRV1**.
3. Hosts in `172.16.2.0/24` must **not access HTTP or HTTPS services on SRV2**.
4. All other traffic should remain permitted unless explicitly denied by an ACL rule.
5. Verify the ACL configuration using connectivity tests and router show commands.

---

## 🗺️ Network Information

| Device / Network | Address |
|---|---|
| PC1 | `172.16.1.1` |
| PC2 | `172.16.1.2` |
| Network 1 | `172.16.1.0/24` |
| R1 G0/0 | `172.16.1.254/24` |
| Network 2 | `172.16.2.0/24` |
| R1 G0/1 | `172.16.2.254/24` |
| SRV1 | `192.168.1.100` |
| SRV2 | `192.168.2.100` |
| R1 S0/0/0 | `203.0.113.1/30` |
| OSPF Neighbor | `203.0.113.2` |

### Services

**SRV1**
- DNS: UDP/53
- DNS: TCP/53

**SRV2**
- HTTP: TCP/80
- HTTPS: TCP/443

---

# 🔐 Security Policies

## Policy 1 — Block Network 172.16.2.0/24 from PC1

Hosts in:

```text
172.16.2.0/24
```

must not communicate with:

```text
PC1 = 172.16.1.1
```

ACL rule:

```cisco
deny ip 172.16.2.0 0.0.0.255 host 172.16.1.1
```

This blocks all IP traffic from the `172.16.2.0/24` network to PC1.

---

## Policy 2 — Block DNS Access to SRV1

Hosts in:

```text
172.16.1.0/24
```

must not access the DNS service on:

```text
SRV1 = 192.168.1.100
```

DNS uses:

- UDP port `53`
- TCP port `53`

Therefore, both protocols must be denied:

```cisco
deny udp 172.16.1.0 0.0.0.255 host 192.168.1.100 eq 53
deny tcp 172.16.1.0 0.0.0.255 host 192.168.1.100 eq 53
```

---

## Policy 3 — Block HTTP/HTTPS Access to SRV2

Hosts in:

```text
172.16.2.0/24
```

must not access the web services running on:

```text
SRV2 = 192.168.2.100
```

HTTP uses TCP port `80`:

```cisco
deny tcp 172.16.2.0 0.0.0.255 host 192.168.2.100 eq 80
```

HTTPS uses TCP port `443`:

```cisco
deny tcp 172.16.2.0 0.0.0.255 host 192.168.2.100 eq 443
```

---

# ⚙️ ACL Configuration

## ACL 100 — Network 172.16.1.0/24

ACL 100 is intended to block DNS access from the `172.16.1.0/24` network to SRV1.

```cisco
R1# configure terminal

R1(config)# ip access-list extended 100

R1(config-ext-nacl)# deny udp 172.16.1.0 0.0.0.255 host 192.168.1.100 eq 53

R1(config-ext-nacl)# deny tcp 172.16.1.0 0.0.0.255 host 192.168.1.100 eq 53

R1(config-ext-nacl)# permit ip any any

R1(config-ext-nacl)# exit
```

Apply the ACL inbound on R1's G0/0 interface:

```cisco
R1(config)# interface GigabitEthernet0/0

R1(config-if)# ip access-group 100 in

R1(config-if)# end

R1# write memory
```

### Traffic flow

```text
172.16.1.0/24
      |
      | DNS UDP/TCP 53
      X
      |
192.168.1.100 (SRV1)
```

Other traffic is permitted by:

```cisco
permit ip any any
```

---

# ACL 101 — Network 172.16.2.0/24

ACL 101 implements the restrictions for hosts in `172.16.2.0/24`.

```cisco
R1# configure terminal

R1(config)# ip access-list extended 101

R1(config-ext-nacl)# deny ip 172.16.2.0 0.0.0.255 host 172.16.1.1

R1(config-ext-nacl)# deny tcp 172.16.2.0 0.0.0.255 host 192.168.2.100 eq 80

R1(config-ext-nacl)# deny tcp 172.16.2.0 0.0.0.255 host 192.168.2.100 eq 443

R1(config-ext-nacl)# permit ip any any

R1(config-ext-nacl)# exit
```

Apply ACL 101 inbound on G0/1:

```cisco
R1(config)# interface GigabitEthernet0/1

R1(config-if)# ip access-group 101 in

R1(config-if)# end

R1# write memory
```

### Traffic filtering

```text
172.16.2.0/24
      |
      +---- X ----> 172.16.1.1 (PC1)
      |
      +---- X ----> 192.168.2.100:80  (HTTP)
      |
      +---- X ----> 192.168.2.100:443 (HTTPS)
      |
      +-----------> Other permitted traffic
```

---

# 🔍 Verification

## Check ACLs

Use:

```cisco
R1# show access-lists
```

Expected output should contain rules similar to:

```text
Extended IP access list 100
    10 deny udp 172.16.1.0 0.0.0.255 host 192.168.1.100 eq domain
    20 deny tcp 172.16.1.0 0.0.0.255 host 192.168.1.100 eq domain
    30 permit ip any any

Extended IP access list 101
    10 deny ip 172.16.2.0 0.0.0.255 host 172.16.1.1
    20 deny tcp 172.16.2.0 0.0.0.255 host 192.168.2.100 eq www
    30 deny tcp 172.16.2.0 0.0.0.255 host 192.168.2.100 eq 443
    40 permit ip any any
```

The values in parentheses, such as:

```text
(16 match(es))
```

represent the number of packets that have matched the corresponding ACL entry.

This is useful for confirming that the ACL rules are actively filtering traffic.

---

# 🧪 Connectivity Testing

## Test 1 — Network 172.16.2.0/24 → PC1

From a host in `172.16.2.0/24`:

```text
C:\> ping 172.16.1.1
```

Expected result:

```text
Request timed out.
```

or:

```text
Destination host unreachable.
```

This confirms that traffic from `172.16.2.0/24` to PC1 is being blocked by ACL 101.

Example observed result:

```text
Pinging 172.16.1.1 with 32 bytes of data:

Reply from 172.16.2.254: Destination host unreachable.
Reply from 172.16.2.254: Destination host unreachable.
Reply from 172.16.2.254: Destination host unreachable.
Reply from 172.16.2.254: Destination host unreachable.

Packets: Sent = 4, Received = 0, Lost = 4 (100% loss)
```

---

## Test 2 — Network 172.16.2.0/24 → SRV2

Test connectivity to SRV2:

```text
C:\> ping 192.168.2.100
```

A successful ping does **not** contradict the HTTP/HTTPS ACL policy.

The ACL only blocks:

```text
TCP/80
TCP/443
```

It does not block ICMP.

Therefore, ICMP ping may still succeed:

```text
Reply from 192.168.2.100
```

This demonstrates an important ACL concept:

> Blocking a particular application service does not necessarily block all communication with the destination host.

---

## Test 3 — Network 172.16.1.0/24 → PC2

From a host in `172.16.1.0/24`:

```text
C:\> ping 172.16.1.2
```

Expected result:

```text
Reply from 172.16.1.2
```

This confirms that the ACL is not unnecessarily blocking normal communication within the network.

---

# ⚠️ Important ACL Ordering Issue

ACL entries are processed **top-to-bottom**.

The first matching ACL statement determines the action.

In the configuration captured from the lab, ACL 100 was displayed as:

```text
Extended IP access list 100
    10 deny udp 172.16.1.0 0.0.0.255 host 192.168.1.100 eq domain
    20 permit ip any any
    30 deny tcp 172.16.1.0 0.0.0.255 host 192.168.1.100 eq domain
```

This means the TCP DNS deny at sequence `30` is effectively unreachable for matching traffic because:

```cisco
permit ip any any
```

at sequence `20` already permits the traffic.

### Correct order

The TCP deny must appear **before** the permit:

```text
10 deny udp ... eq 53
20 deny tcp ... eq 53
30 permit ip any any
```

The corrected ACL should therefore be:

```cisco
ip access-list extended 100
 deny udp 172.16.1.0 0.0.0.255 host 192.168.1.100 eq 53
 deny tcp 172.16.1.0 0.0.0.255 host 192.168.1.100 eq 53
 permit ip any any
```

This is one of the most important lessons from the lab: **ACL sequence matters.**

---

# 🛠️ Complete R1 ACL Configuration

A clean final configuration can be represented as:

```cisco
!
interface GigabitEthernet0/0
 ip address 172.16.1.254 255.255.255.0
 ip access-group 100 in
!
interface GigabitEthernet0/1
 ip address 172.16.2.254 255.255.255.0
 ip access-group 101 in
!
ip access-list extended 100
 deny udp 172.16.1.0 0.0.0.255 host 192.168.1.100 eq 53
 deny tcp 172.16.1.0 0.0.0.255 host 192.168.1.100 eq 53
 permit ip any any
!
ip access-list extended 101
 deny ip 172.16.2.0 0.0.0.255 host 172.16.1.1
 deny tcp 172.16.2.0 0.0.0.255 host 192.168.2.100 eq 80
 deny tcp 172.16.2.0 0.0.0.255 host 192.168.2.100 eq 443
 permit ip any any
!
```

---

# 📊 Policy Verification Summary

| Source | Destination | Service | Expected Result | ACL |
|---|---|---|---|---|
| `172.16.2.0/24` | `172.16.1.1` | Any IP | ❌ Denied | 101 |
| `172.16.1.0/24` | `192.168.1.100` | DNS UDP/53 | ❌ Denied | 100 |
| `172.16.1.0/24` | `192.168.1.100` | DNS TCP/53 | ❌ Denied | 100 |
| `172.16.2.0/24` | `192.168.2.100` | HTTP TCP/80 | ❌ Denied | 101 |
| `172.16.2.0/24` | `192.168.2.100` | HTTPS TCP/443 | ❌ Denied | 101 |
| `172.16.2.0/24` | `192.168.2.100` | ICMP | ✅ Allowed | 101 |
| Other permitted traffic | Any | IP | ✅ Allowed | 100/101 |

---

# 💡 Key Concepts Learned

### 1. Extended ACLs

Extended ACLs can filter traffic based on:

- Source IP
- Destination IP
- Protocol
- TCP/UDP port
- Specific services

### 2. Wildcard Masks

For a `/24` network:

```text
Subnet mask:   255.255.255.0
Wildcard mask: 0.0.0.255
```

Therefore:

```cisco
172.16.2.0 0.0.0.255
```

represents the entire `172.16.2.0/24` network.

### 3. Host Keyword

The following:

```cisco
host 172.16.1.1
```

is equivalent to:

```text
172.16.1.1 0.0.0.0
```

and identifies one specific host.

### 4. Service Ports

| Service | Protocol | Port |
|---|---|---:|
| DNS | UDP | 53 |
| DNS | TCP | 53 |
| HTTP | TCP | 80 |
| HTTPS | TCP | 443 |

### 5. Implicit Deny

Every ACL has an implicit:

```cisco
deny ip any any
```

at the end.

The explicit:

```cisco
permit ip any any
```

was therefore used to ensure that traffic not matching the specified security policies remains permitted.

---

# 📝 Conclusion

This lab demonstrated how to use **Cisco Extended ACLs** to implement granular network security policies.

ACL 100 controls traffic originating from `172.16.1.0/24`, specifically preventing access to the DNS service on SRV1.

ACL 101 controls traffic originating from `172.16.2.0/24`, preventing access to PC1 and blocking HTTP/HTTPS services on SRV2.

The lab also demonstrates the importance of **ACL sequence numbers and processing order**. A `permit ip any any` statement placed before a more specific deny statement can unintentionally bypass the intended security policy.

The final configuration therefore places all specific `deny` statements before the general `permit ip any any` statement.

---

## 🧰 Technologies Used

- Cisco Packet Tracer
- Cisco IOS
- Cisco 2911 Router
- Extended IPv4 ACLs
- OSPF
- TCP/IP
- DNS
- HTTP
- HTTPS
- ICMP

---

## 👨‍💻 Lab Skills Demonstrated

- Creating extended ACLs
- Filtering traffic by source and destination
- Filtering traffic by TCP/UDP ports
- Applying ACLs to router interfaces
- Understanding inbound ACL processing
- Verifying ACL hit counters
- Troubleshooting blocked connectivity
- Understanding implicit deny
- Understanding ACL sequence/order
- Testing network security policies with Packet Tracer