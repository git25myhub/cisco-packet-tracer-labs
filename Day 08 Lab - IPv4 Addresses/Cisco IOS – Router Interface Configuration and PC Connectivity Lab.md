# Cisco IOS – Router Interface Configuration and PC Connectivity Lab

## 📌 Lab Overview

This lab focuses on basic Cisco IOS router configuration and end-device connectivity.

You will configure **R1**, assign IP addresses to its Gigabit Ethernet interfaces, add interface descriptions, verify the configuration using `show` commands, save the configuration, configure IP addressing on **PC1, PC2, and PC3**, and finally test end-to-end connectivity using `ping`.

### Cisco Device

- **Router:** Cisco 2911
- **IOS:** Cisco IOS C2900-UNIVERSALK9-M
- **IOS Version:** 15.1(4)M5
- **Interfaces:** GigabitEthernet0/0, GigabitEthernet0/1, GigabitEthernet0/2
- **Simulator:** Cisco Packet Tracer

---

## 🎯 Objectives

By the end of this lab, you should be able to:

- Configure a Cisco router hostname.
- Use `show` commands to inspect router interfaces.
- Configure IPv4 addresses on router interfaces.
- Enable router interfaces.
- Configure meaningful interface descriptions.
- Verify interface status and IP addressing.
- View and verify the running configuration.
- Save the running configuration to NVRAM.
- Configure IPv4 addresses on PCs in Packet Tracer.
- Test network connectivity using ICMP `ping`.

---

## 🗺️ Lab Topology

The topology consists of:

- **R1** – Cisco 2911 router
- **SW1** – Switch connected to R1
- **PC1**
- **PC2**
- **PC3**

R1 provides connectivity between the three different networks.

### R1 Interface Addressing

| Interface | IP Address | Subnet Mask | Description |
|---|---|---|---|
| G0/0 | `15.255.255.254` | `255.0.0.0` | `## to SW1 ##` |
| G0/1 | `182.98.255.254` | `255.255.0.0` | Appropriate LAN description |
| G0/2 | `201.191.20.254` | `255.255.255.0` | Appropriate LAN description |

> **Note:** The provided running configuration already shows the R1 interface addressing and the G0/0 description. If you are completing the lab from a fresh Packet Tracer topology, enter the configuration yourself rather than simply copying the final configuration.

---

# 1. Configure R1's Hostname

Enter privileged EXEC mode and then global configuration mode.

```text
R1> enable
R1# configure terminal
R1(config)# hostname R1
R1(config)# end
```

Verify the hostname:

```text
R1# show running-config
```

You should see:

```text
hostname R1
```

---

# 2. View R1's Interfaces

Use the `show ip interface brief` command to obtain a quick summary of the router's interfaces.

```text
R1# show ip interface brief
```

This command displays:

- Interface name
- IP address
- IP address assignment method
- Status
- Protocol state

Example:

```text
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     unassigned      YES unset  administratively down down
GigabitEthernet0/1     unassigned      YES unset  administratively down down
GigabitEthernet0/2     unassigned      YES unset  administratively down down
```

### Important

The following states are important:

- **administratively down/down** – the interface has been manually shut down.
- **down/down** – the interface is enabled but there is a physical/link problem.
- **up/up** – the interface is operational.

---

# 3. Configure R1's Interfaces

Configure the appropriate IP address, subnet mask, description, and enable each interface.

## GigabitEthernet0/0

```text
R1# configure terminal
R1(config)# interface gigabitEthernet 0/0
R1(config-if)# description ## to SW1 ##
R1(config-if)# ip address 15.255.255.254 255.0.0.0
R1(config-if)# no shutdown
R1(config-if)# exit
```

The `no shutdown` command enables the interface.

---

## GigabitEthernet0/1

```text
R1(config)# interface gigabitEthernet 0/1
R1(config-if)# description ## to PC2 LAN ##
R1(config-if)# ip address 182.98.255.254 255.255.0.0
R1(config-if)# no shutdown
R1(config-if)# exit
```

---

## GigabitEthernet0/2

```text
R1(config)# interface gigabitEthernet 0/2
R1(config-if)# description ## to PC3 LAN ##
R1(config-if)# ip address 201.191.20.254 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# exit
```

Return to privileged EXEC mode:

```text
R1(config)# end
```

---

# 4. Verify R1's Interfaces

Run:

```text
R1# show ip interface brief
```

The interfaces should now have their configured IP addresses.

Expected result:

```text
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     15.255.255.254  YES manual up                    up
GigabitEthernet0/1     182.98.255.254  YES manual up                    up
GigabitEthernet0/2     201.191.20.254  YES manual up                    up
```

The exact status may depend on whether the interfaces are physically connected to active devices in your Packet Tracer topology.

### Verify Interface Descriptions

You can also use:

```text
R1# show interfaces description
```

This provides a convenient summary of:

- Interface
- Status
- Protocol
- Description

---

# 5. View and Save the Running Configuration

Display the current configuration:

```text
R1# show running-config
```

Check that the following information is present:

```text
hostname R1
```

And verify the interface configurations:

```text
interface GigabitEthernet0/0
 description ## to SW1 ##
 ip address 15.255.255.254 255.0.0.0

interface GigabitEthernet0/1
 ip address 182.98.255.254 255.255.0.0

interface GigabitEthernet0/2
 ip address 201.191.20.254 255.255.255.0
```

The interfaces should also contain:

```text
no shutdown
```

### Save the Configuration

Save the running configuration to NVRAM:

```text
R1# copy running-config startup-config
```

When prompted:

```text
Destination filename [startup-config]?
```

Press **Enter**.

Alternatively, you can use:

```text
R1# write memory
```

or:

```text
R1# copy run start
```

The configuration should be saved successfully.

---

# 6. Configure IP Addresses on PC1, PC2, and PC3

In Cisco Packet Tracer, configure the PCs through:

**PC → Desktop → IP Configuration**

Assign each PC an IP address, subnet mask, and default gateway according to the addressing information provided by the lab topology.

The default gateway for a PC should normally be the IP address of the R1 interface connected to that PC's network.

### Example Addressing Structure

| Device | IP Address | Subnet Mask | Default Gateway |
|---|---|---|---|
| R1 G0/0 | `15.255.255.254` | `255.0.0.0` | — |
| R1 G0/1 | `182.98.255.254` | `255.255.0.0` | — |
| R1 G0/2 | `201.191.20.254` | `255.255.255.0` | — |
| PC1 | According to topology | According to topology | R1 |
| PC2 | According to topology | According to topology | R1 |
| PC3 | According to topology | According to topology | R1 |

> **Important:** Use the exact PC addressing shown in your Packet Tracer topology/lab instructions. Do not assign arbitrary addresses if the topology specifies particular values.

---

# 7. Test Connectivity

After configuring the PCs, open the **Command Prompt** on PC1.

Navigate to:

**PC1 → Desktop → Command Prompt**

First, test connectivity to PC2:

```text
ping <PC2-IP-address>
```

Example:

```text
ping 182.98.x.x
```

Then test connectivity to PC3:

```text
ping <PC3-IP-address>
```

Example:

```text
ping 201.191.20.x
```

A successful test should produce replies similar to:

```text
Reply from 182.98.x.x: bytes=32 time<1ms TTL=127
Reply from 182.98.x.x: bytes=32 time<1ms TTL=127
Reply from 182.98.x.x: bytes=32 time<1ms TTL=127
Reply from 182.98.x.x: bytes=32 time<1ms TTL=127
```

---

# 🔍 Troubleshooting

If the ping fails, verify the following.

### Check R1 interfaces

```text
R1# show ip interface brief
```

Make sure the required interfaces are:

```text
up                    up
```

### Check the running configuration

```text
R1# show running-config
```

Verify:

- Correct hostname
- Correct IP addresses
- Correct subnet masks
- Interface descriptions
- Interfaces are not shutdown

### Check an individual interface

```text
R1# show interfaces gigabitEthernet 0/0
```

Replace `0/0` with the required interface.

### Check PC configuration

On each PC, verify:

- IP address
- Subnet mask
- Default gateway

You can also use the PC command prompt:

```text
ipconfig
```

### Test the default gateway

From a PC:

```text
ping <R1-interface-IP>
```

If the PC cannot ping its gateway, check the PC's IP configuration, cable connection, switch connection, and R1 interface status.

---

# 🧪 Verification Checklist

- [ ] R1 hostname configured as `R1`
- [ ] `show ip interface brief` used to inspect interfaces
- [ ] R1 G0/0 configured with `15.255.255.254/8`
- [ ] R1 G0/1 configured with `182.98.255.254/16`
- [ ] R1 G0/2 configured with `201.191.20.254/24`
- [ ] Interface descriptions configured
- [ ] All required interfaces enabled with `no shutdown`
- [ ] Interface status verified
- [ ] Running configuration reviewed
- [ ] Configuration saved to startup-config
- [ ] PC1 IP configuration completed
- [ ] PC2 IP configuration completed
- [ ] PC3 IP configuration completed
- [ ] PC1 can ping PC2
- [ ] PC1 can ping PC3

---

# 📝 Key Commands

| Command | Purpose |
|---|---|
| `enable` | Enter privileged EXEC mode |
| `configure terminal` | Enter global configuration mode |
| `hostname R1` | Configure router hostname |
| `show ip interface brief` | Quickly view interface IP/status information |
| `interface g0/0` | Enter interface configuration mode |
| `ip address X.X.X.X Y.Y.Y.Y` | Assign an IPv4 address |
| `description ...` | Add an interface description |
| `no shutdown` | Enable an interface |
| `show interfaces description` | View interface descriptions and status |
| `show running-config` | Display active configuration |
| `copy running-config startup-config` | Save configuration |
| `ping X.X.X.X` | Test IP connectivity |

---

# ✅ Expected Result

At the end of the lab:

1. R1 should have the correct hostname.
2. All three Gigabit Ethernet interfaces should have their assigned IP addresses.
3. The interfaces should be enabled and operational.
4. Interface descriptions should identify their connections.
5. The running configuration should contain the changes.
6. The configuration should be saved to NVRAM.
7. PC1, PC2, and PC3 should have valid IP configurations.
8. **PC1 should successfully ping both PC2 and PC3.**

This lab demonstrates the fundamental Cisco IOS workflow:

```text
Configure → Verify → Test → Save
```