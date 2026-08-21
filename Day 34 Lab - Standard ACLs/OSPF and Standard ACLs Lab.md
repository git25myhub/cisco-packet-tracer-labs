# OSPF and Standard ACLs Lab

## Lab Overview

In this lab, **OSPF** is configured between R1 and R2 to provide dynamic routing and full connectivity between the client networks and server networks.

After establishing OSPF connectivity, **standard IPv4 ACLs** are configured to enforce specific network security policies.

The lab demonstrates:

- OSPF configuration and neighbor formation
- OSPF route learning
- Standard numbered ACLs
- Standard named ACLs
- ACL placement and direction
- Source-based traffic filtering
- ACL verification using match counters
- Connectivity testing with `ping`

---

## Objectives

By completing this lab, you will:

1. Configure OSPF on R1 and R2.
2. Establish an OSPF adjacency between the routers.
3. Verify that OSPF routes are installed.
4. Configure standard numbered ACLs on R1.
5. Configure standard named ACLs on R2.
6. Apply ACLs to the correct router interfaces and direction.
7. Restrict traffic according to the required network policies.
8. Verify permitted and denied traffic.

---

## Network Policies

The following policies must be implemented:

| Policy | Required Behavior |
|---|---|
| Access to `192.168.1.0/24` | **Only PC1 and PC3** may access this network |
| Access to `192.168.2.0/24` | Hosts in `172.16.2.0/24` must be denied |
| `172.16.1.0/24` → `172.16.2.0/24` | Must be denied |
| `172.16.2.0/24` → `172.16.1.0/24` | Must be denied |

---

## Addressing Overview

### R1

| Interface | IP Address | Network |
|---|---|---|
| G0/0 | `172.16.1.254/24` | `172.16.1.0/24` |
| G0/1 | `172.16.2.254/24` | `172.16.2.0/24` |
| S0/0/0 | `203.0.113.1/30` | `203.0.113.0/30` |

### R2

| Interface | IP Address | Network |
|---|---|---|
| G0/0 | `192.168.1.254/24` | `192.168.1.0/24` |
| G0/1 | `192.168.2.254/24` | `192.168.2.0/24` |
| S0/0/0 | `203.0.113.2/30` | `203.0.113.0/30` |

---

# Part 1 — Configure OSPF

OSPF is required so that R1 and R2 can dynamically exchange routes for all four LANs.

## R1 OSPF Configuration

Enter:

```cisco
enable
configure terminal

router ospf 1
 network 172.16.0.0 0.0.255.255 area 0
 network 203.0.113.0 0.0.0.3 area 0

end
```

The first network statement includes:

```text
172.16.1.0/24
172.16.2.0/24
```

The second network statement enables OSPF on the R1–R2 serial link.

---

## R2 OSPF Configuration

Enter:

```cisco
enable
configure terminal

router ospf 1
 network 192.168.0.0 0.0.255.255 area 0
 network 203.0.113.0 0.0.0.3 area 0

end
```

---

## Verify the OSPF Neighbor Relationship

On either router:

```cisco
show ip ospf neighbor
```

Expected result on R2:

```text
Neighbor ID     Pri   State           Dead Time   Address         Interface
203.0.113.1       0   FULL/  -        00:00:30    203.0.113.1     Serial0/0/0
```

The important value is:

```text
FULL
```

This confirms that R1 and R2 have successfully formed an OSPF adjacency.

---

## Verify the Routing Table

On R2:

```cisco
show ip route
```

The routes learned from R1 should appear with an `O` designation:

```text
O  172.16.1.0/24 [110/65] via 203.0.113.1
O  172.16.2.0/24 [110/65] via 203.0.113.1
```

This confirms that R2 has learned the two `172.16.x.x` networks through OSPF.

Likewise, R1 should learn:

```text
192.168.1.0/24
192.168.2.0/24
```

through OSPF.

---

# Part 2 — Configure Standard Numbered ACLs on R1

R1 will use **standard numbered ACLs**.

Remember that standard ACLs filter traffic based primarily on the **source IPv4 address**.

---

## Policy: Block `172.16.2.0/24` from `192.168.2.0/24`

Create ACL 1:

```cisco
configure terminal

access-list 1 deny 172.16.1.0 0.0.0.255
access-list 1 permit any
```

Apply it outbound on G0/1:

```cisco
interface GigabitEthernet0/1
 ip access-group 1 out
end
```

This prevents traffic sourced from:

```text
172.16.1.0/24
```

from leaving R1 toward:

```text
172.16.2.0/24
```

---

## Policy: Block `172.16.2.0/24` from `172.16.1.0/24`

Create ACL 2:

```cisco
configure terminal

access-list 2 deny 172.16.2.0 0.0.0.255
access-list 2 permit any
```

Apply it outbound on G0/0:

```cisco
interface GigabitEthernet0/0
 ip access-group 2 out
end
```

This prevents traffic sourced from:

```text
172.16.2.0/24
```

from entering the `172.16.1.0/24` network through R1.

---

## Verify R1 ACLs

Use:

```cisco
show access-lists
```

Expected:

```text
Standard IP access list 1
    10 deny 172.16.1.0 0.0.0.255
    20 permit any

Standard IP access list 2
    10 deny 172.16.2.0 0.0.0.255
    20 permit any
```

The match counters should increase when denied traffic is generated.

For example:

```text
10 deny 172.16.1.0 0.0.0.255 (9 match(es))
```

indicates that the ACL has matched and denied traffic from that network.

---

# Part 3 — Configure Standard Named ACLs on R2

R2 will use **standard named ACLs**.

Named ACLs are easier to identify and maintain than numbered ACLs because their names can describe their purpose.

---

## ACL for `192.168.1.0/24`

Only PC1 and PC3 should be allowed to access:

```text
192.168.1.0/24
```

According to the lab addressing, the permitted hosts are:

```text
172.16.1.1
172.16.2.1
```

Create the named ACL:

```cisco
configure terminal

ip access-list standard TO_192.168.1.0/24
 permit host 172.16.1.1
 permit host 172.16.2.1
 deny any
exit
```

Apply it outbound toward the `192.168.1.0/24` network:

```cisco
interface GigabitEthernet0/0
 ip access-group TO_192.168.1.0/24 out
exit
```

### Important

The ACL contains:

```cisco
permit host 172.16.1.1
permit host 172.16.2.1
deny any
```

Therefore, only these two source addresses can reach the destination network.

Every other source is denied.

---

# Part 4 — Block `172.16.2.0/24` from `192.168.2.0/24`

Create another named standard ACL:

```cisco
configure terminal

ip access-list standard TO_192.168.2.0/24
 deny 172.16.2.0 0.0.0.255
 permit any
exit
```

Apply it outbound on R2 G0/1:

```cisco
interface GigabitEthernet0/1
 ip access-group TO_192.168.2.0/24 out
exit
```

This prevents hosts in:

```text
172.16.2.0/24
```

from accessing:

```text
192.168.2.0/24
```

while allowing other sources.

---

# Part 5 — Verify R2 ACLs

Run:

```cisco
show access-lists
```

Expected output:

```text
Standard IP access list TO_192.168.1.0/24
    10 permit host 172.16.1.1
    20 permit host 172.16.2.1
    30 deny any

Standard IP access list TO_192.168.2.0/24
    10 deny 172.16.2.0 0.0.0.255
    20 permit any
```

The match counters provide useful evidence that the ACLs are actually processing traffic.

---

# ACL Placement

The placement of standard ACLs is particularly important.

Because standard ACLs match the **source address**, they should generally be placed as close to the **destination network** as possible.

In this lab, the ACLs are applied outbound toward the protected networks.

```text
                 R1                         R2
                 │                          │
       ┌─────────┴─────────┐       ┌────────┴────────┐
       │                   │       │                 │
172.16.1.0/24        172.16.2.0/24  192.168.1.0/24  192.168.2.0/24
       │                   │       │                 │
      PC1                 PC3    Server Network   Server Network
```

The outbound ACLs control traffic immediately before it enters the protected LANs.

---

# Part 6 — Test the ACL Policies

## Test 1 — PC1 to `192.168.1.0/24`

PC1 should be permitted.

From PC1:

```text
ping 192.168.1.100
```

Expected:

```text
Reply from 192.168.1.100
```

A successful test confirms that PC1 is permitted by:

```text
TO_192.168.1.0/24
```

---

## Test 2 — PC3 to `192.168.1.0/24`

PC3 should also be permitted.

```text
ping 192.168.1.100
```

Expected:

```text
Reply from 192.168.1.100
```

---

## Test 3 — Unauthorized Host to `192.168.1.0/24`

A host other than PC1 or PC3 should be denied.

```text
ping 192.168.1.100
```

Expected:

```text
Request timed out.
```

or an appropriate unreachable response.

This demonstrates the effect of:

```cisco
deny any
```

in the named ACL.

---

## Test 4 — `172.16.2.0/24` to `192.168.2.0/24`

From a host in `172.16.2.0/24`:

```text
ping 192.168.2.100
```

Expected:

```text
Request timed out.
```

The traffic is denied by:

```text
TO_192.168.2.0/24
```

---

## Test 5 — `172.16.1.0/24` to `172.16.2.0/24`

From a host in `172.16.1.0/24`:

```text
ping 172.16.2.1
```

Expected:

```text
Destination host unreachable.
```

The traffic is blocked by ACL 1 on R1.

---

## Test 6 — `172.16.2.0/24` to `172.16.1.0/24`

From a host in `172.16.2.0/24`:

```text
ping 172.16.1.1
```

Expected:

```text
Destination host unreachable.
```

The traffic is blocked by ACL 2 on R1.

---

# Verify ACL Match Counters

After performing the tests, run:

```cisco
show access-lists
```

Example:

```text
Standard IP access list 1
    10 deny 172.16.1.0 0.0.0.255 (9 match(es))
    20 permit any

Standard IP access list 2
    10 deny 172.16.2.0 0.0.0.255 (6 match(es))
    20 permit any
```

The increasing match counters confirm that traffic is reaching and being processed by the ACL.

---

# Important ACL Concepts

## Implicit Deny

Every ACL has an implicit:

```text
deny any
```

at the end.

For example:

```cisco
access-list 1 deny 172.16.1.0 0.0.0.255
```

would deny the specified network **and also deny all other traffic** because of the implicit deny.

Therefore, when the intention is to block only one source and allow everything else, use:

```cisco
access-list 1 deny 172.16.1.0 0.0.0.255
access-list 1 permit any
```

---

## Standard ACL Wildcard Masks

The wildcard mask:

```text
0.0.0.255
```

matches an entire `/24` network.

Therefore:

```cisco
deny 172.16.1.0 0.0.0.255
```

matches:

```text
172.16.1.0 – 172.16.1.255
```

A host-specific entry uses:

```cisco
permit host 172.16.1.1
```

which is equivalent to:

```cisco
permit 172.16.1.1 0.0.0.0
```

---

# Useful Verification Commands

### Verify OSPF neighbors

```cisco
show ip ospf neighbor
```

### Verify OSPF routes

```cisco
show ip route ospf
```

### View the complete routing table

```cisco
show ip route
```

### View ACLs

```cisco
show access-lists
```

### View interface ACL assignments

```cisco
show ip interface GigabitEthernet0/0
show ip interface GigabitEthernet0/1
```

### View the running configuration

```cisco
show running-config
```

### Save the configuration

```cisco
write memory
```

or:

```cisco
copy running-config startup-config
```

---

# Expected Final Configuration

## R1 ACLs

```cisco
access-list 1 deny 172.16.1.0 0.0.0.255
access-list 1 permit any

access-list 2 deny 172.16.2.0 0.0.0.255
access-list 2 permit any
```

Applied as:

```cisco
interface GigabitEthernet0/1
 ip access-group 1 out

interface GigabitEthernet0/0
 ip access-group 2 out
```

## R2 ACLs

```cisco
ip access-list standard TO_192.168.1.0/24
 permit host 172.16.1.1
 permit host 172.16.2.1
 deny any

ip access-list standard TO_192.168.2.0/24
 deny 172.16.2.0 0.0.0.255
 permit any
```

Applied as:

```cisco
interface GigabitEthernet0/0
 ip access-group TO_192.168.1.0/24 out

interface GigabitEthernet0/1
 ip access-group TO_192.168.2.0/24 out
```

---

# Verification Checklist

- [ ] OSPF process 1 configured on R1.
- [ ] OSPF process 1 configured on R2.
- [ ] R1 and R2 form a `FULL` OSPF adjacency.
- [ ] R1 learns the `192.168.1.0/24` network through OSPF.
- [ ] R1 learns the `192.168.2.0/24` network through OSPF.
- [ ] R2 learns the `172.16.1.0/24` network through OSPF.
- [ ] R2 learns the `172.16.2.0/24` network through OSPF.
- [ ] Standard numbered ACLs configured on R1.
- [ ] Standard named ACLs configured on R2.
- [ ] ACLs applied in the correct direction.
- [ ] Only PC1 and PC3 can access `192.168.1.0/24`.
- [ ] `172.16.2.0/24` cannot access `192.168.2.0/24`.
- [ ] `172.16.1.0/24` cannot access `172.16.2.0/24`.
- [ ] `172.16.2.0/24` cannot access `172.16.1.0/24`.
- [ ] Allowed traffic successfully passes.
- [ ] Denied traffic fails.
- [ ] ACL match counters confirm the policies are being enforced.
- [ ] Configurations are saved.

---

## Lab Summary

This lab combines **OSPF dynamic routing** with **standard IPv4 ACLs** to provide both network reachability and traffic control.

OSPF provides the routing information required for communication between the four networks, while the ACLs enforce the security policies.

The key design principle demonstrated is:

```text
OSPF = Determines where traffic can go
ACL  = Determines which traffic is allowed
```

By verifying both the routing table and ACL match counters, you can distinguish between a **routing problem** and an **ACL policy decision** when troubleshooting connectivity.