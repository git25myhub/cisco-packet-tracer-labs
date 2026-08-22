# Cisco CDP and LLDP Neighbor Discovery Lab

## Overview

This lab focuses on **network neighbor discovery and device identification** using Cisco Discovery Protocol (CDP) and Link Layer Discovery Protocol (LLDP).

The lab begins by using CDP and other IOS commands to identify missing interface IDs and IP addresses. CDP is then disabled on switch ports connected to end devices and globally on all network devices. Finally, LLDP is enabled globally, with LLDP transmission and reception enabled only on interfaces connecting network devices.

The lab uses Cisco Packet Tracer devices running Cisco IOS.

## Objectives

By completing this lab, you will:

1. Use CDP and other IOS commands to identify missing:
   - IP addresses
   - Interface IDs
   - Neighboring devices
2. Disable CDP on switch interfaces connected to PCs.
3. Disable CDP globally on every network device.
4. Enable LLDP globally on every network device.
5. Enable LLDP transmit and receive on interfaces connected to other network devices.
6. Verify the final CDP and LLDP configuration.

---

## Network Devices

The topology contains the following devices:

| Device | Type | IOS |
|---|---|---|
| R1 | Cisco 2911 Router | IOS 15.1 |
| R2 | Cisco 2911 Router | IOS 15.1 |
| R3 | Cisco 2911 Router | IOS 15.1 |
| SW1 | Cisco 2960 Switch | IOS 12.2(25)FX |
| SW2 | Cisco 2960 Switch | IOS 12.2(25)FX |
| SW3 | Cisco 2960 Switch | IOS 12.2(25)FX |

---

## 1. Identify Missing IP Addresses and Interfaces

Before disabling CDP, use CDP to discover neighboring devices and determine which interfaces are connected.

### Useful commands

```cisco
show cdp neighbors
show cdp neighbors detail
show ip interface brief
show ip route
show running-config
```

The most useful commands for this task are:

```cisco
show cdp neighbors
show cdp neighbors detail
```

### Example

On SW3:

```cisco
SW3#show cdp neighbors
```

Output:

```text
Device ID    Local Intrfce   Holdtme    Capability   Platform    Port ID
R3           Gig 0/1          177        R            C2900       Gig 0/0
```

This identifies the connection as:

```text
SW3 GigabitEthernet0/1
        |
        |
R3 GigabitEthernet0/0
```

The detailed CDP command can then be used to obtain additional information such as the neighbor's IP address:

```cisco
SW3#show cdp neighbors detail
```

### IP Address Information

The router configurations identified the following addressing:

| Device | Interface | IP Address | Subnet |
|---|---|---|---|
| R1 | G0/0 | 10.0.13.1 | 10.0.13.0/30 |
| R1 | G0/1 | 10.0.12.1 | 10.0.12.0/30 |
| R1 | G0/2 | 192.168.1.254 | 192.168.1.0/24 |
| R2 | G0/0 | 10.0.12.2 | 10.0.12.0/30 |
| R2 | G0/1 | 192.168.2.254 | 192.168.2.0/24 |
| R2 | G0/2 | 10.0.23.1 | 10.0.23.0/30 |
| R3 | G0/0 | 192.168.3.254 | 192.168.3.0/24 |
| R3 | G0/1 | 10.0.13.2 | 10.0.13.0/30 |
| R3 | G0/2 | 10.0.23.2 | 10.0.23.0/30 |

The point-to-point router connections are therefore:

```text
R1 G0/1  <---->  R2 G0/0
10.0.12.1       10.0.12.2

R1 G0/0  <---->  R3 G0/1
10.0.13.1       10.0.13.2

R2 G0/2  <---->  R3 G0/2
10.0.23.1       10.0.23.2
```

---

# 2. Disable CDP on Switch Ports Connected to PCs

CDP should not be unnecessarily exposed on interfaces connected to end-user PCs.

The command used is:

```cisco
interface <interface-id>
no cdp enable
```

### SW3

The PC-connected interface identified in the lab was FastEthernet0/24:

```cisco
SW3(config)#interface fastethernet0/24
SW3(config-if)#no cdp enable
```

### SW2

The PC-connected interface identified was FastEthernet0/1:

```cisco
SW2(config)#interface fastethernet0/1
SW2(config-if)#no cdp enable
```

### SW1

The PC-connected interface identified was FastEthernet0/10:

```cisco
SW1(config)#interface fastethernet0/10
SW1(config-if)#no cdp enable
```

### Save the configuration

```cisco
do write
```

or:

```cisco
end
write memory
```

---

# 3. Disable CDP Globally

After using CDP to identify the topology, CDP should be disabled globally on every network device.

The command is:

```cisco
no cdp run
```

### Switches

Apply the command on SW1, SW2, and SW3:

```cisco
SW1(config)#no cdp run
SW2(config)#no cdp run
SW3(config)#no cdp run
```

### Routers

Apply the same command on R1, R2, and R3:

```cisco
R1(config)#no cdp run
R2(config)#no cdp run
R3(config)#no cdp run
```

### Verify CDP is disabled

Use:

```cisco
show cdp
```

or:

```cisco
show running-config
```

The running configuration should contain:

```text
no cdp run
```

CDP neighbor discovery should no longer operate after CDP is disabled globally.

---

# 4. Enable LLDP Globally

LLDP is a vendor-neutral Layer 2 neighbor discovery protocol.

Enable LLDP globally with:

```cisco
lldp run
```

Apply this to every network device.

### Switches

```cisco
SW1(config)#lldp run
SW2(config)#lldp run
SW3(config)#lldp run
```

### Routers

```cisco
R1(config)#lldp run
R2(config)#lldp run
R3(config)#lldp run
```

---

# 5. Enable LLDP Transmit and Receive

The lab specifies that **LLDP transmit and receive are initially disabled on the interfaces**.

LLDP should therefore be enabled only on interfaces connected to other network devices.

The two commands are:

```cisco
lldp transmit
lldp receive
```

---

## SW1

Enable LLDP Tx/Rx on the network-device-facing interface:

```cisco
SW1(config)#interface gigabitethernet0/1
SW1(config-if)#lldp transmit
SW1(config-if)#lldp receive
```

---

## SW2

Enable LLDP Tx/Rx on the appropriate network-device-facing interface:

```cisco
SW2(config)#interface gigabitethernet0/2
SW2(config-if)#lldp transmit
SW2(config-if)#lldp receive
```

> Use the interface identified from the topology/CDP discovery phase if the physical topology differs.

---

## SW3

The lab identified GigabitEthernet0/1 as the connection toward R3:

```cisco
SW3(config)#interface gigabitethernet0/1
SW3(config-if)#lldp transmit
SW3(config-if)#lldp receive
```

---

## R1

R1 has three GigabitEthernet interfaces. LLDP Tx/Rx can be enabled on the router interfaces participating in network-device connections:

```cisco
R1(config)#interface range gigabitethernet0/0 - 2
R1(config-if-range)#lldp transmit
R1(config-if-range)#lldp receive
```

---

## R2

```cisco
R2(config)#interface range gigabitethernet0/0 - 2
R2(config-if-range)#lldp transmit
R2(config-if-range)#lldp receive
```

---

## R3

```cisco
R3(config)#interface range gigabitethernet0/0 - 2
R3(config-if-range)#lldp transmit
R3(config-if-range)#lldp receive
```

---

# 6. Verify LLDP

After enabling LLDP, verify the global configuration:

```cisco
show lldp
```

Then verify discovered neighbors:

```cisco
show lldp neighbors
```

For detailed neighbor information:

```cisco
show lldp neighbors detail
```

You can also verify individual interface settings with:

```cisco
show running-config interface gigabitethernet0/1
```

A correctly configured interface should contain:

```text
lldp transmit
lldp receive
```

---

# 7. Verification Commands

Use the following commands to verify the completed lab.

### Verify interfaces and IP addresses

```cisco
show ip interface brief
```

### Verify CDP status

```cisco
show cdp
```

Expected global configuration:

```text
no cdp run
```

### Verify LLDP status

```cisco
show lldp
```

Expected configuration:

```text
lldp run
```

### Verify LLDP neighbors

```cisco
show lldp neighbors
```

### Get detailed LLDP information

```cisco
show lldp neighbors detail
```

### Verify the complete configuration

```cisco
show running-config
```

---

# 8. Final Configuration Summary

The completed lab should follow these principles:

| Requirement | Implementation |
|---|---|
| Identify neighbors using CDP | `show cdp neighbors` |
| Identify IP addresses | `show cdp neighbors detail` / `show ip interface brief` |
| Disable CDP on PC ports | `no cdp enable` |
| Disable CDP globally | `no cdp run` |
| Enable LLDP globally | `lldp run` |
| Enable LLDP Tx | `lldp transmit` |
| Enable LLDP Rx | `lldp receive` |
| Verify LLDP neighbors | `show lldp neighbors` |
| Verify detailed LLDP data | `show lldp neighbors detail` |
| Save configuration | `write memory` |

---

# 9. Key IOS Commands

```cisco
! CDP discovery
show cdp neighbors
show cdp neighbors detail

! Interface/IP discovery
show ip interface brief
show running-config

! Disable CDP globally
no cdp run

! Disable CDP on an individual interface
interface <interface-id>
no cdp enable

! Enable LLDP globally
lldp run

! Enable LLDP on an interface
interface <interface-id>
lldp transmit
lldp receive

! Verify LLDP
show lldp
show lldp neighbors
show lldp neighbors detail

! Save configuration
write memory
```

---

# 10. Lab Outcome

At the end of the lab:

- CDP was used during the initial discovery phase to identify neighboring devices, interfaces, and IP addressing.
- CDP was disabled on the switch interfaces connected to PCs.
- CDP was disabled globally on all routers and switches.
- LLDP was enabled globally on all network devices.
- LLDP transmit and receive were enabled on network-device-facing interfaces.
- LLDP can now be used to discover neighboring network devices without relying on Cisco-proprietary CDP.

This provides a practical demonstration of the difference between **CDP**, which is Cisco proprietary, and **LLDP**, which is an open standards-based neighbor discovery protocol.