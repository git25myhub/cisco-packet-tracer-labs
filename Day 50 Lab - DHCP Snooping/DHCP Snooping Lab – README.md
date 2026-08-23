# DHCP Snooping Lab

## 📌 Lab Overview

This lab demonstrates how to configure a Cisco router as a **DHCP server** and protect the network against unauthorized DHCP servers using **DHCP Snooping** on Cisco switches.

The lab covers:

- Configuring R1 as a DHCP server.
- Excluding reserved IP addresses from the DHCP pool.
- Configuring DHCP Snooping for VLAN 1.
- Identifying DHCP trusted interfaces.
- Understanding why DHCP clients may initially fail to obtain an address.
- Testing DHCP address assignment with `ipconfig /renew`.
- Troubleshooting and correcting DHCP Snooping configuration.
- Verifying the DHCP Snooping configuration and DHCP bindings.

---

# 🎯 Lab Objectives

By completing this lab, you should be able to:

1. Configure a Cisco router as a DHCP server.
2. Exclude specific addresses from a DHCP pool.
3. Create and configure a DHCP pool.
4. Enable DHCP Snooping on a switch.
5. Configure the appropriate uplink interfaces as trusted.
6. Understand the difference between trusted and untrusted DHCP Snooping interfaces.
7. Verify DHCP address assignment from a client.
8. Troubleshoot DHCP Snooping when DHCP requests fail.

---

# 🗺️ Lab Topology

The lab consists of:

- **R1** — DHCP Server
- **SW1** — DHCP Snooping switch
- **SW2** — DHCP Snooping switch
- **PC1** — DHCP Client

The network uses:

```text
Network:       192.168.1.0/24
R1:            192.168.1.1
DHCP clients:  192.168.1.10 and above
VLAN:          VLAN 1
```

The router interface connected to the LAN is:

```text
R1 GigabitEthernet0/0
```

with:

```text
192.168.1.1/24
```

---

# 🧩 Part 1 — Configure R1 as a DHCP Server

## DHCP Requirements

The DHCP server must:

- Use network `192.168.1.0/24`.
- Exclude `192.168.1.1` through `192.168.1.9`.
- Use R1 as the default gateway.
- Assign addresses beginning with `192.168.1.10`.

---

## Configure R1

Enter privileged EXEC mode:

```cisco
R1> enable
R1# configure terminal
```

Exclude the reserved addresses:

```cisco
R1(config)# ip dhcp excluded-address 192.168.1.1 192.168.1.9
```

Create the DHCP pool:

```cisco
R1(config)# ip dhcp pool POOL1
```

Configure the network:

```cisco
R1(dhcp-config)# network 192.168.1.0 255.255.255.0
```

Configure the default gateway:

```cisco
R1(dhcp-config)# default-router 192.168.1.1
```

Exit configuration mode:

```cisco
R1(dhcp-config)# exit
R1(config)# end
```

Save the configuration:

```cisco
R1# write memory
```

---

# 🔎 Verify the DHCP Configuration

Display the running configuration:

```cisco
R1# show running-config
```

The relevant configuration should look similar to:

```cisco
ip dhcp excluded-address 192.168.1.1 192.168.1.9

ip dhcp pool POOL1
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.1
```

Verify DHCP pool information:

```cisco
R1# show ip dhcp pool
```

Verify DHCP bindings:

```cisco
R1# show ip dhcp binding
```

Verify DHCP statistics:

```cisco
R1# show ip dhcp server statistics
```

---

# 🔐 Part 2 — Configure DHCP Snooping on SW1

DHCP Snooping helps protect the network against **rogue DHCP servers**.

The switch classifies ports as either:

- **Trusted** — DHCP server responses are allowed.
- **Untrusted** — DHCP server responses are blocked.

In this topology, the interface connected toward the legitimate DHCP server should be trusted.

For SW1, the uplink toward R1 is:

```text
GigabitEthernet0/2
```

---

## Enable DHCP Snooping

Enter configuration mode:

```cisco
SW1> enable
SW1# configure terminal
```

Enable DHCP Snooping for VLAN 1:

```cisco
SW1(config)# ip dhcp snooping
SW1(config)# ip dhcp snooping vlan 1
```

Configure the uplink as trusted:

```cisco
SW1(config)# interface gigabitethernet 0/2
SW1(config-if)# ip dhcp snooping trust
SW1(config-if)# exit
```

Save:

```cisco
SW1(config)# end
SW1# write memory
```

---

# 🔎 Verify SW1 DHCP Snooping

Use:

```cisco
SW1# show running-config
```

The configuration should contain:

```cisco
ip dhcp snooping vlan 1
no ip dhcp snooping information option
```

and:

```cisco
interface GigabitEthernet0/2
 ip dhcp snooping trust
```

You can also use:

```cisco
SW1# show ip dhcp snooping
```

The output should indicate that DHCP Snooping is enabled for VLAN 1 and that G0/2 is trusted.

---

# 🔐 Part 3 — Configure DHCP Snooping on SW2

SW2 also needs DHCP Snooping enabled for VLAN 1.

The uplink toward the DHCP server/network is:

```text
GigabitEthernet0/1
```

Enter configuration mode:

```cisco
SW2> enable
SW2# configure terminal
```

Enable DHCP Snooping:

```cisco
SW2(config)# ip dhcp snooping
SW2(config)# ip dhcp snooping vlan 1
```

Configure the uplink as trusted:

```cisco
SW2(config)# interface gigabitethernet 0/1
SW2(config-if)# ip dhcp snooping trust
SW2(config-if)# exit
```

Save the configuration:

```cisco
SW2(config)# end
SW2# write memory
```

---

# 🔎 Verify SW2 DHCP Snooping

Use:

```cisco
SW2# show running-config
```

The relevant configuration should contain:

```cisco
ip dhcp snooping vlan 1
no ip dhcp snooping information option
```

and:

```cisco
interface GigabitEthernet0/1
 ip dhcp snooping trust
```

You can also verify the operational status:

```cisco
SW2# show ip dhcp snooping
```

---

# 🧪 Part 4 — Test DHCP on PC1

On PC1, open the command prompt:

```text
C:\> ipconfig /renew
```

The client should send a DHCP request and obtain an address from R1.

The lab produced:

```text
IP Address......................: 192.168.1.10
Subnet Mask.....................: 255.255.255.0
Default Gateway.................: 192.168.1.1
DNS Server......................: 0.0.0.0
```

This confirms that PC1 successfully received an address from the DHCP pool.

---

# ❓ Does DHCP Work?

### Yes — after the correct DHCP Snooping configuration is applied.

The successful result was:

```text
IP Address:      192.168.1.10
Subnet Mask:     255.255.255.0
Default Gateway: 192.168.1.1
```

The address `192.168.1.10` is significant because:

```text
192.168.1.1 - 192.168.1.9
```

were excluded from the DHCP pool.

Therefore, the first available client address was:

```text
192.168.1.10
```

---

# ⚠️ Why DHCP Snooping Can Break DHCP

DHCP Snooping treats switch ports as **untrusted by default**.

This is important because DHCP communication involves messages traveling in both directions.

A simplified DHCP process is:

```text
PC1                         R1
 |                           |
 |---- DHCP Discover ------->|
 |                           |
 |<----- DHCP Offer ---------|
 |                           |
 |---- DHCP Request -------->|
 |                           |
 |<---- DHCP ACK ------------|
```

The DHCP client sends requests from an untrusted access port.

However, the DHCP server's responses must be permitted through the trusted uplink.

If the interface toward the legitimate DHCP server is **not trusted**, the switch can discard DHCP server responses.

The client may therefore fail to obtain an IP address.

---

# 🛠️ Part 5 — Troubleshooting DHCP Snooping

If PC1 does not receive an address, first check DHCP Snooping:

```cisco
SW1# show ip dhcp snooping
```

Check the trusted interface:

```cisco
SW1# show running-config interface gigabitethernet 0/2
```

You should see:

```cisco
interface GigabitEthernet0/2
 ip dhcp snooping trust
```

On SW2:

```cisco
SW2# show running-config interface gigabitethernet 0/1
```

Expected:

```cisco
interface GigabitEthernet0/1
 ip dhcp snooping trust
```

---

# 🔧 Necessary Configuration Change

If DHCP fails because the uplink is untrusted, configure the interface connected toward the legitimate DHCP server as trusted.

### SW1

```cisco
SW1(config)# interface gigabitethernet 0/2
SW1(config-if)# ip dhcp snooping trust
```

### SW2

```cisco
SW2(config)# interface gigabitethernet 0/1
SW2(config-if)# ip dhcp snooping trust
```

Save the configuration:

```cisco
SW1# write memory
SW2# write memory
```

Then renew the address on PC1:

```text
C:\> ipconfig /renew
```

---

# 🧠 Important Security Principle

Only interfaces that lead toward a **legitimate DHCP server** should normally be trusted.

Client-facing interfaces should remain untrusted.

For example:

```text
                 Trusted
                    |
                    v
              +-----------+
              |    R1     |
              | DHCP Svr  |
              +-----+-----+
                    |
                    |
              G0/2 | Trusted
                    |
              +-----+-----+
              |   SW1     |
              +-----+-----+
                    |
              F0/x | Untrusted
                    |
                  PC1
```

A rogue DHCP server connected to an untrusted access port cannot freely send DHCP server responses through the switch.

---

# 📋 DHCP Snooping Verification Commands

## Check DHCP Snooping status

```cisco
show ip dhcp snooping
```

## Check DHCP Snooping bindings

```cisco
show ip dhcp snooping binding
```

## Check the running configuration

```cisco
show running-config
```

## Check a specific interface

```cisco
show running-config interface gigabitethernet 0/2
```

or:

```cisco
show running-config interface gigabitethernet 0/1
```

## Check interface status

```cisco
show interfaces status
```

---

# 📡 DHCP Server Verification

On R1, verify whether the client received a DHCP lease:

```cisco
R1# show ip dhcp binding
```

You should see a binding similar to:

```text
IP address       Client-ID/              Lease expiration        Type
                 Hardware address
192.168.1.10     ...                     ...                    Automatic
```

You can also check:

```cisco
R1# show ip dhcp pool
```

This allows you to see the pool utilization and available addresses.

---

# 🧪 Troubleshooting Workflow

When a DHCP client cannot obtain an address, follow this sequence:

### 1. Check the router interface

```cisco
R1# show ip interface brief
```

Confirm that G0/0 is:

```text
up/up
```

and has:

```text
192.168.1.1/24
```

### 2. Check the DHCP configuration

```cisco
R1# show running-config
```

Confirm:

```cisco
ip dhcp excluded-address 192.168.1.1 192.168.1.9

ip dhcp pool POOL1
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.1
```

### 3. Check DHCP Snooping

```cisco
SW1# show ip dhcp snooping
SW2# show ip dhcp snooping
```

### 4. Verify trusted uplinks

SW1:

```cisco
SW1# show running-config interface g0/2
```

SW2:

```cisco
SW2# show running-config interface g0/1
```

### 5. Renew the client address

```text
C:\> ipconfig /renew
```

### 6. Verify the result

```text
C:\> ipconfig
```

Expected:

```text
IP Address:      192.168.1.10
Subnet Mask:     255.255.255.0
Default Gateway: 192.168.1.1
```

---

# 🔍 Key Configuration Explained

## DHCP Excluded Addresses

```cisco
ip dhcp excluded-address 192.168.1.1 192.168.1.9
```

Prevents R1 from assigning these addresses to DHCP clients.

This reserves the first nine addresses for infrastructure or statically configured devices.

---

## DHCP Pool

```cisco
ip dhcp pool POOL1
```

Creates a DHCP pool named `POOL1`.

---

## Network

```cisco
network 192.168.1.0 255.255.255.0
```

Defines the network from which DHCP addresses are allocated.

---

## Default Router

```cisco
default-router 192.168.1.1
```

Tells DHCP clients to use R1 as their default gateway.

---

## DHCP Snooping

```cisco
ip dhcp snooping
```

Globally enables DHCP Snooping.

---

## DHCP Snooping VLAN

```cisco
ip dhcp snooping vlan 1
```

Enables DHCP Snooping for VLAN 1.

---

## Trusted Interface

```cisco
ip dhcp snooping trust
```

Marks an interface as trusted for DHCP server traffic.

---

# 📊 Final Configuration Summary

| Device | Interface | Configuration | Purpose |
|---|---|---|---|
| R1 | G0/0 | `192.168.1.1/24` | DHCP Server / Gateway |
| SW1 | G0/2 | DHCP Snooping Trusted | Uplink |
| SW2 | G0/1 | DHCP Snooping Trusted | Uplink |
| PC1 | NIC | DHCP | Client |

### DHCP Pool

| Parameter | Value |
|---|---|
| Pool Name | POOL1 |
| Network | 192.168.1.0/24 |
| Excluded | 192.168.1.1–192.168.1.9 |
| First Available Address | 192.168.1.10 |
| Default Gateway | 192.168.1.1 |
| VLAN | 1 |

---

# 📝 Lab Findings

### R1

R1 was successfully configured as the DHCP server using:

```text
192.168.1.0/24
```

The addresses `192.168.1.1` through `192.168.1.9` were excluded.

R1's G0/0 interface was configured as:

```text
192.168.1.1/24
```

and was used as the default gateway.

### SW1

DHCP Snooping was enabled for VLAN 1.

The uplink `G0/2` was configured as trusted:

```cisco
ip dhcp snooping trust
```

### SW2

DHCP Snooping was enabled for VLAN 1.

The uplink `G0/1` was configured as trusted.

### PC1

PC1 successfully obtained:

```text
IP Address:      192.168.1.10
Subnet Mask:     255.255.255.0
Default Gateway: 192.168.1.1
```

The successful assignment confirms that DHCP traffic was able to pass through the DHCP Snooping-enabled switches correctly.

---

# ⚠️ Important Observation

The lab configuration also included:

```cisco
no ip dhcp snooping information option
```

This disables the DHCP Snooping **Option 82 information insertion** feature.

It is useful in Packet Tracer labs because some simulated DHCP environments can behave differently when DHCP Option 82 is inserted.

The important security configuration remains:

```cisco
ip dhcp snooping
ip dhcp snooping vlan 1
```

with the legitimate DHCP-server-facing uplinks configured as trusted.

---

# ✅ Final Verification Checklist

- [x] R1 configured as DHCP server.
- [x] DHCP pool `POOL1` created.
- [x] Network `192.168.1.0/24` configured.
- [x] `192.168.1.1–192.168.1.9` excluded.
- [x] R1 configured as default gateway.
- [x] DHCP Snooping enabled on SW1.
- [x] DHCP Snooping enabled for VLAN 1 on SW1.
- [x] SW1 G0/2 configured as trusted.
- [x] DHCP Snooping enabled on SW2.
- [x] DHCP Snooping enabled for VLAN 1 on SW2.
- [x] SW2 G0/1 configured as trusted.
- [x] PC1 tested using `ipconfig /renew`.
- [x] PC1 received `192.168.1.10`.
- [x] Default gateway received as `192.168.1.1`.
- [x] DHCP Snooping configuration verified.
- [x] DHCP server configuration verified.
- [x] Configuration saved using `write memory`.

---

# 🏁 Conclusion

This lab demonstrated the relationship between **DHCP Server configuration** and **DHCP Snooping**.

R1 provides DHCP services to clients on the `192.168.1.0/24` network, while SW1 and SW2 inspect DHCP traffic to protect the network from unauthorized DHCP servers.

The key concept is that **DHCP server-facing interfaces must be trusted**, while client-facing interfaces should normally remain untrusted.

The successful `ipconfig /renew` operation resulting in `192.168.1.10` confirmed that the DHCP server, switching infrastructure, and DHCP Snooping configuration were working together correctly.