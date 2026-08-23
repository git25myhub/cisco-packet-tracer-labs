# Cisco DHCP Snooping & Dynamic ARP Inspection Lab

## 📌 Lab Overview

This lab demonstrates the configuration of **DHCP Snooping** and **Dynamic ARP Inspection (DAI)** on Cisco switches to protect the Layer 2 network against rogue DHCP servers and ARP-based attacks.

The lab uses **R1 as the DHCP server**, while **SW1 and SW2** enforce DHCP Snooping and DAI security policies.

### Objectives

By completing this lab, you will:

- Configure R1 as a DHCP server.
- Exclude reserved IP addresses from the DHCP pool.
- Configure DHCP Snooping on SW1 and SW2.
- Configure trusted DHCP Snooping interfaces.
- Configure Dynamic ARP Inspection on SW1 and SW2.
- Enable DAI validation checks for source MAC, destination MAC, and IP addresses.
- Trust switch/router uplinks for DAI.
- Verify DHCP addressing and network connectivity.

---

## 🖥️ Lab Topology

The logical topology consists of:

```text
             R1
        192.168.1.1/24
              |
            SW1
              |
            SW2
              |
             PC
```

### Device Information

| Device | Role | IOS |
|---|---|---|
| R1 | DHCP Server / Default Gateway | Cisco IOS 15.1(4)M5 |
| SW1 | DHCP Snooping + DAI | Cisco IOS 15.0(2)SE4 |
| SW2 | DHCP Snooping + DAI | Cisco IOS 15.0(2)SE4 |
| PC | DHCP Client | Packet Tracer PC |

---

# 1. Configure R1 as a DHCP Server

R1 provides DHCP services for the `192.168.1.0/24` network.

The first nine addresses are reserved and must not be assigned to clients.

### Requirements

- Network: `192.168.1.0/24`
- Excluded addresses: `192.168.1.1 - 192.168.1.9`
- Default gateway: `192.168.1.1`
- DHCP pool name: `POOL1`

### Configuration

```cisco
R1> enable
R1# configure terminal

R1(config)# ip dhcp excluded-address 192.168.1.1 192.168.1.9

R1(config)# ip dhcp pool POOL1
R1(dhcp-config)# network 192.168.1.0 255.255.255.0
R1(dhcp-config)# default-router 192.168.1.1
R1(dhcp-config)# exit
```

Save the configuration:

```cisco
R1(config)# end
R1# write memory
```

### Verify DHCP Configuration

```cisco
R1# show running-config
```

You should see:

```cisco
ip dhcp excluded-address 192.168.1.1 192.168.1.9

ip dhcp pool POOL1
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.1
```

You can also verify DHCP bindings with:

```cisco
R1# show ip dhcp binding
```

---

# 2. Configure DHCP Snooping on SW1

DHCP Snooping prevents unauthorized DHCP servers from responding to DHCP client requests.

Ports are **untrusted by default**. Only interfaces connected toward a legitimate DHCP server or another trusted network device should be configured as trusted.

### Enable DHCP Snooping for VLAN 1

```cisco
SW1> enable
SW1# configure terminal

SW1(config)# ip dhcp snooping
SW1(config)# ip dhcp snooping vlan 1
```

Disable DHCP Option 82 insertion for this Packet Tracer lab:

```cisco
SW1(config)# no ip dhcp snooping information option
```

### Trust the Appropriate Uplink

In this topology, `GigabitEthernet0/2` is connected toward the DHCP server/network infrastructure.

```cisco
SW1(config)# interface GigabitEthernet0/2
SW1(config-if)# ip dhcp snooping trust
SW1(config-if)# exit
```

Save:

```cisco
SW1(config)# end
SW1# write memory
```

### Verify DHCP Snooping

```cisco
SW1# show ip dhcp snooping
```

Also verify the trusted interface:

```cisco
SW1# show ip dhcp snooping interface
```

The DHCP server/uplink interface should show as **Trusted**, while client-facing interfaces should remain **Untrusted**.

---

# 3. Configure DHCP Snooping on SW2

SW2 must also inspect DHCP traffic because DHCP messages pass through the switch.

### Enable DHCP Snooping

```cisco
SW2> enable
SW2# configure terminal

SW2(config)# ip dhcp snooping
SW2(config)# ip dhcp snooping vlan 1
SW2(config)# no ip dhcp snooping information option
```

### Trust the Uplink

The connection toward SW1 is on `GigabitEthernet0/1`.

```cisco
SW2(config)# interface GigabitEthernet0/1
SW2(config-if)# ip dhcp snooping trust
SW2(config-if)# exit
```

Save the configuration:

```cisco
SW2(config)# end
SW2# write memory
```

### Verify

```cisco
SW2# show ip dhcp snooping
```

And:

```cisco
SW2# show ip dhcp snooping interface
```

`GigabitEthernet0/1` should be displayed as a trusted interface.

---

# 4. Configure Dynamic ARP Inspection on SW1

Dynamic ARP Inspection protects the network from ARP spoofing and ARP poisoning.

DAI validates ARP packets against trusted information learned through DHCP Snooping.

### Enable DAI for VLAN 1

```cisco
SW1# configure terminal

SW1(config)# ip arp inspection vlan 1
```

### Enable All Additional Validation Checks

The lab requires validation of:

- Source MAC address
- Destination MAC address
- IP address

```cisco
SW1(config)# ip arp inspection validate src-mac dst-mac ip
```

### Trust Network Device Ports

The ports connected to routers or switches must be trusted.

```cisco
SW1(config)# interface range GigabitEthernet0/1 - 2
SW1(config-if-range)# ip arp inspection trust
SW1(config-if-range)# exit
```

Save:

```cisco
SW1(config)# end
SW1# write memory
```

### Verify DAI

```cisco
SW1# show ip arp inspection
```

Check the interface trust state:

```cisco
SW1# show ip arp inspection interfaces
```

Network-device interfaces should show:

```text
Trust State: Trusted
```

Client-facing interfaces should remain:

```text
Trust State: Untrusted
```

---

# 5. Configure Dynamic ARP Inspection on SW2

Enable DAI on VLAN 1:

```cisco
SW2# configure terminal

SW2(config)# ip arp inspection vlan 1
```

Enable all requested validation checks:

```cisco
SW2(config)# ip arp inspection validate src-mac dst-mac ip
```

### Trust the Uplink

The uplink to SW1 is `GigabitEthernet0/1`.

```cisco
SW2(config)# interface GigabitEthernet0/1
SW2(config-if)# ip arp inspection trust
SW2(config-if)# exit
```

Save:

```cisco
SW2(config)# end
SW2# write memory
```

### Verify DAI

```cisco
SW2# show ip arp inspection
```

Check the interface trust state:

```cisco
SW2# show ip arp inspection interfaces
```

The uplink should be **Trusted**, while access ports remain **Untrusted**.

---

# 6. Verify the PC DHCP Address

The PC should obtain its address automatically from R1.

On the Packet Tracer PC:

```text
C:\> ipconfig
```

Expected result:

```text
IPv4 Address:    192.168.1.10
Subnet Mask:     255.255.255.0
Default Gateway: 192.168.1.1
```

The address `192.168.1.10` is the first available address because:

```text
192.168.1.1 - 192.168.1.9
```

were excluded from the DHCP pool.

---

# 7. Test Connectivity

From the PC, ping the default gateway:

```text
C:\> ping 192.168.1.1
```

Expected result:

```text
Reply from 192.168.1.1
Reply from 192.168.1.1
Reply from 192.168.1.1
Reply from 192.168.1.1
```

The successful ping confirms that the PC received valid Layer 3 addressing and can reach R1.

---

# 8. Verification Commands

## R1

Check DHCP configuration:

```cisco
show running-config
```

Check DHCP leases:

```cisco
show ip dhcp binding
```

Check DHCP pool statistics:

```cisco
show ip dhcp pool
```

---

## SW1 and SW2

### DHCP Snooping

```cisco
show ip dhcp snooping
```

```cisco
show ip dhcp snooping binding
```

```cisco
show ip dhcp snooping interface
```

### Dynamic ARP Inspection

```cisco
show ip arp inspection
```

```cisco
show ip arp inspection interfaces
```

```cisco
show ip arp inspection statistics
```

### Running Configuration

```cisco
show running-config
```

---

# 9. Expected Security Configuration

The final configuration should follow this security model:

```text
                  R1
             DHCP Server
            192.168.1.1
                  |
             TRUSTED PORT
                  |
                 SW1
          DHCP Snooping + DAI
                  |
             TRUSTED PORT
                  |
                 SW2
          DHCP Snooping + DAI
                  |
          UNTRUSTED ACCESS PORT
                  |
                 PC
```

### Trust Model

| Interface Type | DHCP Snooping | DAI |
|---|---|---|
| Router uplink | Trusted | Trusted |
| Switch uplink | Trusted | Trusted |
| PC/access port | Untrusted | Untrusted |

This prevents an ordinary client port from being used to introduce rogue DHCP or unauthorized ARP information into the network.

---

# 10. Final Verification Checklist

- [x] R1 configured as a DHCP server.
- [x] `192.168.1.1 - 192.168.1.9` excluded from DHCP.
- [x] DHCP pool `POOL1` configured.
- [x] Network `192.168.1.0/24` configured.
- [x] Default gateway `192.168.1.1` configured.
- [x] DHCP Snooping enabled on SW1.
- [x] DHCP Snooping enabled on SW2.
- [x] Appropriate uplinks configured as DHCP Snooping trusted ports.
- [x] DAI enabled on SW1 for VLAN 1.
- [x] DAI enabled on SW2 for VLAN 1.
- [x] Source MAC validation enabled.
- [x] Destination MAC validation enabled.
- [x] IP address validation enabled.
- [x] Router/switch uplinks configured as DAI trusted ports.
- [x] PC received `192.168.1.10` through DHCP.
- [x] PC successfully pinged `192.168.1.1`.
- [x] Configurations saved with `write memory`.

---

## 🧠 Key Concepts

### DHCP Snooping

DHCP Snooping acts as a security filter for DHCP traffic. It distinguishes between **trusted** and **untrusted** interfaces and prevents unauthorized DHCP servers from responding to clients.

### Dynamic ARP Inspection

DAI uses information learned by DHCP Snooping to validate ARP packets. This helps protect against:

- ARP spoofing
- ARP poisoning
- Man-in-the-middle attacks
- Unauthorized ARP mappings

### Why Uplinks Must Be Trusted

Legitimate DHCP responses and ARP traffic must be allowed to pass through the switch infrastructure. Therefore, interfaces connected to the legitimate DHCP server or another trusted switch/router are configured as trusted.

Client-facing interfaces remain untrusted to prevent end devices from injecting unauthorized DHCP or ARP traffic.

## Conclusion

This lab demonstrates how **DHCP Snooping and Dynamic ARP Inspection work together as Layer 2 security mechanisms**.

DHCP Snooping establishes trusted DHCP infrastructure and builds the DHCP binding database, while DAI uses that trusted information to validate ARP traffic. Together, they provide protection against common Layer 2 attacks while allowing legitimate DHCP and ARP communication to operate normally.