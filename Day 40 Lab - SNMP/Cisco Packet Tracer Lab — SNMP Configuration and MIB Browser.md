# Cisco Packet Tracer Lab — SNMP Configuration and MIB Browser

## Overview

This lab demonstrates the basics of **Simple Network Management Protocol (SNMP)** using Cisco Packet Tracer. The objective is to configure SNMP communities on a Cisco router, use a PC's **MIB Browser** to retrieve information from the router with SNMP `Get` messages, and modify the router's hostname using an SNMP `Set` message.

> **Important:** SNMP functionality in Cisco Packet Tracer is **very limited** compared with real Cisco IOS environments. Some SNMP features, MIB objects, and operations may not behave exactly as they would on physical or virtual Cisco devices.

---

## Lab Topology

The topology consists of:

```text
192.168.1.0/24

        G0/0
R1 -------------- SW1 -------------- PC1
.254                                .1
```

### Device Addressing

| Device | Interface | IP Address |
|---|---|---|
| R1 | G0/0 | `192.168.1.254/24` |
| SW1 | — | — |
| PC1 | NIC | `192.168.1.1/24` |

PC1 uses the router's `192.168.1.254` address to communicate with R1's SNMP agent.

---

# Objectives

By the end of this lab, you should be able to:

- Configure SNMP read-only and read/write community strings.
- Understand the purpose of SNMP community strings.
- Use the Packet Tracer MIB Browser.
- Retrieve information from a Cisco router using SNMP `Get` messages.
- Identify the router's system uptime.
- Retrieve the configured hostname.
- Determine the number of interfaces on the router.
- Identify the router's interfaces.
- Explore additional information available through SNMP.
- Modify the router's hostname using an SNMP `Set` operation.
- Verify SNMP configuration from the Cisco IOS CLI.

---

# SNMP Community Configuration

Two SNMP communities are required:

| Community | Access | Purpose |
|---|---|---|
| `Cisco1` | Read-only (`RO`) | Retrieve information using SNMP `Get` |
| `Cisco2` | Read-write (`RW`) | Retrieve and modify supported SNMP objects |

---

## Step 1 — Configure SNMP on R1

Enter global configuration mode on R1:

```cisco
R1# configure terminal
R1(config)# snmp-server community Cisco1 ro
R1(config)# snmp-server community Cisco2 rw
```

### Explanation

The first command creates the read-only community:

```cisco
snmp-server community Cisco1 ro
```

The `ro` keyword gives the community **read-only access**.

The second command creates the read/write community:

```cisco
snmp-server community Cisco2 rw
```

The `rw` keyword gives the community **read-write access**, allowing supported SNMP objects to be modified.

Packet Tracer may display an SNMP warm-start message after enabling SNMP:

```text
%SNMP-5-WARMSTART: SNMP agent on host R1 is undergoing a warm start
```

This indicates that the SNMP agent has started or restarted.

---

## Step 2 — Save the Configuration

Save the router configuration:

```cisco
R1(config)# do write
```

Expected output:

```text
Building configuration...

[OK]
```

Alternatively:

```cisco
R1# copy running-config startup-config
```

---

# Step 3 — Verify the SNMP Configuration

Use the following command to verify the configured SNMP communities:

```cisco
R1# show running-config | include snmp
```

Expected output:

```text
snmp-server community Cisco1 RO
snmp-server community Cisco2 RW
```

This confirms that both SNMP communities have been configured successfully.

---

# Step 4 — Configure PC1

PC1 should be configured with an IP address in the same subnet as R1.

Example:

```text
IP Address:      192.168.1.1
Subnet Mask:     255.255.255.0
Default Gateway: 192.168.1.254
```

From PC1, verify connectivity to R1:

```text
ping 192.168.1.254
```

The ping should succeed before attempting SNMP operations.

---

# Step 5 — Open the MIB Browser

On PC1:

1. Open **PC1**.
2. Select the **Desktop** tab.
3. Open **MIB Browser**.
4. Set the target **Address** to:

```text
192.168.1.254
```

5. Select the appropriate SNMP operation.
6. Navigate through the available MIB tree to locate the required objects.

The MIB Browser allows PC1 to send SNMP requests to R1 and display the returned information.

---

# Step 6 — Retrieve the System Uptime

The first required SNMP `Get` operation is to determine how long R1 has been running.

The standard SNMP object is:

```text
sysUpTime
```

OID:

```text
1.3.6.1.2.1.1.3.0
```

### Procedure

In the MIB Browser:

1. Navigate to:

```text
iso
 └── org
     └── dod
         └── internet
             └── mgmt
                 └── mib-2
                     └── system
                         └── sysUpTime
```

2. Select `sysUpTime`.
3. Use the `Get` operation.
4. Click **GO**.
5. Examine the returned value.

The result represents the amount of time the SNMP agent has been running, normally expressed in **hundredths of a second**.

---

# Step 7 — Retrieve the Hostname

The hostname is stored in the `sysName` object.

OID:

```text
1.3.6.1.2.1.1.5.0
```

### Procedure

Navigate to:

```text
MIB Tree
 └── mib-2
     └── system
         └── sysName
```

Select `sysName` and perform an SNMP `Get`.

The result should display the hostname currently configured on R1.

For example:

```text
Name/OID: 1.3.6.1.2.1.1.5.0
Value:    R11
Type:     OctetString
```

The screenshot from this lab demonstrates an SNMP `Get` returning:

```text
R11
```

This confirms that the SNMP agent can successfully retrieve the router's configured hostname.

---

# Step 8 — Determine the Number of Interfaces

The standard MIB contains the `ifNumber` object, which reports the number of network interfaces known to the device.

OID:

```text
1.3.6.1.2.1.2.1.0
```

Navigate to:

```text
mib-2
 └── interfaces
     └── ifNumber
```

Perform an SNMP `Get` operation.

The returned value represents the number of interfaces reported by R1 through the SNMP interface MIB.

---

# Step 9 — Identify the Interfaces

The interface table contains information about individual interfaces.

The relevant MIB is:

```text
ifTable
```

OID:

```text
1.3.6.1.2.1.2.2
```

Useful objects include:

| Object | OID | Information |
|---|---|---|
| `ifNumber` | `1.3.6.1.2.1.2.1.0` | Number of interfaces |
| `ifTable` | `1.3.6.1.2.1.2.2` | Interface table |
| `ifIndex` | `1.3.6.1.2.1.2.2.1.1` | Interface index |
| `ifDescr` | `1.3.6.1.2.1.2.2.1.2` | Interface description |
| `ifType` | `1.3.6.1.2.1.2.2.1.3` | Interface type |
| `ifSpeed` | `1.3.6.1.2.1.2.2.1.5` | Interface speed |
| `ifAdminStatus` | `1.3.6.1.2.1.2.2.1.7` | Administrative status |
| `ifOperStatus` | `1.3.6.1.2.1.2.2.1.8` | Operational status |

The router configuration used in this lab shows these interfaces:

```text
GigabitEthernet0/0
GigabitEthernet0/1
GigabitEthernet0/2
Vlan1
```

`GigabitEthernet0/0` is the active interface connected to the LAN:

```text
interface GigabitEthernet0/0
 ip address 192.168.1.254 255.255.255.0
```

The other interfaces are currently administratively shut down in the configuration.

---

# Step 10 — Explore Additional SNMP Information

The MIB Browser can provide much more information than just the hostname and interface count.

Explore the following objects under the `system` MIB:

| Object | OID | Description |
|---|---|---|
| `sysDescr` | `1.3.6.1.2.1.1.1.0` | System/device description |
| `sysObjectID` | `1.3.6.1.2.1.1.2.0` | Vendor/device object identifier |
| `sysUpTime` | `1.3.6.1.2.1.1.3.0` | System uptime |
| `sysContact` | `1.3.6.1.2.1.1.4.0` | System contact |
| `sysName` | `1.3.6.1.2.1.1.5.0` | Device hostname |
| `sysLocation` | `1.3.6.1.2.1.1.6.0` | Device location |
| `sysServices` | `1.3.6.1.2.1.1.7.0` | Services provided by the system |

Also explore the interface MIB to investigate information such as:

- Interface descriptions
- Interface status
- Interface speed
- Interface type
- Interface indexes
- Input/output counters
- Packet statistics
- Error counters

This demonstrates one of the main advantages of SNMP: a network management system can remotely collect information about network devices without requiring an administrator to log directly into each device.

---

# Step 11 — Change the Hostname Using SNMP Set

The `sysName` object can be used to demonstrate an SNMP `Set` operation.

The OID is:

```text
1.3.6.1.2.1.1.5.0
```

Because changing an SNMP object requires write access, use the read/write community:

```text
Cisco2
```

### Procedure

On PC1's MIB Browser:

1. Enter R1's address:

```text
192.168.1.254
```

2. Locate:

```text
sysName
```

3. Select the `Set` operation.
4. Enter the new hostname.
5. Use the `Cisco2` read/write community.
6. Execute the operation.

For example, change the hostname from:

```text
R1
```

to:

```text
R11
```

After the `Set` operation, use another SNMP `Get` on `sysName` to verify the change.

The returned value should be:

```text
R11
```

---

# Step 12 — Verify the Hostname from Cisco IOS

After performing the SNMP `Set`, verify the hostname directly from the router CLI:

```cisco
R11# show running-config | include hostname
```

Expected output:

```text
hostname R11
```

You can also display the complete running configuration:

```cisco
R11# show running-config
```

The resulting configuration should contain:

```text
hostname R11

...

snmp-server community Cisco1 RO
snmp-server community Cisco2 RW
```

This provides confirmation that the SNMP `Set` operation successfully modified the router's hostname.

---

# Key SNMP Operations Used

| Operation | Purpose | Community |
|---|---|---|
| `Get` | Retrieve information from R1 | `Cisco1` |
| `Set` | Modify supported information on R1 | `Cisco2` |

The important distinction is:

```text
Cisco1 → Read Only
Cisco2 → Read/Write
```

Therefore, `Cisco1` can be used to retrieve information but should not be used for configuration changes.

---

# Verification Checklist

- [x] R1 has an SNMP read-only community configured.
- [x] R1 has an SNMP read/write community configured.
- [x] SNMP configuration was saved.
- [x] PC1 can reach R1 at `192.168.1.254`.
- [x] MIB Browser can communicate with R1.
- [x] `sysUpTime` was retrieved using an SNMP `Get`.
- [x] `sysName` was retrieved using an SNMP `Get`.
- [x] The number of interfaces was investigated.
- [x] Interface information was investigated through the interface MIB.
- [x] Additional SNMP system information was explored.
- [x] An SNMP `Set` was used to change the hostname.
- [x] The hostname change was verified from the Cisco IOS CLI.

---

# Important Commands

### Configure SNMP

```cisco
R1(config)# snmp-server community Cisco1 ro
R1(config)# snmp-server community Cisco2 rw
```

### Verify SNMP configuration

```cisco
R1# show running-config | include snmp
```

Expected:

```text
snmp-server community Cisco1 RO
snmp-server community Cisco2 RW
```

### Verify hostname

```cisco
R11# show running-config | include hostname
```

Expected:

```text
hostname R11
```

### Save configuration

```cisco
R1# write
```

---

# Key Takeaways

This lab demonstrates how SNMP can be used to remotely monitor and manage network devices.

The main concepts demonstrated are:

1. **SNMP communities** provide a basic mechanism for controlling access to an SNMP agent.
2. **Read-only communities** allow monitoring systems to retrieve information.
3. **Read/write communities** can modify supported SNMP objects.
4. The **MIB** organizes information about a network device into manageable objects.
5. SNMP `Get` operations can retrieve information such as system uptime, hostname, interface information, and device statistics.
6. SNMP `Set` operations can modify supported configuration objects.
7. The MIB Browser in Packet Tracer provides a practical introduction to SNMP, although its functionality is significantly more limited than a real Cisco environment.

---

## Lab Result

The SNMP configuration was successfully applied to R1:

```text
snmp-server community Cisco1 RO
snmp-server community Cisco2 RW
```

SNMP `Get` operations were used to retrieve system information, including the router hostname. An SNMP `Set` operation using the read/write community was then used to change the hostname to:

```text
R11
```

The change was subsequently verified from the Cisco IOS command line.