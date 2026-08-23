# Cisco Port Security Lab

## 📌 Lab Overview

This lab demonstrates how to configure and verify **Cisco switch port security** using different security policies on SW1 and SW2.

The lab focuses on:

- Configuring access ports for port security.
- Limiting the number of MAC addresses allowed on a port.
- Configuring MAC address aging.
- Comparing **Shutdown** and **Restrict** violation modes.
- Understanding **sticky MAC address learning**.
- Triggering port security violations.
- Observing switch behavior and security violation counters.
- Verifying port-security configuration and operational status.

---

## 🎯 Lab Objectives

By completing this lab, you should be able to:

1. Configure port security on switch access ports.
2. Configure maximum secure MAC addresses.
3. Configure port-security violation modes.
4. Configure MAC address aging.
5. Configure sticky MAC address learning.
6. Trigger and identify port-security violations.
7. Explain the difference between `shutdown` and `restrict` violation modes.
8. Verify port-security status using Cisco IOS commands.

---

## 🗺️ Lab Topology

The lab uses two Cisco switches:

- **SW1**
  - FastEthernet0/1
  - FastEthernet0/2
  - FastEthernet0/3

- **SW2**
  - GigabitEthernet0/1

The ports operate as **access ports in VLAN 1**, which is the default VLAN.

---

## 🔐 Port Security Requirements

### SW1

Configure port security on:

- `FastEthernet0/1`
- `FastEthernet0/2`
- `FastEthernet0/3`

| Setting | Requirement |
|---|---|
| Port Mode | Access |
| Access VLAN | VLAN 1 |
| Port Security | Enabled |
| Maximum MAC Addresses | 1 |
| Violation Mode | Shutdown |
| Sticky Learning | Disabled |
| Aging Time | 1 hour |
| Aging Type | Absolute |

### SW2

Configure port security on:

- `GigabitEthernet0/1`

| Setting | Requirement |
|---|---|
| Port Mode | Access |
| Access VLAN | VLAN 1 |
| Port Security | Enabled |
| Maximum MAC Addresses | 4 |
| Violation Mode | Restrict |
| Sticky Learning | Enabled |
| Aging Time | Default / 0 minutes |

---

# ⚙️ SW1 Configuration

Enter configuration mode:

```cisco
SW1> enable
SW1# configure terminal
```

Configure the three FastEthernet interfaces as access ports and enable port security:

```cisco
SW1(config)# interface range fastethernet 0/1 - 3
SW1(config-if-range)# switchport mode access
SW1(config-if-range)# switchport port-security
SW1(config-if-range)# switchport port-security maximum 1
SW1(config-if-range)# switchport port-security violation shutdown
SW1(config-if-range)# switchport port-security aging time 60
SW1(config-if-range)# exit
```

Save the configuration:

```cisco
SW1(config)# end
SW1# write memory
```

### Important

Sticky learning is **not configured on SW1**.

Therefore, the switch does not dynamically create sticky secure MAC addresses on these ports.

---

# 🔎 Verify SW1 Port Security

Check the configuration:

```cisco
SW1# show running-config
```

Check port-security status:

```cisco
SW1# show port-security
```

Check an individual interface:

```cisco
SW1# show port-security interface fastethernet 0/1
```

Expected output includes:

```text
Port Security              : Enabled
Port Status                : Secure-up
Violation Mode             : Shutdown
Aging Time                 : 60 mins
Aging Type                 : Absolute
SecureStatic Address Aging : Disabled
Maximum MAC Addresses      : 1
```

You can repeat the verification for F0/2 and F0/3:

```cisco
SW1# show port-security interface fastethernet 0/2
SW1# show port-security interface fastethernet 0/3
```

---

# 🚨 Triggering a Port-Security Violation on SW1

The maximum number of secure MAC addresses on each SW1 port is **1**.

Connect a device and allow its MAC address to be learned.

Then introduce another MAC address on the same interface, for example by connecting another PC or changing the connected device.

The switch should detect a violation.

Check:

```cisco
SW1# show port-security interface fastethernet 0/1
```

The resulting status should resemble:

```text
Port Security              : Enabled
Port Status                : Secure-shutdown
Violation Mode             : Shutdown
Maximum MAC Addresses      : 1
Security Violation Count   : 1
```

The switch also generates messages similar to:

```text
%PM-4-ERR_DISABLE: psecure-violation error detected on Fa0/1,
putting Fa0/1 in err-disable state.

%PORT_SECURITY-2-PSECURE_VIOLATION:
Security violation occurred, caused by MAC address ...
```

### Result

Because the violation mode is **shutdown**, SW1:

1. Detects the unauthorized MAC address.
2. Generates a port-security violation.
3. Places the interface into an **err-disabled** state.
4. Stops forwarding traffic through the affected port.

This explains why connectivity was lost during the test.

---

# 🔧 Recovering a SW1 Interface

After resolving the violation, the interface can be manually recovered.

```cisco
SW1# configure terminal
SW1(config)# interface fastethernet 0/1
SW1(config-if)# shutdown
SW1(config-if)# no shutdown
SW1(config-if)# end
```

Verify:

```cisco
SW1# show interfaces fastethernet 0/1 status
SW1# show port-security interface fastethernet 0/1
```

---

# ⚙️ SW2 Configuration

Enter configuration mode:

```cisco
SW2> enable
SW2# configure terminal
```

Configure G0/1:

```cisco
SW2(config)# interface gigabitethernet 0/1
SW2(config-if)# switchport mode access
SW2(config-if)# switchport port-security
SW2(config-if)# switchport port-security maximum 4
SW2(config-if)# switchport port-security mac-address sticky
SW2(config-if)# switchport port-security violation restrict
SW2(config-if)# exit
```

Save the configuration:

```cisco
SW2(config)# end
SW2# write memory
```

---

# 🧲 Sticky MAC Address Learning

SW2 uses:

```cisco
switchport port-security mac-address sticky
```

This allows the switch to dynamically learn MAC addresses and convert them into **sticky secure MAC addresses**.

The maximum number of secure MAC addresses is:

```text
4
```

Therefore, G0/1 can have up to four secure MAC addresses.

Verify:

```cisco
SW2# show port-security interface gigabitethernet 0/1
```

The observed configuration showed:

```text
Port Security              : Enabled
Port Status                : Secure-up
Violation Mode             : Restrict
Maximum MAC Addresses      : 4
Total MAC Addresses        : 4
Sticky MAC Addresses       : 4
Security Violation Count   : 0
```

The learned addresses can also be seen in the running configuration:

```cisco
SW2# show running-config
```

For example:

```text
switchport port-security mac-address sticky 0001.0001.0001
switchport port-security mac-address sticky 0002.0002.0002
switchport port-security mac-address sticky 0003.0003.0003
switchport port-security mac-address sticky 0060.471C.1D19
```

---

# 🔎 Verify SW2 MAC Address Table

Use:

```cisco
SW2# show mac address-table
```

The lab showed four secure MAC addresses associated with G0/1:

```text
Vlan    Mac Address       Type      Ports
----    -----------       --------  -----
1       0001.0001.0001    STATIC    Gig0/1
1       0002.0002.0002    STATIC    Gig0/1
1       0003.0003.0003    STATIC    Gig0/1
1       0060.471c.1d19    STATIC    Gig0/1
```

---

# 🚨 Triggering a Port-Security Violation on SW2

Once four secure MAC addresses have been learned, introduce another device/MAC address on G0/1.

For example, the lab generated a violation from:

```text
0060.7024.2366
```

The switch generated:

```text
%PORT_SECURITY-2-PSECURE_VIOLATION:
Security violation occurred, caused by MAC address
0060.7024.2366 on port GigabitEthernet0/1.
```

Because SW2 uses:

```cisco
switchport port-security violation restrict
```

the interface **remains operational**.

Verify:

```cisco
SW2# show port-security interface gigabitethernet 0/1
```

The lab produced:

```text
Port Security              : Enabled
Port Status                : Secure-up
Violation Mode             : Restrict
Maximum MAC Addresses      : 4
Total MAC Addresses        : 4
Sticky MAC Addresses       : 4
Security Violation Count   : 6
```

---

# 📊 Comparing SW1 and SW2

| Feature | SW1 | SW2 |
|---|---|---|
| Interface | F0/1–F0/3 | G0/1 |
| Port Mode | Access | Access |
| VLAN | 1 | 1 |
| Port Security | Enabled | Enabled |
| Maximum MACs | 1 | 4 |
| Sticky Learning | Disabled | Enabled |
| Violation Mode | Shutdown | Restrict |
| Aging | 60 minutes | Default |
| Violation Result | Err-disabled | Traffic from violating MAC dropped |
| Interface Status After Violation | Secure-shutdown | Secure-up |
| Security Counter | Increments | Increments |

---

# 🔥 Shutdown vs Restrict

## Shutdown

SW1 uses:

```cisco
switchport port-security violation shutdown
```

When a violation occurs, the switch places the interface into an **err-disabled** state.

Typical result:

```text
Secure-shutdown
```

Traffic is stopped until the interface is recovered.

### Advantage

Provides a strong response against unauthorized devices.

### Disadvantage

A single violation can cause complete loss of connectivity on the port.

---

## Restrict

SW2 uses:

```cisco
switchport port-security violation restrict
```

When a violation occurs:

- The violating traffic is dropped.
- The interface remains operational.
- The security violation counter increases.
- Syslog messages are generated.

The port therefore remains:

```text
Secure-up
```

### Advantage

Provides security while keeping the interface operational.

### Disadvantage

It does not completely shut down the port when an unauthorized MAC address appears.

---

# 🧪 Connectivity Testing

Before triggering the violations, connectivity was confirmed using:

```text
C:\> ping 10.0.0.254
```

Successful connectivity produced:

```text
Reply from 10.0.0.254
```

After the SW1 violation, connectivity failed:

```text
Request timed out.
Request timed out.
Request timed out.
Request timed out.
```

This confirms the effect of the **shutdown** violation mode.

After the port was restored, connectivity returned:

```text
Reply from 10.0.0.254
```

This demonstrates the practical difference between normal operation, a port-security shutdown, and subsequent recovery.

---

# 🛠️ Useful Verification Commands

### Display all port-security interfaces

```cisco
show port-security
```

### Display detailed port-security information

```cisco
show port-security interface fastethernet 0/1
```

```cisco
show port-security interface gigabitethernet 0/1
```

### Display secure MAC addresses

```cisco
show port-security address
```

### Display the MAC address table

```cisco
show mac address-table
```

### Display the running configuration

```cisco
show running-config
```

### Check interface status

```cisco
show interfaces status
```

### Check err-disabled interfaces

```cisco
show interfaces status err-disabled
```

### Save configuration

```cisco
write memory
```

or:

```cisco
copy running-config startup-config
```

---

# 📝 Lab Observations

### SW1

- Port security was enabled on F0/1–F0/3.
- Each port allowed only one secure MAC address.
- Aging was configured for 60 minutes.
- Sticky learning was disabled.
- A second MAC address triggered a security violation.
- The interface entered an **err-disabled / secure-shutdown** state.
- Connectivity was lost while the interface was disabled.
- The security violation counter increased.

### SW2

- Port security was enabled on G0/1.
- The port allowed four secure MAC addresses.
- Sticky learning dynamically learned secure MAC addresses.
- A fifth unauthorized MAC address triggered violations.
- The interface remained **Secure-up**.
- The violating traffic was restricted rather than shutting down the port.
- The security violation counter increased.
- Connectivity on the legitimate secure MAC addresses remained available.

---

# 📚 Key Concepts Learned

### Port Security

Port security controls which MAC addresses are permitted to use a switch port.

### Maximum MAC Addresses

Defines how many secure MAC addresses can exist on an interface.

### Sticky MAC Learning

Allows the switch to dynamically learn MAC addresses and retain them as secure addresses.

### Violation Mode

Defines what the switch does when an unauthorized MAC address is detected.

### Aging

Allows secure MAC addresses to be removed after a specified period, depending on the configured aging behavior.

### Err-Disabled State

A protected interface can be automatically disabled when a port-security violation occurs under the shutdown violation mode.

---

# ✅ Final Verification Checklist

- [x] SW1 F0/1 configured for port security.
- [x] SW1 F0/2 configured for port security.
- [x] SW1 F0/3 configured for port security.
- [x] SW1 maximum MAC address limit set to 1.
- [x] SW1 violation mode set to Shutdown.
- [x] SW1 aging time set to 60 minutes.
- [x] Sticky learning disabled on SW1.
- [x] SW2 G0/1 configured for port security.
- [x] SW2 maximum MAC address limit set to 4.
- [x] SW2 violation mode set to Restrict.
- [x] Sticky MAC learning enabled on SW2.
- [x] SW1 violation successfully triggered.
- [x] SW1 interface entered err-disabled state.
- [x] SW2 violation successfully triggered.
- [x] SW2 remained operational.
- [x] Security violation counters verified.
- [x] MAC address tables verified.
- [x] Connectivity tested before and after violations.
- [x] Configurations saved to startup-config.

---

## 🏁 Conclusion

This lab demonstrated how Cisco **Port Security** can be used to control access to switch interfaces based on MAC addresses.

The most important comparison was between the two violation modes:

**SW1 — Shutdown:**  
An unauthorized MAC address caused the port to enter an err-disabled state, resulting in loss of connectivity.

**SW2 — Restrict:**  
An unauthorized MAC address was rejected and logged, but the interface remained operational for legitimate secure MAC addresses.

The lab therefore demonstrates why port-security settings should be selected according to the desired balance between **network availability and security enforcement**.