# Cisco Packet Tracer Lab — Syslog Configuration and Logging

## Overview

This lab demonstrates how to configure and verify **Syslog and Cisco IOS logging** using Cisco Packet Tracer.

The lab covers several logging destinations and concepts, including:

- Console logging
- VTY/monitor logging
- Buffered logging
- Syslog server logging
- Syslog severity levels
- Logging timestamps
- Remote logging to a Syslog server
- Verifying logging configuration with `show logging`

> **Important:** Cisco Packet Tracer has some limitations compared with real Cisco IOS devices. In particular, the `logging monitor` command may not be available in Packet Tracer. Monitor logging is enabled by default in the simulated environment, so `terminal monitor` can be used to display logging messages during the current VTY session.

---

# Lab Topology

The topology consists of:

```text
                         PC2
                          |
                          |
                         SW1
                        /   \
                       /     \
                     PC1     R1
                              |
                             G0/0
                              |
                         192.168.1.1

                         SRV1
                          |
                         SW1
```

### Addressing

| Device | Interface | IP Address |
|---|---|---|
| R1 | G0/0 | `192.168.1.1/24` |
| PC1 | NIC | `192.168.1.12/24` |
| PC2 | NIC | `192.168.1.13/24` |
| SRV1 | NIC | `192.168.1.100/24` |

SRV1 acts as the **Syslog server**.

---

# Device Credentials

The router is configured with the following credentials:

| Credential | Value |
|---|---|
| Username | `jeremy` |
| Password | `ccna` |
| Enable Password | `ccna` |

The local user is configured on R1 as:

```cisco
username jeremy password 0 ccna
```

The enable password is:

```cisco
enable password ccna
```

---

# Objectives

By the end of this lab, you should be able to:

- Access a Cisco router through the console.
- Generate Syslog messages by changing interface states.
- Identify Syslog severity levels.
- Enable timestamps for logging messages.
- Access the router remotely using Telnet.
- Understand why logging messages may not appear on a VTY session.
- Enable terminal monitoring for the current VTY session.
- Configure buffered logging.
- Increase the logging buffer size to 8192 bytes.
- Configure R1 to send Syslog messages to SRV1.
- Configure the Syslog server to receive debugging-level messages.
- Verify the complete logging configuration.

---

# Syslog Severity Levels

Cisco Syslog messages contain a severity level from **0 through 7**.

| Level | Name | Description |
|---:|---|---|
| 0 | Emergencies | System unusable |
| 1 | Alerts | Immediate action required |
| 2 | Critical | Critical conditions |
| 3 | Errors | Error conditions |
| 4 | Warnings | Warning conditions |
| 5 | Notifications | Normal but significant conditions |
| 6 | Informational | Informational messages |
| 7 | Debugging | Debugging messages |

The lower the number, the **more severe** the message.

For example:

```text
%LINK-3-UPDOWN
```

has severity level **3 — Error**.

And:

```text
%LINK-5-CHANGED
```

has severity level **5 — Notification**.

---

# Step 1 — Connect to R1 Using the Console

Connect **PC2** to the console port of R1.

Open:

```text
PC2 → Desktop → Terminal
```

Accept the default terminal settings and press **Enter**.

You should eventually receive the router login prompt:

```text
User Access Verification

Username:
```

Enter:

```text
Username: jeremy
Password: ccna
```

Then enter privileged EXEC mode:

```cisco
R1> enable
Password: ccna

R1#
```

---

# Step 2 — Shut Down G0/0

Enter global configuration mode:

```cisco
R1# configure terminal
```

Enter the G0/0 interface:

```cisco
R1(config)# interface gigabitEthernet 0/0
```

Shut down the interface:

```cisco
R1(config-if)# shutdown
```

R1 should generate Syslog messages similar to:

```text
%LINK-5-CHANGED: Interface GigabitEthernet0/0, changed state to administratively down

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0, changed state to down
```

These messages are displayed through the console because console logging is enabled.

---

# Step 3 — Identify the Severity Levels

Examine the messages generated when G0/0 is shut down.

### Message 1

```text
%LINK-5-CHANGED
```

The number `5` represents:

```text
Severity 5 — Notifications
```

### Message 2

```text
%LINEPROTO-5-UPDOWN
```

The number `5` also represents:

```text
Severity 5 — Notifications
```

Therefore, the interface shutdown messages generated in this exercise are **severity level 5 (Notifications)**.

Other messages generated during the lab may have different severity levels. For example:

```text
%LINK-3-UPDOWN
```

is a **severity 3 (Errors)** message.

---

# Step 4 — Re-enable G0/0

After observing the Syslog messages, restore the interface:

```cisco
R1(config-if)# no shutdown
```

R1 should generate messages indicating that the interface and line protocol have returned to the up state:

```text
%LINK-5-CHANGED: Interface GigabitEthernet0/0, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0, changed state to up
```

---

# Step 5 — Enable Logging Timestamps

By default, logging messages may not include useful date/time information.

Enable timestamps with:

```cisco
R1(config)# service timestamps log datetime msec
```

This configures Cisco IOS to include:

- Date
- Time
- Milliseconds

in Syslog messages.

After enabling timestamps, messages will look similar to:

```text
*Feb 28, 18:04:22.044: SYS-5-CONFIG_I: Configured from console by console
```

The timestamp makes it much easier to determine **when an event occurred**, which is particularly important when troubleshooting network incidents.

---

# Step 6 — Save the Configuration

Save the configuration:

```cisco
R1(config)# end
R1# write
```

Expected output:

```text
Building configuration...

[OK]
```

---

# Step 7 — Telnet from PC1 to R1

From PC1, open:

```text
PC1 → Desktop → Command Prompt
```

Connect to R1's G0/0 address:

```text
C:\> telnet 192.168.1.1
```

Expected output:

```text
Trying 192.168.1.1 ...Open

User Access Verification

Username: jeremy
Password:
```

Enter privileged EXEC mode:

```cisco
R1> enable
Password:

R1#
```

---

# Step 8 — Enable the Unused G0/1 Interface

From the Telnet session, enter:

```cisco
R1# configure terminal
R1(config)# interface gigabitEthernet 0/1
R1(config-if)# no shutdown
```

The router may generate a message such as:

```text
%LINK-3-UPDOWN: Interface GigabitEthernet0/1, changed state to down
```

Depending on the state of the interface and Packet Tracer simulation, you may not immediately see the message on the Telnet session.

---

# Why Does No Syslog Message Appear?

The reason is that **console logging and VTY/monitor logging are separate logging destinations**.

When you connect through the console, Syslog messages are displayed directly on the console.

A Telnet session is a **VTY session**, and logging messages are not automatically displayed on the VTY line in the same way.

Therefore, a message can be generated and stored by the router without appearing in the current Telnet session.

---

# Step 9 — Enable Logging to the VTY Session

Cisco IOS normally uses the following command to enable logging messages on a terminal session:

```cisco
R1# terminal monitor
```

In this Packet Tracer lab, the `logging monitor` configuration command is not available. However, monitor logging is enabled by default.

Therefore, use:

```cisco
R1(config-if)# do terminal monitor
```

or from privileged EXEC mode:

```cisco
R1# terminal monitor
```

Now generate another logging event.

For example:

```cisco
R1(config)# interface gigabitEthernet 0/1
R1(config-if)# shutdown
```

You should now see the Syslog message in the current Telnet session.

Example:

```text
*Feb 28, 18:06:59.066: %LINK-5-CHANGED: Interface GigabitEthernet0/1, changed state to administratively down
```

This demonstrates the difference between the console and VTY logging destinations.

---

# Step 10 — Verify Logging Status

Use:

```cisco
R1# show logging
```

The output provides information about the configured logging destinations.

The lab output shows:

```text
Syslog logging: enabled
```

and:

```text
Console logging: level debugging
```

It also shows:

```text
Monitor logging: level debugging
```

This confirms that monitor logging is available for the current terminal session.

---

# Step 11 — Configure Buffered Logging

Buffered logging allows the router to store Syslog messages in memory.

This is useful because messages can be reviewed later even if they were not visible on the console or terminal at the time they occurred.

Configure an 8192-byte buffer:

```cisco
R1(config)# logging buffered 8192
```

The router should generate a configuration message similar to:

```text
SYS-5-LOG_CONFIG_CHANGE: Buffer logging: level debugging, xml disabled, filtering disabled, size (8192)
```

---

# Step 12 — Verify the Logging Buffer

Use:

```cisco
R1# show logging
```

Look for:

```text
Buffer logging: level debugging
```

and:

```text
Log Buffer (8192 bytes):
```

This confirms that the logging buffer has been increased from the default size to **8192 bytes**.

---

# Step 13 — Configure SRV1 as the Syslog Server

SRV1 has the IP address:

```text
192.168.1.100
```

Configure R1 to send Syslog messages to SRV1:

```cisco
R1(config)# logging host 192.168.1.100
```

This tells R1 to forward its Syslog messages to the remote Syslog server.

The router should generate a message similar to:

```text
%SYS-6-LOGGINGHOST_STARTSTOP: Logging to host 192.168.1.100 port 514 started - CLI initiated
```

The default Syslog port is:

```text
UDP 514
```

---

# Step 14 — Configure the Logging Trap Level

The `logging trap` command controls which Syslog messages are sent to the remote Syslog server.

Configure the level as **debugging**:

```cisco
R1(config)# logging trap debugging
```

Because debugging is severity level **7**, configuring:

```text
debugging
```

allows messages from severity levels **0 through 7** to be sent to the remote Syslog server.

In other words:

```text
0 Emergencies
1 Alerts
2 Critical
3 Errors
4 Warnings
5 Notifications
6 Informational
7 Debugging
```

All levels are included when the trap level is set to `debugging`.

---

# Step 15 — Configure and Verify SRV1

Open SRV1 and navigate to:

```text
Services → SYSLOG
```

Make sure the Syslog service is:

```text
On
```

The server should begin displaying messages received from R1.

The Packet Tracer Syslog server in this lab displays information such as:

| Time | HostName | Message |
|---|---|---|
| Timestamp | `192.168.1.1` | Interface/status message |
| Timestamp | `192.168.1.1` | Configuration message |
| Timestamp | `192.168.1.1` | Syslog event |

The screenshot from the lab shows SRV1 receiving messages from:

```text
192.168.1.1
```

This confirms that R1 is successfully communicating with the Syslog server.

---

# Step 16 — Generate Test Syslog Messages

To confirm that the complete logging configuration works, generate interface events.

For example:

```cisco
R1# configure terminal
R1(config)# interface gigabitEthernet 0/1
R1(config-if)# no shutdown
```

Then:

```cisco
R1(config-if)# shutdown
```

The router should generate messages similar to:

```text
%LINK-3-UPDOWN: Interface GigabitEthernet0/1, changed state to down

%LINK-5-CHANGED: Interface GigabitEthernet0/1, changed state to administratively down
```

These messages should be visible through the configured logging destinations and forwarded to SRV1.

---

# Final Configuration

The completed R1 configuration contains the important logging commands:

```cisco
service timestamps log datetime msec
!
logging trap debugging
logging 192.168.1.100
```

The relevant interface configuration is:

```cisco
interface GigabitEthernet0/0
 ip address 192.168.1.1 255.255.255.0
 duplex auto
 speed auto

interface GigabitEthernet0/1
 no ip address
 duplex auto
 speed auto
 shutdown
```

The VTY configuration allows Telnet access:

```cisco
line vty 0 4
 login local
 transport input telnet

line vty 5 15
 login local
 transport input telnet
```

The local authentication configuration is:

```cisco
username jeremy password 0 ccna
```

---

# Verification Commands

## Verify Logging Configuration

```cisco
R1# show logging
```

Check for:

```text
Syslog logging: enabled
Console logging: level debugging
Monitor logging: level debugging
Buffer logging: level debugging
```

and:

```text
Log Buffer (8192 bytes)
```

---

## Verify Running Configuration

```cisco
R1# show running-config
```

Important commands should appear as:

```text
service timestamps log datetime msec

logging trap debugging
logging 192.168.1.100
```

---

## Verify Interface Status

```cisco
R1# show ip interface brief
```

Example:

```text
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     192.168.1.1     YES manual up                    up
GigabitEthernet0/1     unassigned      YES unset  administratively down down
GigabitEthernet0/2     unassigned      YES unset  administratively down down
Vlan1                  unassigned      YES unset  administratively down down
```

---

# Important Commands Summary

### Enable timestamps

```cisco
R1(config)# service timestamps log datetime msec
```

### Enable buffered logging

```cisco
R1(config)# logging buffered 8192
```

### Configure remote Syslog server

```cisco
R1(config)# logging host 192.168.1.100
```

### Send all severity levels to the Syslog server

```cisco
R1(config)# logging trap debugging
```

### Display logging information

```cisco
R1# show logging
```

### Display logging messages on the current VTY session

```cisco
R1# terminal monitor
```

### Save configuration

```cisco
R1# write
```

---

# Lab Verification Checklist

- [x] Connected to R1 through the console using PC2.
- [x] Authenticated using the local username `jeremy`.
- [x] Entered privileged EXEC mode.
- [x] Shut down G0/0.
- [x] Observed Syslog messages generated by the interface state change.
- [x] Identified Syslog severity levels.
- [x] Re-enabled G0/0.
- [x] Enabled logging timestamps with millisecond precision.
- [x] Connected to R1 through Telnet from PC1.
- [x] Enabled G0/1.
- [x] Investigated why logging messages did not initially appear on the VTY session.
- [x] Enabled terminal monitoring using `terminal monitor`.
- [x] Verified logging with `show logging`.
- [x] Configured an 8192-byte logging buffer.
- [x] Configured SRV1 (`192.168.1.100`) as the remote Syslog server.
- [x] Configured the remote logging level as `debugging`.
- [x] Verified Syslog messages on SRV1.
- [x] Saved the final router configuration.

---

# Key Takeaways

This lab demonstrates that Cisco IOS can send logging information to several different destinations.

```text
                    R1
                    |
        +-----------+-----------+
        |           |           |
     Console      VTY       Buffer
        |           |           |
       PC2         PC1       R1 Memory
                                |
                                |
                          Syslog Server
                             SRV1
                         192.168.1.100
```

The major concepts demonstrated are:

1. **Console logging** displays messages directly through the router console.
2. **Monitor logging** allows messages to appear in an active VTY session when `terminal monitor` is enabled.
3. **Buffered logging** stores messages in memory for later inspection.
4. **Remote Syslog logging** sends messages to an external Syslog server.
5. **Severity levels** determine the importance of Syslog messages and which messages are forwarded.
6. **Timestamps** make it possible to determine exactly when events occurred.
7. Setting the remote logging level to **debugging** allows Syslog messages from severity `0` through `7` to be forwarded to the configured Syslog server.

The final configuration successfully sends R1's logging information to **SRV1 at `192.168.1.100`**, with the remote logging level set to **debugging**.