# Cisco VTP, Trunking & Access Port Configuration Lab

## 📌 Lab Overview

This lab demonstrates the configuration and verification of **VTP (VLAN Trunking Protocol)** across three Cisco switches:

- `SW1` — VTP Server
- `SW2` — VTP Transparent
- `SW3` — VTP Client

The lab also covers:

- 802.1Q trunk configuration
- Disabling DTP negotiation
- Verifying administrative and operational switchport modes
- VTP domain configuration
- VLAN propagation
- VTP transparent behavior
- VTP client restrictions
- Manual access-port configuration
- Verification using Cisco IOS commands

The main purpose is to understand how VLAN information behaves differently under the three VTP operating modes.

---

# 🎯 Lab Objectives

By completing this lab, the following objectives should be achieved:

1. Configure switch-to-switch links as trunk ports.
2. Disable DTP on trunk interfaces using `switchport nonegotiate`.
3. Verify the administrative and operational mode of trunk interfaces.
4. Configure `SW1` as a VTP server in the `CCNA` domain.
5. Create VLANs 10, 20, and 30 on `SW1`.
6. Verify that VLANs 10, 20, and 30 are propagated to the other switches.
7. Configure `SW2` as a VTP transparent switch.
8. Create VLAN 40 locally on `SW2`.
9. Determine whether VLAN 40 appears automatically on `SW1` and `SW3`.
10. Configure `SW3` as a VTP client.
11. Attempt to create VLAN 50 on `SW3`.
12. Verify that VLAN creation is rejected on a VTP client.
13. Configure host-facing switchports as static access ports.
14. Determine whether DTP remains enabled on manually configured access ports.
15. Verify the final switch configurations.

---

# 🗺️ Network Design

The lab consists of three interconnected switches:

```text
             Trunk
       +----------------+
       |                |
     SW1              SW2
 VTP Server       VTP Transparent
       |                |
       |                |
       +-------SW3------+
          VTP Client
```

The switches use the following VTP roles:

| Switch | VTP Mode | VTP Domain |
|---|---|---|
| SW1 | Server | CCNA |
| SW2 | Transparent | CCNA |
| SW3 | Client | CCNA |

### VLANs

| VLAN | Purpose |
|---|---|
| VLAN 10 | Host network |
| VLAN 20 | Host network |
| VLAN 30 | Host network |
| VLAN 40 | Local VLAN on SW2 |
| VLAN 50 | Test VLAN on SW3 |

---

# 🔗 Part 1 — Configure Switch-to-Switch Trunks

Switch-to-switch connections must be configured as trunk ports so that multiple VLANs can traverse the links.

The lab also requires **DTP to be disabled**.

DTP is disabled using:

```cisco
switchport nonegotiate
```

This prevents the interface from sending DTP negotiation frames.

---

## SW1 Trunk Configuration

The connection from SW1 to another switch uses `GigabitEthernet0/1`.

```cisco
enable
configure terminal

interface gigabitEthernet0/1
 switchport mode trunk
 switchport nonegotiate
exit

end
write memory
```

### Final SW1 Interface Configuration

```text
interface GigabitEthernet0/1
 switchport mode trunk
 switchport nonegotiate
```

---

## SW2 Trunk Configuration

SW2 has two switch-to-switch connections.

```cisco
configure terminal

interface gigabitEthernet0/1
 switchport mode trunk
 switchport nonegotiate
exit

interface gigabitEthernet0/2
 switchport mode trunk
 switchport nonegotiate
exit

end
write memory
```

The final configuration shown in the lab confirms:

```text
interface GigabitEthernet0/1
 switchport mode trunk
 switchport nonegotiate

interface GigabitEthernet0/2
 switchport mode trunk
 switchport nonegotiate
```

---

## SW3 Trunk Configuration

The switch-to-switch connection on SW3 is configured as:

```cisco
configure terminal

interface gigabitEthernet0/1
 switchport mode trunk
 switchport nonegotiate
exit

end
write memory
```

---

# 🔍 Verify Trunk Administrative and Operational Modes

The following command can be used to inspect a switchport:

```cisco
show interfaces gigabitEthernet0/1 switchport
```

A more specific interface can also be inspected:

```cisco
show interfaces fa0/1 switchport
```

Important fields include:

```text
Administrative Mode
Operational Mode
Administrative Trunking Encapsulation
Operational Trunking Encapsulation
Negotiation of Trunking
```

For a manually configured trunk, the expected result is:

```text
Administrative Mode: static trunk
Operational Mode: trunk
Negotiation of Trunking: Off
```

The exact wording may vary slightly depending on the Packet Tracer switch model and IOS version.

---

# 🧠 Understanding DTP

**DTP (Dynamic Trunking Protocol)** allows Cisco switches to negotiate whether a link should operate as a trunk.

In this lab, trunking is manually configured:

```cisco
switchport mode trunk
```

Therefore, there is no need for DTP negotiation.

DTP is disabled using:

```cisco
switchport nonegotiate
```

This creates a more predictable configuration because the trunk state is explicitly configured rather than negotiated.

---

# 🌐 Part 2 — Configure SW1 as VTP Server

By default, SW1 initially showed:

```text
VTP Operating Mode              : Server
VTP Domain Name                 :
Configuration Revision          : 0
```

The VTP domain was then configured as:

```cisco
configure terminal

vtp domain CCNA
```

The switch confirmed:

```text
Changing VTP domain name from NULL to CCNA
```

---

# 🏷️ Create VLANs on SW1

VLANs 10, 20, and 30 were created on SW1:

```cisco
vlan 10
vlan 20
vlan 30
exit
```

The VTP status subsequently showed:

```text
Configuration Revision          : 3
Number of existing VLANs        : 8
VTP Operating Mode              : Server
VTP Domain Name                 : CCNA
```

The configuration revision increased because three VLANs were added.

---

# 🔄 VLAN Propagation Through VTP

Because SW1 is operating as a VTP server and the switches are connected through trunks, VLAN information can be propagated to compatible VTP switches in the same domain.

The VLANs created on SW1 were:

```text
VLAN 10
VLAN 20
VLAN 30
```

On SW2, before changing to transparent mode, the VLAN database showed:

```text
10   VLAN0010   active
20   VLAN0020   active
30   VLAN0030   active
```

SW3 also showed:

```text
10   VLAN0010   active
20   VLAN0020   active
30   VLAN0030   active
```

### Result

**Yes. VLANs 10, 20, and 30 were propagated to SW2 and SW3 while they were participating in the VTP domain.**

---

# 🔹 Part 3 — Configure SW2 as VTP Transparent

SW2 was changed from VTP Server mode to Transparent mode:

```cisco
configure terminal

vtp mode transparent
```

The switch confirmed:

```text
Setting device to VTP TRANSPARENT mode.
```

Verification:

```cisco
show vtp status
```

Expected result:

```text
VTP Operating Mode              : Transparent
VTP Domain Name                 : CCNA
```

---

# 🏷️ Create VLAN 40 on SW2

Because SW2 is now in transparent mode, VLANs created on the switch are locally significant.

VLAN 40 was created:

```cisco
vlan 40
exit
```

Verification showed:

```text
Number of existing VLANs        : 9
VTP Operating Mode              : Transparent
VTP Domain Name                 : CCNA
```

The running configuration also contained:

```text
vlan 40
```

---

# ❓ Does VLAN 40 Appear on SW1/SW3?

**No.**

VLAN 40 is locally configured on SW2 because SW2 is operating in **VTP transparent mode**.

The important distinction is:

```text
VTP Server
     ↓
Can distribute VLAN information

VTP Transparent
     ↓
Maintains its own VLAN database
Does not synchronize its VLAN database through VTP

VTP Client
     ↓
Receives VLAN information
Cannot create VLANs locally
```

Therefore:

> VLAN 40 exists on SW2, but it is not automatically added to the VLAN database of SW1 or SW3.

---

# 💻 Assign VLAN 40 to SW2 Host Ports

The first two FastEthernet interfaces on SW2 were manually assigned to VLAN 40:

```cisco
configure terminal

interface range fastEthernet0/1 - 2
 switchport mode access
 switchport access vlan 40
exit

end
write memory
```

The running configuration confirms:

```text
interface FastEthernet0/1
 switchport access vlan 40
 switchport mode access

interface FastEthernet0/2
 switchport access vlan 40
 switchport mode access
```

---

# 🔹 Part 4 — Configure SW3 as VTP Client

SW3 was initially showing the VLANs learned from the VTP environment:

```text
10   VLAN0010
20   VLAN0020
30   VLAN0030
```

SW3 was then changed to VTP client mode:

```cisco
configure terminal

vtp mode client
```

The switch confirmed:

```text
Setting device to VTP CLIENT mode.
```

Verification:

```cisco
show vtp status
```

The expected operating mode is:

```text
VTP Operating Mode : Client
```

---

# 🚫 Attempt to Create VLAN 50 on SW3

The lab requires testing whether a VTP client can create VLANs.

The following command was attempted:

```cisco
vlan 50
```

SW3 rejected the command with:

```text
VTP VLAN configuration not allowed when device is in CLIENT mode.
```

### Result

**VLAN 50 was not created.**

This demonstrates an important characteristic of VTP client mode:

> A VTP client cannot independently create, modify, or delete VLANs from its local VLAN configuration.

VLAN information must be received from a VTP server.

---

# 🖥️ Part 5 — Configure Host-Facing Ports

Host-facing ports should be manually configured as access ports.

This prevents them from attempting to negotiate trunking with end devices.

---

# SW1 Host Ports

### VLAN 10

```cisco
interface range fastEthernet0/1 - 2
 switchport mode access
 switchport access vlan 10
exit
```

### VLAN 20

```cisco
interface fastEthernet0/3
 switchport mode access
 switchport access vlan 20
exit
```

The resulting configuration was:

```text
interface FastEthernet0/1
 switchport access vlan 10
 switchport mode access

interface FastEthernet0/2
 switchport access vlan 10
 switchport mode access

interface FastEthernet0/3
 switchport access vlan 20
 switchport mode access
```

---

# SW2 Host Ports

SW2 host ports `Fa0/1` and `Fa0/2` were assigned to VLAN 40:

```cisco
interface range fastEthernet0/1 - 2
 switchport mode access
 switchport access vlan 40
exit
```

---

# SW3 Host Ports

SW3 has the following host assignments:

| Interface | VLAN |
|---|---|
| Fa0/1 | VLAN 10 |
| Fa0/2 | VLAN 30 |
| Fa0/3 | VLAN 30 |
| Fa0/4 | VLAN 20 |

Configuration:

```cisco
interface fastEthernet0/1
 switchport mode access
 switchport access vlan 10
exit

interface range fastEthernet0/2 - 3
 switchport mode access
 switchport access vlan 30
exit

interface fastEthernet0/4
 switchport mode access
 switchport access vlan 20
exit
```

---

# ❓ Is DTP Still Enabled on Host Ports?

The lab specifically asks whether DTP remains enabled after configuring the ports as access ports.

On SW3, verification of `Fa0/1` produced:

```text
Name: Fa0/1
Switchport: Enabled
Administrative Mode: static access
Operational Mode: static access
Administrative Trunking Encapsulation: dot1q
Operational Trunking Encapsulation: native
Negotiation of Trunking: Off
Access Mode VLAN: 10 (VLAN0010)
```

The important line is:

```text
Negotiation of Trunking: Off
```

### Result

**DTP negotiation is off on the statically configured access port.**

The port is explicitly configured with:

```cisco
switchport mode access
```

Therefore, it cannot dynamically negotiate itself into a trunk.

---

# 🔍 Verification Commands

## Verify VTP Status

```cisco
show vtp status
```

Useful information includes:

```text
VTP Version
Configuration Revision
Number of existing VLANs
VTP Operating Mode
VTP Domain Name
VTP Pruning Mode
```

---

## Verify VLAN Database

```cisco
show vlan brief
```

This verifies:

- VLAN existence
- VLAN status
- Access-port assignments

---

## Verify Interface Mode

```cisco
show interfaces switchport
```

Or for a specific interface:

```cisco
show interfaces fastEthernet0/1 switchport
```

---

## Verify Trunks

```cisco
show interfaces trunk
```

This confirms:

- Trunk status
- Encapsulation
- Native VLAN
- Allowed VLANs
- Active VLANs

---

## Verify Running Configuration

```cisco
show running-config
```

---

# 📊 Final VTP Configuration

| Switch | VTP Mode | Domain | VLAN Creation |
|---|---|---|---|
| SW1 | Server | CCNA | Allowed |
| SW2 | Transparent | CCNA | Allowed locally |
| SW3 | Client | CCNA | Not allowed |

---

# 📊 VLAN Behavior Summary

| Action | SW1 | SW2 | SW3 |
|---|---|---|---|
| VLAN 10 created on SW1 | Exists | Receives | Receives |
| VLAN 20 created on SW1 | Exists | Receives | Receives |
| VLAN 30 created on SW1 | Exists | Receives | Receives |
| VLAN 40 created on SW2 | No | Exists | No |
| VLAN 50 created on SW3 | N/A | N/A | Rejected |

---

# 🧠 Key Concepts Learned

## VTP Server

A VTP server can:

- Create VLANs
- Modify VLANs
- Delete VLANs
- Advertise VLAN information to other VTP switches

In this lab:

```text
SW1 = VTP Server
Domain = CCNA
```

---

## VTP Transparent

A VTP transparent switch:

- Maintains its own VLAN database
- Does not synchronize its VLAN database with VTP advertisements
- Can locally create VLANs
- Can locally modify VLANs
- Can locally delete VLANs

In this lab:

```text
SW2 = VTP Transparent
```

Therefore:

```text
VLAN 40 → SW2 only
```

---

## VTP Client

A VTP client:

- Receives VLAN information
- Uses VLAN information learned through VTP
- Cannot create VLANs locally

In this lab, attempting:

```cisco
vlan 50
```

on SW3 produced:

```text
VTP VLAN configuration not allowed when device is in CLIENT mode.
```

---

# ⚠️ Important VTP Considerations

VTP can simplify VLAN administration by allowing VLAN information to be centrally distributed. However, it also means that a change made on a VTP server can affect multiple switches.

For this reason, it is important to understand:

- VTP domain names
- VTP operating modes
- Configuration revisions
- Trunk connectivity
- VLAN propagation
- VTP client restrictions

Always verify VTP status before making VLAN changes.

Useful command:

```cisco
show vtp status
```

---

# 🧪 Troubleshooting Performed

## Issue 1 — VTP Domain Was Initially Empty

SW1 initially displayed:

```text
VTP Domain Name :
```

The domain was configured with:

```cisco
vtp domain CCNA
```

After configuration:

```text
VTP Domain Name : CCNA
```

---

## Issue 2 — VLANs Were Propagated to SW2/SW3

After VLANs 10, 20, and 30 were created on SW1, they appeared on the other switches because they were participating in the VTP domain.

This confirmed that:

```text
SW1 → VTP Server
SW2/SW3 → VTP participants
```

---

## Issue 3 — VLAN 40 Was Not Propagated

After SW2 was changed to transparent mode:

```cisco
vtp mode transparent
```

VLAN 40 was created locally.

It did not become part of the VLAN database of the other switches.

This demonstrated the local nature of VTP transparent mode.

---

## Issue 4 — VLAN 50 Creation Failed on SW3

SW3 was changed to client mode:

```cisco
vtp mode client
```

Attempting:

```cisco
vlan 50
```

returned:

```text
VTP VLAN configuration not allowed when device is in CLIENT mode.
```

This confirmed that VLAN creation is restricted on VTP clients.

---

# 💾 Saving Configurations

Configurations were saved throughout the lab using:

```cisco
write memory
```

or:

```cisco
do write
```

The expected confirmation is:

```text
Building configuration...

[OK]
```

---

# ✅ Final Lab Checklist

- [x] Switch-to-switch interfaces configured as trunks
- [x] DTP disabled using `switchport nonegotiate`
- [x] Administrative and operational modes verified
- [x] SW1 configured as VTP Server
- [x] VTP domain configured as `CCNA`
- [x] VLAN 10 created on SW1
- [x] VLAN 20 created on SW1
- [x] VLAN 30 created on SW1
- [x] VLANs 10, 20 and 30 propagated to participating switches
- [x] SW2 configured as VTP Transparent
- [x] VLAN 40 created locally on SW2
- [x] VLAN 40 confirmed as locally significant
- [x] SW3 configured as VTP Client
- [x] VLAN 50 creation attempted on SW3
- [x] VLAN 50 creation rejected as expected
- [x] Host-facing ports manually configured as access ports
- [x] Access VLANs assigned correctly
- [x] DTP negotiation verified as off on statically configured access ports
- [x] Configurations saved

---

# 🏁 Conclusion

This lab demonstrated the differences between **VTP Server, Transparent, and Client modes** while reinforcing the importance of correctly configuring trunk links and access ports.

The final topology can be summarized as:

```text
                 VTP Domain: CCNA

                    SW1
                VTP SERVER
             VLANs 10,20,30
                    |
              802.1Q Trunk
              DTP Disabled
                    |
                    |
                    SW2
             VTP TRANSPARENT
                  VLAN 40
                    |
              802.1Q Trunk
              DTP Disabled
                    |
                    SW3
                VTP CLIENT
             VLANs 10,20,30
```

The key findings from the lab were:

1. **VLANs created on a VTP server can be propagated to VTP participants.**
2. **VTP Transparent mode maintains its own local VLAN database.**
3. **VLAN 40 created on SW2 did not propagate to SW1 or SW3.**
4. **A VTP client cannot create VLANs locally.**
5. **VLAN 50 creation on SW3 was correctly rejected.**
6. **Switch-to-switch links were manually configured as trunks.**
7. **DTP was disabled using `switchport nonegotiate`.**
8. **Host-facing interfaces were manually configured as static access ports.**
9. **Static access ports showed `Negotiation of Trunking: Off`.**

This lab provides a practical foundation for understanding VLAN administration, VTP behavior, trunking, DTP, and switchport configuration in Cisco IOS.