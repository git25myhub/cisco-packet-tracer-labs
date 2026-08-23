# Wireless LANs — WLC, Dynamic Interfaces & WPA2-PSK

## 📌 Lab Overview

This lab introduces **Wireless LAN Controller (WLC)** configuration and centralized wireless network management using Cisco Packet Tracer.

The lab begins by accessing the **WLC1 web GUI** from PC1, exploring the available management tabs, and reviewing the current network state. The WLC is then configured with dynamic interfaces for the **Internal** and **Guest** WLANs. Two WLANs are created using **WPA2-PSK**, and a wireless client is finally associated with an access point.

The underlying switch configuration provides three VLANs:

- **VLAN 10** — Management
- **VLAN 100** — Internal WLAN
- **VLAN 200** — Guest WLAN

---

## 🎯 Lab Objectives

- Access the WLC1 GUI using HTTPS.
- Log in to the WLC using the provided credentials.
- Familiarize yourself with the WLC management interface.
- Identify the current network state from the WLC GUI.
- Configure WLC dynamic interfaces for Internal and Guest WLANs.
- Create Internal and Guest WLANs.
- Configure WPA2-PSK security.
- Associate a wireless client with an access point.
- Verify wireless client connectivity and DHCP operation.

---

## 🗺️ Network Topology

```text
                         WLC1
                          |
                       VLAN 10
                    Management
                          |
                        SW1
                  G1/0/1 Trunk
                   /     |      \
                  /      |       \
                AP1     PC1      AP2
                          |
                    Management
                    /    |     \
               VLAN 100 VLAN 200
               Internal  Guest
                          |
                    Smartphone1
```

---

## 📋 VLAN and IP Addressing

| VLAN | Name | Network | Default Gateway | Purpose |
|---|---|---|---|---|
| 10 | Management | `172.16.1.0/24` | `172.16.1.1` | WLC/AP/Switch management |
| 100 | Internal | `10.0.0.0/24` | `10.0.0.1` | Internal wireless users |
| 200 | Guest | `10.1.0.0/24` | `10.1.0.1` | Guest wireless users |

### WLC Management

```text
WLC1 Management IP: 172.16.1.10
Management VLAN:    VLAN 10
```

The switch configuration uses DHCP Option 43 to provide the WLC address to lightweight access points:

```cisco
ip dhcp pool VLAN10
 network 172.16.1.0 255.255.255.0
 default-router 172.16.1.1
 option 43 ip 172.16.1.10
```

---

# 1. Access the WLC1 Web GUI

Use **PC1** to access the WLC management interface.

Open:

```text
PC1 → Desktop → Web Browser
```

Because the lab requires HTTPS, enter:

```text
https://172.16.1.10
```

> **Important:** Do not use `http://`. The lab specifically requires an HTTPS connection.

### Login Credentials

```text
Username: admin
Password: Cisco123
```

After successfully logging in, the WLC management GUI should appear.

---

# 2. Familiarize Yourself with the WLC GUI

Spend some time exploring the WLC GUI before making configuration changes.

The exact tabs available may vary slightly depending on the Packet Tracer WLC model/version, but common areas include:

| GUI Area | Purpose |
|---|---|
| **Monitor** | View system status, clients, APs, WLANs and network activity |
| **WLANs** | Create and manage wireless LANs/SSIDs |
| **Controller** | Configure controller-wide settings and interfaces |
| **Wireless** | View and manage access points and wireless parameters |
| **Management** | Configure controller management and administrative settings |

### Questions to Consider

While exploring the GUI, determine:

- How many access points are currently visible?
- Are the APs joined to the WLC?
- What is the WLC management IP address?
- What interfaces are currently configured?
- What WLANs currently exist?
- What clients are currently associated?
- What is the current network status?
- Which VLAN is being used for WLC management?

The management network in this lab is:

```text
VLAN 10
172.16.1.0/24
```

---

# 3. Configure Dynamic Interfaces

The WLC needs separate dynamic interfaces for the Internal and Guest WLANs.

These interfaces map wireless traffic to the appropriate VLANs.

## Dynamic Interface — Internal

Create a dynamic interface for the Internal WLAN.

Suggested values:

```text
Interface Name: Internal
VLAN ID:        100
IP Address:     10.0.0.2
Subnet Mask:    255.255.255.0
Default Gateway: 10.0.0.1
```

The exact WLC GUI fields may differ depending on the Packet Tracer version.

The important relationship is:

```text
Internal WLAN
      ↓
Dynamic Interface
      ↓
VLAN 100
      ↓
10.0.0.0/24
```

---

## Dynamic Interface — Guest

Create a second dynamic interface for the Guest WLAN.

Suggested values:

```text
Interface Name: Guest
VLAN ID:        200
IP Address:     10.1.0.2
Subnet Mask:    255.255.255.0
Default Gateway: 10.1.0.1
```

The relationship is:

```text
Guest WLAN
      ↓
Dynamic Interface
      ↓
VLAN 200
      ↓
10.1.0.0/24
```

---

# 4. Verify the Switch Trunk

SW1 must carry all required VLANs toward the WLC.

The trunk interface is:

```text
GigabitEthernet1/0/1
```

The existing configuration is:

```cisco
interface GigabitEthernet1/0/1
 switchport trunk native vlan 10
 switchport trunk allowed vlan 10,100,200
 switchport mode trunk
```

This is important because the WLC needs to send and receive traffic for:

```text
VLAN 10  → Management
VLAN 100 → Internal WLAN
VLAN 200 → Guest WLAN
```

Verify the trunk with:

```cisco
SW1#show interfaces trunk
```

You should see VLANs `10,100,200` allowed on the trunk.

---

# 5. Verify VLAN Configuration on SW1

Check the VLAN database:

```cisco
SW1#show vlan brief
```

Expected VLANs:

```text
10    Management
100   Internal
200   Guest
```

The switch also has Layer 3 interfaces for these VLANs:

```cisco
interface Vlan10
 ip address 172.16.1.1 255.255.255.0

interface Vlan100
 ip address 10.0.0.1 255.255.255.0

interface Vlan200
 ip address 10.1.0.1 255.255.255.0
```

---

# 6. Verify DHCP Configuration

SW1 is acting as the DHCP server for the three VLANs.

### Management VLAN

```cisco
ip dhcp pool VLAN10
 network 172.16.1.0 255.255.255.0
 default-router 172.16.1.1
 option 43 ip 172.16.1.10
```

Option 43 tells lightweight APs where to find the WLC.

### Internal VLAN

```cisco
ip dhcp pool VLAN100
 network 10.0.0.0 255.255.255.0
 default-router 10.0.0.1
```

### Guest VLAN

```cisco
ip dhcp pool VLAN200
 network 10.1.0.0 255.255.255.0
 default-router 10.1.0.1
```

Excluded addresses include the first ten addresses in each subnet.

Verify DHCP operation:

```cisco
SW1#show ip dhcp binding
```

---

# 7. Create the Internal WLAN

From the WLC GUI, navigate to the WLAN configuration section and create a new WLAN.

Use values similar to:

```text
WLAN Name:     Internal
SSID:          Internal
WLAN ID:       10
```

Associate the WLAN with the previously created **Internal dynamic interface**.

The WLAN should map to:

```text
SSID:     Internal
VLAN:     100
Network:  10.0.0.0/24
Gateway:  10.0.0.1
```

---

# 8. Configure WPA2-PSK for Internal WLAN

Configure the Internal WLAN security using:

```text
Security: WPA2
Authentication: Personal / PSK
Encryption: AES
```

Choose a strong pre-shared key.

For a Packet Tracer lab, you can use:

```text
Internal@123
```

The resulting wireless security should be conceptually:

```text
Internal SSID
     |
     └── WPA2-PSK
            |
            └── AES
```

---

# 9. Create the Guest WLAN

Create a second WLAN for guest users.

Suggested values:

```text
WLAN Name:     Guest
SSID:          Guest
WLAN ID:       20
```

Associate the WLAN with the **Guest dynamic interface**.

The WLAN should map to:

```text
SSID:     Guest
VLAN:     200
Network:  10.1.0.0/24
Gateway: 10.1.0.1
```

---

# 10. Configure WPA2-PSK for Guest WLAN

Configure the Guest WLAN with:

```text
Security: WPA2
Authentication: Personal / PSK
Encryption: AES
```

For the lab, use:

```text
Guest@123
```

The resulting configuration is:

```text
Guest SSID
     |
     └── WPA2-PSK
            |
            └── AES
```

---

# 11. Add a Wireless Client

Use **Smartphone1** as the wireless client.

Open the smartphone's wireless configuration and scan for available networks.

You should eventually see the configured SSIDs:

```text
Internal
Guest
```

Select the appropriate WLAN and enter its WPA2-PSK password.

For example:

```text
SSID: Internal
Password: Internal@123
```

The client should associate with one of the lightweight access points.

---

# 12. Verify Wireless Association

From the WLC GUI, check the wireless client information.

Verify:

```text
Client status: Associated
AP: AP1 or AP2
WLAN/SSID: Internal or Guest
Authentication: WPA2-PSK
```

The client should receive an IP address from the appropriate DHCP pool.

### Internal Client

Expected addressing:

```text
Network:        10.0.0.0/24
Default Gateway: 10.0.0.1
```

### Guest Client

Expected addressing:

```text
Network:        10.1.0.0/24
Default Gateway: 10.1.0.1
```

---

# 13. Verify DHCP and Connectivity

After the wireless client associates with the AP, verify that it receives an IP address.

For an Internal client, an address should be assigned from:

```text
10.0.0.0/24
```

For a Guest client:

```text
10.1.0.0/24
```

The corresponding default gateway should be:

```text
Internal → 10.0.0.1
Guest    → 10.1.0.1
```

Test connectivity to the VLAN gateway where supported.

For Internal:

```text
ping 10.0.0.1
```

For Guest:

```text
ping 10.1.0.1
```

---

# 🔍 Useful Verification Commands

## Check VLANs

```cisco
SW1#show vlan brief
```

## Check Trunk

```cisco
SW1#show interfaces trunk
```

## Check SVI Status

```cisco
SW1#show ip interface brief
```

## Check DHCP Pools

```cisco
SW1#show ip dhcp pool
```

## Check DHCP Leases

```cisco
SW1#show ip dhcp binding
```

## Check Running Configuration

```cisco
SW1#show running-config
```

---

# 🧠 Key Concepts Learned

### Wireless LAN Controller

A **WLC** provides centralized management of lightweight access points and wireless networks.

Instead of configuring each AP individually, the WLC can centrally manage:

- WLANs/SSIDs
- Wireless security
- AP configuration
- Client associations
- Dynamic interfaces
- Wireless policies

### Dynamic Interfaces

Dynamic interfaces connect WLANs to specific VLANs.

In this lab:

```text
Internal WLAN → Dynamic Interface → VLAN 100
Guest WLAN    → Dynamic Interface → VLAN 200
```

### Management VLAN

VLAN 10 is used for management:

```text
172.16.1.0/24
```

The WLC management address is:

```text
172.16.1.10
```

### Trunking

The switch-to-WLC connection must carry all required VLANs:

```text
VLAN 10
VLAN 100
VLAN 200
```

This is why G1/0/1 is configured as a trunk.

### WPA2-PSK

WPA2-PSK provides wireless authentication using a shared pre-shared key.

The lab uses:

```text
Internal → WPA2-PSK
Guest    → WPA2-PSK
```

---

# 🛠️ Troubleshooting

### WLC GUI Cannot Be Accessed

Check:

```text
PC1 → 172.16.1.10
```

Verify that PC1 has management VLAN connectivity and that you are using:

```text
https://172.16.1.10
```

not HTTP.

---

### AP Does Not Join the WLC

Check DHCP Option 43:

```cisco
SW1#show running-config
```

The management DHCP pool should contain:

```cisco
option 43 ip 172.16.1.10
```

Also verify that VLAN 10 is carried across the trunk.

---

### Wireless Client Does Not Receive an IP Address

Check:

```cisco
SW1#show ip dhcp binding
```

Then verify:

- The correct WLAN is enabled.
- The WLAN is mapped to the correct dynamic interface.
- The dynamic interface is mapped to the correct VLAN.
- VLAN 100/200 is allowed on the trunk.
- The DHCP pool exists.
- The client successfully authenticates with WPA2-PSK.

---

### Client Connects but Gets the Wrong Network

Check the WLAN-to-interface mapping.

The correct mappings are:

```text
Internal WLAN → VLAN 100 → 10.0.0.0/24

Guest WLAN → VLAN 200 → 10.1.0.0/24
```

---

# 💾 Save Configuration

Save the switch configuration:

```cisco
SW1#copy running-config startup-config
```

or:

```cisco
SW1#write
```

Also save the Packet Tracer project after completing the WLC configuration.

---

# ✅ Lab Completion Checklist

- [x] Access WLC1 from PC1 using HTTPS.
- [x] Log in using `admin / Cisco123`.
- [x] Explore the WLC GUI.
- [x] Identify the current network state.
- [x] Verify VLAN 10 management connectivity.
- [x] Verify the WLC management address `172.16.1.10`.
- [x] Verify the WLC/AP trunk carries VLANs `10,100,200`.
- [x] Configure the Internal dynamic interface.
- [x] Configure the Guest dynamic interface.
- [x] Create the Internal WLAN.
- [x] Configure WPA2-PSK for the Internal WLAN.
- [x] Create the Guest WLAN.
- [x] Configure WPA2-PSK for the Guest WLAN.
- [x] Associate a wireless client with an AP.
- [x] Verify the wireless client receives a DHCP address.
- [x] Verify the client is associated with the correct WLAN.
- [x] Test connectivity to the appropriate VLAN gateway.
- [x] Save the configuration.

---

## 🎯 Final Result

The completed lab demonstrates a basic **controller-based wireless LAN architecture**.

The WLC manages the wireless networks while SW1 provides VLAN routing and DHCP services. The Internal and Guest WLANs are separated into different VLANs, with WPA2-PSK protecting wireless access.

```text
                    WLC1
               172.16.1.10
                    |
                    | 802.1Q Trunk
                    | VLAN 10,100,200
                    |
                   SW1
             _______|_______
            |       |       |
          VLAN10  VLAN100  VLAN200
         Mgmt     Internal  Guest
                    |         |
                   APs       APs
                    |
              Wireless Client
```

This provides centralized wireless management, VLAN-based traffic separation, DHCP-based addressing, and WPA2-protected wireless access.