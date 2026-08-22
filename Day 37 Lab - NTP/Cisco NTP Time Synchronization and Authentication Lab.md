# Cisco NTP Time Synchronization and Authentication Lab

## Overview

This lab focuses on configuring **Network Time Protocol (NTP)** across a three-router topology.

The routing infrastructure has already been configured, allowing the routers to communicate with one another and reach the Internet through R1.

The lab demonstrates how to:

- Configure the software clock.
- Configure the local time zone.
- Synchronize R1 with an external NTP server.
- Configure R1 as an NTP master.
- Synchronize R2 and R3 to R1.
- Secure NTP communication using authentication.
- Update the hardware calendar using NTP.

> **Packet Tracer limitation:** The `ntp source` command is unavailable in Packet Tracer, so the physical interface IP addresses of R1 are used as the NTP server addresses for R2 and R3. Packet Tracer also does not provide a way to view the hardware calendar.

---

# 1. Lab Objectives

By completing this lab, you will:

1. Configure R1, R2, and R3 with the required initial software clock.
2. Configure the time zone on all routers.
3. Configure R1 to synchronize with Internet NTP server `1.1.1.1`.
4. Determine the stratum of `1.1.1.1` and R1.
5. Configure R1 as a **stratum 8 NTP master**.
6. Configure R2 and R3 to synchronize with R1.
7. Configure NTP authentication between R1, R2, and R3.
8. Configure NTP to update the hardware calendar.
9. Verify NTP synchronization and clock status.

---

# 2. Preconfigured Routing

Routing has already been configured in the topology.

The routers use OSPF with:

```cisco
router ospf 1
 network 0.0.0.0 255.255.255.255 area 0
```

R1 also has a default route toward the Internet:

```cisco
ip route 0.0.0.0 0.0.0.0 203.0.113.2
```

Because routing is preconfigured, no routing changes are required for this lab.

---

# 3. Network Addressing

The router-to-router connections are:

| Connection | Interface | IP Address |
|---|---|---|
| R1 ↔ Internet | R1 G0/0 | 203.0.113.1/30 |
| R1 ↔ R2 | R1 G0/1 | 192.168.12.1/30 |
| R1 ↔ R3 | R1 G0/2 | 192.168.13.1/30 |
| R2 ↔ R1 | R2 G0/0 | 192.168.12.2/30 |
| R2 ↔ R3 | R2 G0/1 | 192.168.23.1/30 |
| R3 ↔ R1 | R3 G0/0 | 192.168.13.2/30 |
| R3 ↔ R2 | R3 G0/1 | 192.168.23.2/30 |

For this lab:

```text
External NTP Server
       1.1.1.1
          |
       Internet
          |
      R1 G0/0
  203.0.113.1
          |
    +-----+-----+
    |           |
  R2            R3
```

R1 acts as the central NTP master for the internal routers after synchronizing with the external NTP source.

---

# 4. Configure the Initial Software Clock

The required initial time is:

```text
12:00:00 Dec 30 2020 UTC
```

The Cisco IOS clock can be configured with:

```cisco
clock set 12:00:00 30 Dec 2020
```

Perform this on R1, R2, and R3.

### R1

```cisco
R1#clock set 12:00:00 30 Dec 2020
```

### R2

```cisco
R2#clock set 12:00:00 30 Dec 2020
```

### R3

```cisco
R3#clock set 12:00:00 30 Dec 2020
```

Verify the software clock:

```cisco
show clock
```

For additional information:

```cisco
show clock detail
```

---

# 5. Configure the Time Zone

The lab requires the routers to use the local time zone.

For this configuration, the routers use:

```text
GMT +3
```

The IOS command is:

```cisco
clock timezone GMT 3
```

Configure it on all three routers.

### R1

```cisco
R1(config)#clock timezone GMT 3
```

### R2

```cisco
R2(config)#clock timezone GMT 3
```

### R3

```cisco
R3(config)#clock timezone GMT 3
```

Verify:

```cisco
show clock detail
```

The output should identify the configured time zone.

---

# 6. Configure R1 to Use External NTP

R1 must synchronize with the external NTP server:

```text
1.1.1.1
```

Configure:

```cisco
R1(config)#ntp server 1.1.1.1
```

Save the configuration:

```cisco
R1(config)#do write
```

---

# 7. Determine the NTP Stratum

After configuring R1 to use `1.1.1.1`, use:

```cisco
show ntp associations
```

Example:

```text
R1#show ntp associations
```

You can also use:

```cisco
show ntp status
```

The NTP association table contains the **stratum** value.

### Understanding Stratum

NTP uses a hierarchy called **stratum**:

```text
Stratum 0
   |
   v
Stratum 1
   |
   v
Stratum 2
   |
   v
Stratum 3
   |
   ...
```

A lower stratum number generally represents a source closer to a reference clock.

For this lab, determine the values from Packet Tracer using:

```cisco
show ntp associations
show ntp status
```

The important values to record are:

| Device | NTP Source | Stratum |
|---|---|---:|
| 1.1.1.1 | External NTP server | Check with `show ntp associations` |
| R1 | 1.1.1.1 | Check with `show ntp status` |

> Do not assume the stratum values without checking the Packet Tracer NTP implementation.

---

# 8. Configure R1 as a Stratum 8 NTP Master

After configuring R1 to synchronize with the external NTP server, R1 is also configured as the internal NTP master.

Use:

```cisco
R1(config)#ntp master 8
```

The `8` specifies the stratum at which R1 operates as an NTP master.

Verify:

```cisco
show ntp status
```

and:

```cisco
show ntp associations
```

The configuration should contain:

```text
ntp master 8
```

---

# 9. Configure NTP Authentication

NTP authentication prevents unauthorized devices from synchronizing with the NTP server.

A shared authentication key is configured on R1, R2, and R3.

The lab configuration uses key ID:

```text
1
```

and an MD5 authentication key.

## R1

Configure:

```cisco
R1(config)#ntp authentication-key 1 md5 <KEY>
R1(config)#ntp authenticate
R1(config)#ntp trusted-key 1
```

Then configure the NTP server:

```cisco
R1(config)#ntp server 1.1.1.1
```

---

# 10. Configure R2 as an NTP Client

R2 should synchronize with R1 using R1's physical interface address.

Because Packet Tracer does not support the `ntp source` command, use:

```text
192.168.12.1
```

Configure the authentication key:

```cisco
R2(config)#ntp authentication-key 1 md5 <KEY>
R2(config)#ntp authenticate
R2(config)#ntp trusted-key 1
```

Configure R1 as the NTP server:

```cisco
R2(config)#ntp server 192.168.12.1 key 1
```

Enable hardware calendar updates:

```cisco
R2(config)#ntp update-calendar
```

---

# 11. Configure R3 as an NTP Client

R3 synchronizes with R1 through R1's GigabitEthernet0/2 interface.

R1's address is:

```text
192.168.13.1
```

Configure the authentication key:

```cisco
R3(config)#ntp authentication-key 1 md5 <KEY>
R3(config)#ntp authenticate
R3(config)#ntp trusted-key 1
```

Configure R1 as the NTP server:

```cisco
R3(config)#ntp server 192.168.13.1 key 1
```

Enable hardware calendar updates:

```cisco
R3(config)#ntp update-calendar
```

---

# 12. Configure NTP Hardware Calendar Updates

NTP can periodically update the router's hardware calendar.

The command is:

```cisco
ntp update-calendar
```

Configure this on all three routers.

### R1

```cisco
R1(config)#ntp update-calendar
```

### R2

```cisco
R2(config)#ntp update-calendar
```

### R3

```cisco
R3(config)#ntp update-calendar
```

Packet Tracer does not allow the hardware calendar to be viewed, but the configuration can be verified using:

```cisco
show running-config
```

Look for:

```text
ntp update-calendar
```

---

# 13. Complete NTP Configuration

## R1

The relevant R1 configuration should resemble:

```cisco
clock timezone GMT 3

ntp authentication-key 1 md5 <KEY>
ntp authenticate
ntp trusted-key 1
ntp server 1.1.1.1
ntp master 8
ntp update-calendar
```

---

## R2

The relevant R2 configuration should resemble:

```cisco
clock timezone GMT 3

ntp authentication-key 1 md5 <KEY>
ntp authenticate
ntp trusted-key 1
ntp server 192.168.12.1 key 1
ntp update-calendar
```

---

## R3

The relevant R3 configuration should resemble:

```cisco
clock timezone GMT 3

ntp authentication-key 1 md5 <KEY>
ntp authenticate
ntp trusted-key 1
ntp server 192.168.13.1 key 1
ntp update-calendar
```

---

# 14. Verify NTP Associations

Use:

```cisco
show ntp associations
```

on R1, R2, and R3.

For R3, for example:

```text
R3#show ntp associations
```

A successfully synchronized association should eventually show an indicator such as:

```text
*
```

The `*` indicates the current system peer.

An entry showing:

```text
st 16
reach 0
```

indicates that the router has **not yet successfully synchronized** with the configured NTP server.

Therefore, after configuration, allow some time for NTP synchronization and check the association again.

---

# 15. Verify NTP Status

Use:

```cisco
show ntp status
```

This provides information about:

- NTP synchronization status
- Reference clock
- Stratum
- Clock precision
- Root delay
- Root dispersion
- Reference time

A successfully synchronized router should report that the clock is synchronized.

---

# 16. Verify the Clock

Use:

```cisco
show clock
```

For more information:

```cisco
show clock detail
```

For example:

```text
R3#show clock detail
```

The output should identify NTP as the time source after synchronization:

```text
Time source is NTP
```

This confirms that the router's software clock is being synchronized through NTP.

---

# 17. Verify the Running Configuration

Use:

```cisco
show running-config
```

Check that the configuration contains the appropriate NTP commands.

For R3, the completed configuration includes:

```text
ntp authentication-key 1 md5 ...
ntp authenticate
ntp trusted-key 1
ntp server 192.168.13.1 key 1
ntp update-calendar
```

For R2:

```text
ntp authentication-key 1 md5 ...
ntp authenticate
ntp trusted-key 1
ntp server 192.168.12.1 key 1
ntp update-calendar
```

For R1:

```text
ntp authentication-key 1 md5 ...
ntp authenticate
ntp trusted-key 1
ntp server 1.1.1.1
ntp master 8
ntp update-calendar
```

---

# 18. Verification Command Reference

| Purpose | Command |
|---|---|
| Display current time | `show clock` |
| Display detailed clock information | `show clock detail` |
| Display NTP associations | `show ntp associations` |
| Display NTP status | `show ntp status` |
| View NTP configuration | `show running-config` |
| Configure time zone | `clock timezone GMT 3` |
| Configure NTP server | `ntp server <IP>` |
| Configure NTP authentication | `ntp authenticate` |
| Configure trusted key | `ntp trusted-key 1` |
| Configure authentication key | `ntp authentication-key 1 md5 <KEY>` |
| Configure NTP master | `ntp master 8` |
| Update hardware calendar | `ntp update-calendar` |
| Save configuration | `write memory` |

---

# 19. NTP Troubleshooting

If R2 or R3 does not synchronize, check the following.

### Check connectivity

From R2:

```cisco
ping 192.168.12.1
```

From R3:

```cisco
ping 192.168.13.1
```

The ping must succeed before NTP synchronization can occur.

### Check the NTP association

```cisco
show ntp associations
```

Look for:

```text
reach 0
```

A reach value of `0` indicates that NTP communication has not successfully occurred.

### Check NTP status

```cisco
show ntp status
```

### Check authentication

Make sure the following match between the NTP client and server:

```text
Key ID
Authentication algorithm
Authentication key
Trusted key
```

For R2/R3, make sure the server statement includes:

```cisco
ntp server <R1-IP> key 1
```

---

# 20. Lab Results

The completed lab demonstrates a hierarchical NTP design:

```text
                 Internet NTP
                   1.1.1.1
                      |
                      v
                +-----------+
                |    R1     |
                | NTP Master|
                | Stratum 8 |
                +-----------+
                   /     \
                  /       \
                 v         v
             +-----+     +-----+
             | R2  |     | R3  |
             | NTP |     | NTP |
             |Client|    |Client|
             +-----+     +-----+
```

R1 synchronizes with the external NTP source and provides NTP service to the internal routers.

R2 and R3 authenticate their NTP communication with R1 using the shared authentication key.

---

# 21. Final Configuration Checklist

- [ ] Set R1 software clock to `12:00:00 Dec 30 2020`.
- [ ] Set R2 software clock to `12:00:00 Dec 30 2020`.
- [ ] Set R3 software clock to `12:00:00 Dec 30 2020`.
- [ ] Configure the local time zone on R1.
- [ ] Configure the local time zone on R2.
- [ ] Configure the local time zone on R3.
- [ ] Configure R1 to use `1.1.1.1` as an NTP server.
- [ ] Determine the stratum of `1.1.1.1`.
- [ ] Determine the stratum of R1.
- [ ] Configure R1 as an NTP master with stratum 8.
- [ ] Configure NTP authentication on R1.
- [ ] Configure R2 to synchronize with `192.168.12.1`.
- [ ] Configure R3 to synchronize with `192.168.13.1`.
- [ ] Configure NTP authentication on R2 and R3.
- [ ] Configure R1, R2, and R3 to update their hardware calendars.
- [ ] Verify NTP associations.
- [ ] Verify NTP status.
- [ ] Verify the clock source.
- [ ] Save the configurations.

---

# Conclusion

This lab demonstrates how NTP can be deployed in a routed network to maintain consistent time across network devices.

R1 is configured to obtain time from the external NTP server `1.1.1.1` and then operates as a **stratum 8 NTP master** for the internal network. R2 and R3 synchronize with R1 using authenticated NTP sessions.

The final configuration provides centralized time synchronization, authentication, and automatic calendar updates across the three-router topology.