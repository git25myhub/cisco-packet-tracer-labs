# Basic QoS Configuration and DSCP Marking

## Lab Overview

In this lab, you will configure basic **Quality of Service (QoS)** policies on R1 to classify different types of traffic, mark packets with specific **DSCP values**, and allocate bandwidth to each traffic class.

The QoS policy will be applied **outbound on interface G0/0/0**.

> **Note:** QoS configuration is not a CCNA exam topic. This lab is intended to introduce basic QoS concepts and Cisco IOS QoS configuration.

---

## Lab Objectives

By completing this lab, you will learn how to:

- Create class maps to classify network traffic.
- Match HTTPS, HTTP, and ICMP traffic.
- Create a QoS policy map.
- Mark packets with DSCP values.
- Configure a priority queue.
- Allocate minimum bandwidth to traffic classes.
- Apply a QoS policy outbound on an interface.
- Use Packet Tracer Simulation Mode to inspect DSCP markings.

---

# QoS Requirements

Configure the following QoS settings on R1 and apply them outbound on **GigabitEthernet0/0/0**.

| Traffic Type | Classification | DSCP Marking | Bandwidth |
|---|---|---|---|
| HTTPS | HTTPS_MAP | AF31 | Priority Queue — 10% |
| HTTP | HTTP_MAP | AF32 | Minimum 10% |
| ICMP | ICMP_MAP | CS2 | Minimum 5% |

---

# Part 1: Verify Connectivity

Before configuring QoS, verify that PC1 can reach the web server.

From PC1, ping:

```text
jeremysitlab.com
```

Example:

```text
C:\> ping jeremysitlab.com
```

The hostname should resolve to:

```text
10.0.0.100
```

You should receive successful replies:

```text
Reply from 10.0.0.100: bytes=32 time<1ms TTL=126
```

You can also ping the server directly:

```text
C:\> ping 10.0.0.100
```

Successful connectivity confirms that the network is functioning correctly before QoS policies are applied.

---

# Part 2: Create QoS Class Maps

Class maps are used to classify traffic based on specific matching criteria.

In this lab, create three class maps:

- HTTPS traffic
- HTTP traffic
- ICMP traffic

## Configure the HTTPS Class Map

```cisco
R1> enable
R1# configure terminal

R1(config)# class-map match-all HTTPS_MAP
R1(config-cmap)# match protocol https
R1(config-cmap)# exit
```

This class map matches HTTPS traffic.

---

## Configure the HTTP Class Map

```cisco
R1(config)# class-map match-all HTTP_MAP
R1(config-cmap)# match protocol http
R1(config-cmap)# exit
```

This class map matches HTTP traffic.

---

## Configure the ICMP Class Map

```cisco
R1(config)# class-map match-all ICMP_MAP
R1(config-cmap)# match protocol icmp
R1(config-cmap)# exit
```

This class map matches ICMP traffic, including ping packets.

---

## Verify the Class Maps

Use the following command:

```cisco
R1# show running-config | section class-map
```

Expected configuration:

```cisco
class-map match-all HTTPS_MAP
 match protocol https

class-map match-all HTTP_MAP
 match protocol http

class-map match-all ICMP_MAP
 match protocol icmp
```

---

# Part 3: Create the QoS Policy Map

Next, create a policy map that defines how each traffic class should be treated.

The policy map will:

- Mark HTTPS traffic as AF31.
- Give HTTPS traffic a priority queue with 10% bandwidth.
- Mark HTTP traffic as AF32.
- Reserve 10% bandwidth for HTTP traffic.
- Mark ICMP traffic as CS2.
- Reserve 5% bandwidth for ICMP traffic.

---

## Configure HTTPS QoS

Create the policy map:

```cisco
R1(config)# policy-map G0/0/0_OUT
```

Enter the HTTPS class:

```cisco
R1(config-pmap)# class HTTPS_MAP
```

Mark HTTPS packets with DSCP AF31:

```cisco
R1(config-pmap-c)# set ip dscp af31
```

Configure HTTPS as a strict priority queue with 10% bandwidth:

```cisco
R1(config-pmap-c)# priority percent 10
```

Exit the class:

```cisco
R1(config-pmap-c)# exit
```

### Why Use `priority percent`?

The `priority` command creates a **Low Latency Queue (LLQ)**.

Traffic in this queue receives strict priority treatment, which can help reduce delay and jitter for important traffic.

In this lab, HTTPS traffic is allocated:

```text
10% of the available interface bandwidth
```

---

## Configure HTTP QoS

Enter the HTTP class:

```cisco
R1(config-pmap)# class HTTP_MAP
```

Mark HTTP packets with DSCP AF32:

```cisco
R1(config-pmap-c)# set ip dscp af32
```

Guarantee a minimum of 10% bandwidth:

```cisco
R1(config-pmap-c)# bandwidth percent 10
```

Exit the class:

```cisco
R1(config-pmap-c)# exit
```

Unlike the HTTPS class, HTTP traffic does not use the strict priority queue.

Instead, it receives a guaranteed minimum bandwidth allocation of 10%.

---

## Configure ICMP QoS

Enter the ICMP class:

```cisco
R1(config-pmap)# class ICMP_MAP
```

Mark ICMP packets with DSCP CS2:

```cisco
R1(config-pmap-c)# set ip dscp cs2
```

Guarantee a minimum of 5% bandwidth:

```cisco
R1(config-pmap-c)# bandwidth percent 5
```

Exit policy-map configuration:

```cisco
R1(config-pmap-c)# exit
R1(config-pmap)# exit
```

---

# Part 4: Verify the Policy Map

Verify the QoS configuration:

```cisco
R1# show running-config | section policy-map
```

Expected output:

```cisco
policy-map G0/0/0_OUT

 class HTTPS_MAP
  priority percent 10
  set ip dscp af31

 class HTTP_MAP
  bandwidth percent 10
  set ip dscp af32

 class ICMP_MAP
  bandwidth percent 5
  set ip dscp cs2
```

---

# Part 5: Apply the QoS Policy

The QoS policy must now be applied outbound on interface **GigabitEthernet0/0/0**.

```cisco
R1(config)# interface gigabitethernet0/0/0
R1(config-if)# service-policy output G0/0/0_OUT
R1(config-if)# end
```

The QoS policy is now active for traffic leaving G0/0/0.

---

# Part 6: Save the Configuration

Save the running configuration:

```cisco
R1# write memory
```

Or:

```cisco
R1# copy running-config startup-config
```

---

# Final R1 Configuration

## Class Maps

```cisco
class-map match-all HTTPS_MAP
 match protocol https

class-map match-all HTTP_MAP
 match protocol http

class-map match-all ICMP_MAP
 match protocol icmp
```

## Policy Map

```cisco
policy-map G0/0/0_OUT

 class HTTPS_MAP
  priority percent 10
  set ip dscp af31

 class HTTP_MAP
  bandwidth percent 10
  set ip dscp af32

 class ICMP_MAP
  bandwidth percent 5
  set ip dscp cs2
```

## Interface Configuration

```cisco
interface GigabitEthernet0/0/0
 ip address 172.16.0.1 255.255.255.252
 service-policy output G0/0/0_OUT
 duplex auto
 speed auto
```

---

# Part 7: Verify the QoS Policy

Use the following commands to verify the QoS configuration.

## View the Applied Policy

```cisco
R1# show policy-map interface gigabitethernet0/0/0
```

This command displays the policy applied to the interface and can show packet counters for each traffic class.

You should see the policy:

```text
G0/0/0_OUT
```

applied in the outbound direction.

---

## View the Running Configuration

```cisco
R1# show running-config
```

Or view only QoS-related sections:

```cisco
R1# show running-config | section class-map
R1# show running-config | section policy-map
```

---

# Part 8: Analyze DSCP Markings in Simulation Mode

Switch Cisco Packet Tracer to **Simulation Mode**.

Generate different types of traffic from PC1 and inspect the packets as they pass through R1.

The QoS policy should classify and mark the traffic as it leaves **G0/0/0**.

---

## Test 1: ICMP Traffic

From PC1:

```text
C:\> ping jeremysitlab.com
```

The hostname resolves to:

```text
10.0.0.100
```

The ICMP traffic should match:

```text
ICMP_MAP
```

R1 should mark the packets with:

```text
DSCP CS2
```

### Expected QoS Treatment

| Property | Value |
|---|---|
| Protocol | ICMP |
| Class Map | ICMP_MAP |
| DSCP | CS2 |
| Bandwidth | Minimum 5% |

In Simulation Mode, inspect the packet after it is processed by R1 to observe the DSCP marking.

---

## Test 2: HTTP Traffic

From PC1, open the web browser and access:

```text
http://jeremysitlab.com
```

The HTTP traffic should match:

```text
HTTP_MAP
```

R1 should mark the traffic with:

```text
DSCP AF32
```

### Expected QoS Treatment

| Property | Value |
|---|---|
| Protocol | HTTP |
| Class Map | HTTP_MAP |
| DSCP | AF32 |
| Bandwidth | Minimum 10% |

Inspect the packet in Simulation Mode after it passes through R1.

---

## Test 3: HTTPS Traffic

From PC1, access:

```text
https://jeremysitlab.com
```

The HTTPS traffic should match:

```text
HTTPS_MAP
```

R1 should mark the traffic with:

```text
DSCP AF31
```

### Expected QoS Treatment

| Property | Value |
|---|---|
| Protocol | HTTPS |
| Class Map | HTTPS_MAP |
| DSCP | AF31 |
| Bandwidth | Priority Queue — 10% |

HTTPS traffic receives strict priority treatment through the Low Latency Queue.

---

# Understanding the DSCP Values

## AF31

AF stands for **Assured Forwarding**.

```text
AF31
```

is used to mark HTTPS traffic in this lab.

The HTTPS traffic is also placed into a priority queue with:

```cisco
priority percent 10
```

---

## AF32

```text
AF32
```

is used to mark HTTP traffic.

HTTP traffic receives a minimum bandwidth allocation of:

```text
10%
```

using:

```cisco
bandwidth percent 10
```

---

## CS2

CS stands for **Class Selector**.

```text
CS2
```

is used to mark ICMP traffic.

ICMP receives a minimum bandwidth allocation of:

```text
5%
```

using:

```cisco
bandwidth percent 5
```

---

# QoS Configuration Summary

```text
                         R1
                          |
                          | G0/0/0
                          | Outbound QoS Policy
                          |
              -------------------------
              |           |           |
              ▼           ▼           ▼
           HTTPS        HTTP         ICMP
              |           |           |
              ▼           ▼           ▼
            AF31        AF32         CS2
              |           |           |
        Priority 10%   BW 10%      BW 5%
```

---

# Important QoS Commands

## Create a Class Map

```cisco
class-map match-all <NAME>
 match protocol <PROTOCOL>
```

Example:

```cisco
class-map match-all HTTPS_MAP
 match protocol https
```

---

## Create a Policy Map

```cisco
policy-map <NAME>
```

Example:

```cisco
policy-map G0/0/0_OUT
```

---

## Select a Traffic Class

```cisco
class <CLASS-MAP-NAME>
```

Example:

```cisco
class HTTPS_MAP
```

---

## Mark Packets with DSCP

```cisco
set ip dscp <DSCP-VALUE>
```

Examples:

```cisco
set ip dscp af31
set ip dscp af32
set ip dscp cs2
```

---

## Configure a Priority Queue

```cisco
priority percent <PERCENTAGE>
```

Example:

```cisco
priority percent 10
```

---

## Allocate Minimum Bandwidth

```cisco
bandwidth percent <PERCENTAGE>
```

Examples:

```cisco
bandwidth percent 10
bandwidth percent 5
```

---

## Apply a QoS Policy

```cisco
interface <INTERFACE>

service-policy output <POLICY-NAME>
```

Example:

```cisco
interface gigabitethernet0/0/0
 service-policy output G0/0/0_OUT
```

---

# Lab Questions

1. Which protocol is matched by `HTTPS_MAP`?

2. Which DSCP marking is assigned to HTTPS traffic?

3. What command creates a strict priority queue?

4. What percentage of bandwidth is allocated to the HTTPS priority queue?

5. Which DSCP marking is assigned to HTTP traffic?

6. What minimum bandwidth percentage is assigned to HTTP traffic?

7. Which protocol is matched by `ICMP_MAP`?

8. Which DSCP marking is assigned to ICMP traffic?

9. What minimum bandwidth percentage is assigned to ICMP traffic?

10. On which interface is the QoS policy applied?

11. Is the policy applied inbound or outbound?

12. Which command can be used to verify QoS statistics on the interface?

---

# Key Takeaways

- QoS allows network devices to classify and treat different types of traffic differently.
- **Class maps** identify specific traffic.
- **Policy maps** define the QoS actions applied to that traffic.
- HTTPS traffic is marked as **AF31** and receives a **10% priority queue**.
- HTTP traffic is marked as **AF32** and receives a **minimum 10% bandwidth allocation**.
- ICMP traffic is marked as **CS2** and receives a **minimum 5% bandwidth allocation**.
- `priority percent` creates a strict priority queue.
- `bandwidth percent` reserves minimum bandwidth for a traffic class.
- `service-policy output` applies the QoS policy to outbound traffic.
- Packet Tracer Simulation Mode can be used to observe how packets are classified and marked with DSCP values.
- The `show policy-map interface` command can be used to verify QoS policy activity and traffic statistics.