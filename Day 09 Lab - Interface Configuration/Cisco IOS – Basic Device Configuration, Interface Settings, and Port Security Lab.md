# Cisco IOS – Basic Device Configuration, Interface Settings, and Port Security Lab

## 📌 Lab Overview

This lab focuses on fundamental Cisco IOS device and interface configuration.

You will configure the hostnames of **R1, SW1, and SW2**, assign the appropriate IP addresses to **R1 and the four PCs**, manually configure **speed and duplex** on interfaces connected to other networking devices, add meaningful **interface descriptions**, and shut down unused interfaces.

The lab uses a Cisco **2911 router** and **2960 switches** in Cisco Packet Tracer.

---

## 🎯 Objectives

By completing this lab, you will learn how to:

- Configure device hostnames.
- Configure IPv4 addresses on router interfaces and PCs.
- Manually configure Ethernet speed.
- Manually configure Ethernet duplex.
- Add descriptions to interfaces.
- Identify interfaces connected to networking devices.
- Disable unused switch and router interfaces.
- Verify interface configuration and status.
- Use Cisco IOS commands to inspect the configuration.

---

# 🖥️ Devices Used

| Device | Model | IOS |
|---|---|---|
| R1 | Cisco 2911 | C2900-UNIVERSALK9-M 15.1(4)M5 |
| SW1 | Cisco 2960-24TT | C2960-LANBASE-M 12.2(25)FX |
| SW2 | Cisco 2960-24TT | C2960-LANBASE-M 12.2(25)FX |
| PC1 | End Host | Packet Tracer |
| PC2 | End Host | Packet Tracer |
| PC3 | End Host | Packet Tracer |
| PC4 | End Host | Packet Tracer |

---

# 🗺️ Lab Topology

The topology contains:

```text
                  R1
                  |
                  |
                 SW1
                /   \
              PC1   SW2
                   /   \
                 PC3   PC4
```

The exact physical interface connections should be followed from the Packet Tracer topology.

### Important distinction

The lab requires **manual speed and duplex configuration only on interfaces connected to other networking devices**.

Therefore:

- Router ↔ Switch: manually configure speed/duplex.
- Switch ↔ Switch: manually configure speed/duplex.
- Switch ↔ PC: do **not** manually configure speed/duplex unless the lab specifically requires it.

---

# 1. Configure Device Hostnames

Configure the hostname on each device.

## R1

```text id="q5lq0s"
R1> enable
R1# configure terminal
R1(config)# hostname R1
R1(config)# end
```

Verify:

```text id="ks2cv1"
R1# show running-config
```

---

## SW1

```text id="5hby1g"
Switch> enable
Switch# configure terminal
Switch(config)# hostname SW1
SW1(config)# end
```

Verify:

```text id="2h6k2j"
SW1# show running-config
```

---

## SW2

```text id="3qz3at"
Switch> enable
Switch# configure terminal
Switch(config)# hostname SW2
SW2(config)# end
```

Verify:

```text id="q1f0jk"
SW2# show running-config
```

---

# 2. Configure IP Addresses

## R1 IP Address

According to the provided configuration, R1 uses:

| Interface | IP Address | Subnet Mask |
|---|---|---|
| G0/0 | `172.16.255.254` | `255.255.0.0` |

Configure it with:

```text id="8hj8o4"
R1# configure terminal
R1(config)# interface gigabitEthernet 0/0
R1(config-if)# ip address 172.16.255.254 255.255.0.0
R1(config-if)# no shutdown
R1(config-if)# exit
R1(config)# end
```

Verify:

```text id="ak4jhz"
R1# show ip interface brief
```

Expected:

```text id="5eqd4m"
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     172.16.255.254  YES manual up                    up
```

---

# 3. Configure PC IP Addresses

Configure the PCs through:

**PC → Desktop → IP Configuration**

Enter the IP address, subnet mask, and default gateway specified by the lab topology.

The default gateway should normally be the IP address of the R1 interface serving the PC's network.

### PC Configuration Template

| Device | IP Address | Subnet Mask | Default Gateway |
|---|---|---|---|
| PC1 | According to topology | According to topology | R1 |
| PC2 | According to topology | According to topology | R1 |
| PC3 | According to topology | According to topology | R1 |
| PC4 | According to topology | According to topology | R1 |

> **Note:** The console output supplied with the lab does not include the exact PC IP addresses. Use the addresses provided in the Packet Tracer topology or original lab instructions rather than inventing values.

---

# 4. Manually Configure Speed and Duplex

Ethernet interfaces can automatically negotiate their speed and duplex settings. In this lab, however, interfaces connected to **other networking devices** must be manually configured.

The provided R1 configuration demonstrates:

```text id="l9j7jt"
interface GigabitEthernet0/0
 description ## to SW1 ##
 ip address 172.16.255.254 255.255.0.0
 duplex full
 speed 1000
```

This means:

- Duplex: **Full**
- Speed: **1000 Mbps**

---

## R1 G0/0

```text id="kq7s9e"
R1# configure terminal
R1(config)# interface gigabitEthernet 0/0
R1(config-if)# duplex full
R1(config-if)# speed 1000
R1(config-if)# end
```

---

## SW1 Interfaces Connected to Networking Devices

According to the supplied configuration, SW1 has manually configured Gigabit interfaces:

```text id="m1l2rm"
interface GigabitEthernet0/1
 duplex full
 speed 1000

interface GigabitEthernet0/2
 duplex full
 speed 1000
```

Configure the appropriate networking-device-facing interfaces:

```text id="bq1d7n"
SW1# configure terminal

SW1(config)# interface gigabitEthernet 0/1
SW1(config-if)# duplex full
SW1(config-if)# speed 1000
SW1(config-if)# exit

SW1(config)# interface gigabitEthernet 0/2
SW1(config-if)# duplex full
SW1(config-if)# speed 1000
SW1(config-if)# exit

SW1(config)# end
```

> **Important:** Confirm which physical interfaces connect to R1, SW2, or another networking device before applying these commands. Interface numbers can vary depending on the topology.

---

## SW2 Networking-Device-Facing Interface

The supplied SW2 configuration shows:

```text id="o6j9j6"
interface GigabitEthernet0/1
 duplex full
 speed 1000
```

Configure the appropriate interface:

```text id="f0kqba"
SW2# configure terminal
SW2(config)# interface gigabitEthernet 0/1
SW2(config-if)# duplex full
SW2(config-if)# speed 1000
SW2(config-if)# end
```

---

# 5. Configure Interface Descriptions

Interface descriptions make it easier to identify what each interface connects to.

## R1

The provided configuration uses:

```text id="k0q5c3"
R1(config)# interface gigabitEthernet 0/0
R1(config-if)# description ## to SW1 ##
```

For unused interfaces:

```text id="s0q2fk"
R1(config)# interface gigabitEthernet 0/1
R1(config-if)# description ## not in use ##
R1(config-if)# exit

R1(config)# interface gigabitEthernet 0/2
R1(config-if)# description ## not in use ##
R1(config-if)# exit
```

---

## SW2 End-Host Interfaces

The supplied configuration uses:

```text id="x7v7yx"
SW2(config)# interface fastEthernet 0/1
SW2(config-if)# description ## to end hosts ##
SW2(config-if)# exit

SW2(config)# interface fastEthernet 0/2
SW2(config-if)# description ## to end hosts ##
SW2(config-if)# exit
```

Use descriptions that clearly identify the connected device where appropriate.

Examples:

```text id="zqv7u8"
description ## to R1 ##
description ## to SW1 ##
description ## to PC1 ##
description ## to PC2 ##
description ## to end hosts ##
description ## not in use ##
```

---

# 6. Disable Unused Interfaces

Unused interfaces should be administratively shut down.

This reduces unnecessary active ports and makes the configuration easier to manage.

---

## R1 Unused Interfaces

The provided configuration shows G0/1 and G0/2 as unused:

```text id="a9xk9p"
R1# configure terminal

R1(config)# interface gigabitEthernet 0/1
R1(config-if)# description ## not in use ##
R1(config-if)# shutdown
R1(config-if)# exit

R1(config)# interface gigabitEthernet 0/2
R1(config-if)# description ## not in use ##
R1(config-if)# shutdown
R1(config-if)# exit

R1(config)# end
```

---

## SW1 Unused FastEthernet Ports

SW1 has 24 FastEthernet ports.

According to the supplied configuration, Fa0/1 and Fa0/2 are active, while Fa0/3 through Fa0/24 are unused.

You can configure them individually:

```text id="efv5yq"
SW1# configure terminal

SW1(config)# interface range fastEthernet 0/3 - 24
SW1(config-if-range)# shutdown
SW1(config-if-range)# end
```

This is much faster than configuring every port individually.

---

## SW2 Unused FastEthernet Ports

According to the supplied configuration, Fa0/1 and Fa0/2 are used for end hosts.

Therefore, Fa0/3 through Fa0/24 should be disabled:

```text id="upb8jx"
SW2# configure terminal
SW2(config)# interface range fastEthernet 0/3 - 24
SW2(config-if-range)# shutdown
SW2(config-if-range)# end
```

The supplied configuration confirms these ports are shutdown.

---

## Unused Gigabit Interfaces

SW2's G0/2 is also shown as unused:

```text id="5x1o7j"
SW2# configure terminal
SW2(config)# interface gigabitEthernet 0/2
SW2(config-if)# description ## not in use ##
SW2(config-if)# shutdown
SW2(config-if)# end
```

---

# 7. Verify Interface Configuration

After configuring the devices, verify the interfaces.

## R1

```text id="m9y4d7"
R1# show ip interface brief
```

You should see G0/0 operational and G0/1/G0/2 administratively down if they are unused.

Also use:

```text id="1e8z0b"
R1# show interfaces description
```

---

## SW1

Use:

```text id="9u8p4j"
SW1# show interfaces status
```

This provides information about:

- Port
- Status
- VLAN
- Duplex
- Speed
- Type

You can also use:

```text id="s0x8cn"
SW1# show interfaces description
```

---

## SW2

Run:

```text id="w2x5n8"
SW2# show interfaces status
```

And:

```text id="v7k8b5"
SW2# show interfaces description
```

---

# 8. Verify Speed and Duplex

To inspect a specific interface:

```text id="3q6y1v"
R1# show interfaces gigabitEthernet 0/0
```

Look for information similar to:

```text id="q8y0wt"
Full-duplex, 1000Mb/s
```

On the switches:

```text id="k7w5n4"
SW1# show interfaces gigabitEthernet 0/1
```

and:

```text id="0ef2xc"
SW2# show interfaces gigabitEthernet 0/1
```

Verify that the configured networking-device-facing interfaces use:

```text id="n6n5a4"
Full-duplex
1000 Mb/s
```

---

# 9. Verify Shutdown Interfaces

Use:

```text id="d5d6d2"
show ip interface brief
```

on R1 and:

```text id="x7j4e9"
show interfaces status
```

on the switches.

Unused interfaces should appear as administratively disabled/shutdown.

For example, R1 should show:

```text id="q5r1d0"
GigabitEthernet0/1     ...     administratively down    down
GigabitEthernet0/2     ...     administratively down    down
```

---

# 10. Save the Configuration

After completing the configuration, save it on each Cisco device.

### R1

```text id="w4z9d6"
R1# copy running-config startup-config
```

### SW1

```text id="n3d1f8"
SW1# copy running-config startup-config
```

### SW2

```text id="b5c3h1"
SW2# copy running-config startup-config
```

Press **Enter** when prompted for the destination filename.

---

# 🔍 Useful Verification Commands

| Command | Purpose |
|---|---|
| `show running-config` | Display current configuration |
| `show startup-config` | Display saved configuration |
| `show ip interface brief` | View router interface IP/status |
| `show interfaces status` | View switch port status, speed, and duplex |
| `show interfaces description` | View interface descriptions |
| `show interfaces g0/0` | Detailed interface information |
| `show version` | Display IOS and hardware information |
| `copy running-config startup-config` | Save configuration |

---

# 🧪 Troubleshooting

### Interface is down

Check:

```text id="8p7y2m"
show ip interface brief
```

If it shows:

```text id="0x0zcv"
administratively down
```

the interface has been shutdown.

Enable it with:

```text id="4kj5o4"
interface g0/0
no shutdown
```

Only do this if the interface is supposed to be active.

---

### Connectivity problems

Check:

1. Correct IP address.
2. Correct subnet mask.
3. Correct default gateway.
4. Correct physical connections.
5. Interface status.
6. Speed and duplex settings.
7. Unused interfaces have not accidentally been shut down.

---

### Speed/Duplex mismatch

For networking-device connections, make sure both ends use compatible settings.

For example:

```text id="6n9xqp"
R1:
duplex full
speed 1000
```

and the corresponding switch interface:

```text id="j8k2lm"
SW1:
duplex full
speed 1000
```

A mismatch can cause connectivity and performance problems.

---

# 📝 Configuration Summary

## R1

```text id="0t3f8v"
hostname R1

interface GigabitEthernet0/0
 description ## to SW1 ##
 ip address 172.16.255.254 255.255.0.0
 duplex full
 speed 1000
 no shutdown

interface GigabitEthernet0/1
 description ## not in use ##
 shutdown

interface GigabitEthernet0/2
 description ## not in use ##
 shutdown
```

---

## SW1

The important configuration elements are:

```text id="qj8jts"
hostname SW1

interface GigabitEthernet0/1
 duplex full
 speed 1000

interface GigabitEthernet0/2
 duplex full
 speed 1000
```

Unused FastEthernet interfaces:

```text id="w4a5np"
interface range FastEthernet0/3 - 24
 shutdown
```

---

## SW2

```text id="1qj4ab"
hostname SW2

interface FastEthernet0/1
 description ## to end hosts ##

interface FastEthernet0/2
 description ## to end hosts ##

interface GigabitEthernet0/1
 duplex full
 speed 1000

interface GigabitEthernet0/2
 description ## not in use ##
 shutdown

interface range FastEthernet0/3 - 24
 shutdown
```

---

# ✅ Lab Completion Checklist

- [ ] R1 hostname configured as `R1`
- [ ] SW1 hostname configured as `SW1`
- [ ] SW2 hostname configured as `SW2`
- [ ] R1 G0/0 configured with `172.16.255.254/16`
- [ ] PC1 IP address configured
- [ ] PC2 IP address configured
- [ ] PC3 IP address configured
- [ ] PC4 IP address configured
- [ ] Speed manually configured on networking-device connections
- [ ] Duplex manually configured on networking-device connections
- [ ] Interface descriptions added
- [ ] Unused R1 interfaces shutdown
- [ ] Unused SW1 interfaces shutdown
- [ ] Unused SW2 interfaces shutdown
- [ ] Interface status verified
- [ ] Speed and duplex verified
- [ ] Running configurations checked
- [ ] Configurations saved to startup-config

---

# 🎓 Key Concepts

### Hostname

Identifies a Cisco device in the CLI:

```text id="0e2v8d"
hostname R1
```

### Interface Description

Documents what an interface connects to:

```text id="e6v9xq"
description ## to SW1 ##
```

### Speed

Sets Ethernet interface speed:

```text id="d2s6k9"
speed 1000
```

### Duplex

Sets the transmission mode:

```text id="p6t5n8"
duplex full
```

### Shutdown

Administratively disables an interface:

```text id="k2v8c1"
shutdown
```

### No Shutdown

Enables an interface:

```text id="m4q7r2"
no shutdown
```

---

# 🏁 Expected Result

At the end of the lab:

- **R1, SW1, and SW2** should have the correct hostnames.
- **R1 and all PCs** should have the appropriate IP addressing.
- Interfaces connecting **networking devices** should have manually configured speed and duplex.
- Interfaces should contain meaningful descriptions.
- Unused interfaces should be administratively disabled.
- Active interfaces should be operational.
- The configuration should be verified and saved.

The core workflow for this lab is:

```text
Configure
   ↓
Describe
   ↓
Set Speed/Duplex
   ↓
Disable Unused Ports
   ↓
Verify
   ↓
Save
```