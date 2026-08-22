# Cisco Packet Tracer Lab: DHCP Server and DHCP Relay Configuration

## Lab Objectives

1. Configure three DHCP pools on R2:
   - **POOL1:** `192.168.1.0/24`
   - **POOL2:** `192.168.2.0/24`
   - **POOL3:** `203.0.113.0/30`
2. Reserve the required addresses from each DHCP pool.
3. Configure DNS and domain information for the DHCP clients.
4. Configure R1's G0/0 interface as a DHCP client.
5. Determine the IP address dynamically assigned to R1.
6. Configure R1 as a DHCP relay agent for the `192.168.1.0/24` network.
7. Use the PC CLI to request DHCP addresses.
8. Verify the DHCP configuration using `ipconfig /all`.

---

## Network Addressing

| Device | Interface | IP Address | Subnet Mask | Role |
|---|---|---|---|---|
| R1 | G0/0 | DHCP | `255.255.255.252` | DHCP Client |
| R1 | G0/1 | `192.168.1.1` | `255.255.255.0` | LAN Gateway |
| R2 | G0/0 | `203.0.113.1` | `255.255.255.252` | DHCP Server |
| R2 | G0/1 | `192.168.2.1` | `255.255.255.0` | LAN Gateway |
| PC1 | — | DHCP | `255.255.255.0` | DHCP Client |
| PC2 | — | DHCP | `255.255.255.0` | DHCP Client |

---

# 1. Configure DHCP Pools on R2

R2 was configured as the DHCP server for all three networks.

## Reserve Addresses

The first ten addresses in the `192.168.1.0/24` and `192.168.2.0/24` networks were excluded from DHCP allocation.

For POOL3, `.1` was reserved.

```text id="r2excluded"
R2(config)#ip dhcp excluded-address 192.168.1.1 192.168.1.10
R2(config)#ip dhcp excluded-address 192.168.2.1 192.168.2.10
R2(config)#ip dhcp excluded-address 203.0.113.1
```

This means DHCP will begin allocating addresses from:

- POOL1 → `192.168.1.11`
- POOL2 → `192.168.2.11`
- POOL3 → `203.0.113.2`

---

## POOL1 Configuration

POOL1 provides DHCP addresses for the `192.168.1.0/24` network.

```text id="pool1"
R2(config)#ip dhcp pool POOL1
R2(dhcp-config)#network 192.168.1.0 255.255.255.0
R2(dhcp-config)#default-router 192.168.1.1
R2(dhcp-config)#dns-server 8.8.8.8
R2(dhcp-config)#domain-name karisitlab.com
```

### POOL1 Parameters

| Parameter | Value |
|---|---|
| Network | `192.168.1.0/24` |
| Reserved | `192.168.1.1 - 192.168.1.10` |
| DHCP range begins | `192.168.1.11` |
| Default Gateway | `192.168.1.1` |
| DNS | `8.8.8.8` |
| Domain | `karisitlab.com` |

---

## POOL2 Configuration

POOL2 provides DHCP addresses for the `192.168.2.0/24` network.

```text id="pool2"
R2(config)#ip dhcp pool POOL2
R2(dhcp-config)#network 192.168.2.0 255.255.255.0
R2(dhcp-config)#default-router 192.168.2.1
R2(dhcp-config)#dns-server 8.8.8.8
R2(dhcp-config)#domain-name kiarisitlab.com
```

### POOL2 Parameters

| Parameter | Value |
|---|---|
| Network | `192.168.2.0/24` |
| Reserved | `192.168.2.1 - 192.168.2.10` |
| DHCP range begins | `192.168.2.11` |
| Default Gateway | `192.168.2.1` |
| DNS | `8.8.8.8` |
| Domain | `kiarisitlab.com` |

> **Note:** The original lab instructions specify `jeremysitlab.com` as the domain. The configuration captured from the lab uses `karisitlab.com` for POOL1 and `kiarisitlab.com` for POOL2. This README documents the configuration that was actually implemented.

---

## POOL3 Configuration

POOL3 was configured for the point-to-point `203.0.113.0/30` network.

```text id="pool3"
R2(config)#ip dhcp pool POOL3
R2(dhcp-config)#network 203.0.113.0 255.255.255.252
```

The `.1` address was reserved:

```text id="pool3exclude"
R2(config)#ip dhcp excluded-address 203.0.113.1
```

Therefore, the available DHCP address in this `/30` network is:

```text id="available203"
203.0.113.2
```

This is the address that R1 ultimately received on G0/0.

---

# 2. Configure R1 G0/0 as a DHCP Client

R1's G0/0 interface was configured to obtain its IP address dynamically from the DHCP server.

```text id="r1dhcp"
R1(config)#interface GigabitEthernet0/0
R1(config-if)#ip address dhcp
R1(config-if)#no shutdown
```

After enabling the interface, R1 received a DHCP address:

```text id="r1assigned"
%DHCP-6-ADDRESS_ASSIGN: Interface GigabitEthernet0/0 assigned DHCP address 203.0.113.2, mask 255.255.255.252, hostname R1
```

### Answer

**R1 G0/0 configured IP address:**

```text id="r1answer"
203.0.113.2/30
```

The DHCP server is R2 at:

```text id="r2server"
203.0.113.1
```

---

# 3. Configure R1 as a DHCP Relay Agent

DHCP clients on the `192.168.1.0/24` network cannot directly communicate with a DHCP server located on another subnet using normal DHCP broadcasts.

R1 therefore acts as a **DHCP relay agent**.

The relay address points to R2's DHCP server interface:

```text id="relay"
R1(config)#interface GigabitEthernet0/1
R1(config-if)#ip helper-address 203.0.113.1
```

R1's G0/1 interface is configured as:

```text id="r1lan"
interface GigabitEthernet0/1
 ip address 192.168.1.1 255.255.255.0
 ip helper-address 203.0.113.1
```

### DHCP Relay Operation

The DHCP request follows this path:

```text id="dhcpflow"
PC1
 |
 | DHCP Broadcast
 v
R1 G0/1
192.168.1.1
 |
 | DHCP Relay
 | ip helper-address 203.0.113.1
 v
R2
203.0.113.1
 |
 | DHCP Offer
 v
R1
 |
 v
PC1
```

The `ip helper-address` command allows R1 to forward DHCP requests from the local LAN to R2.

---

# 4. Configure Routing Between R1 and R2

Because the two LAN networks are located behind different routers, routes were configured.

On R1:

```text id="r1route"
ip route 192.168.2.0 255.255.255.0 GigabitEthernet0/0
```

On R2:

```text id="r2route"
ip route 192.168.1.0 255.255.255.0 GigabitEthernet0/0
```

These routes allow traffic between:

```text
192.168.1.0/24
        |
       R1
        |
203.0.113.0/30
        |
       R2
        |
192.168.2.0/24
```

---

# 5. Verify R2 DHCP Configuration

The DHCP configuration was verified using:

```text id="verifydhcp"
R2(dhcp-config)#do show running-config | section dhcp
```

The resulting configuration included:

```text id="dhcpconfig"
ip dhcp excluded-address 192.168.1.1 192.168.1.10
ip dhcp excluded-address 192.168.2.1 192.168.2.10
ip dhcp excluded-address 203.0.113.1

ip dhcp pool POOL1
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.1
 dns-server 8.8.8.8
 domain-name karisitlab.com

ip dhcp pool POOL2
 network 192.168.2.0 255.255.255.0
 default-router 192.168.2.1
 dns-server 8.8.8.8
 domain-name kiarisitlab.com

ip dhcp pool POOL3
 network 203.0.113.0 255.255.255.252
```

---

# 6. Request an IP Address on PC2

PC2 was connected directly to the `192.168.2.0/24` network, where R2 is the DHCP server.

The following command was used:

```text id="pc2renew"
C:\>ipconfig /renew
```

PC2 successfully received:

```text id="pc2result"
IP Address......................: 192.168.2.11
Subnet Mask.....................: 255.255.255.0
Default Gateway.................: 192.168.2.1
DNS Server......................: 8.8.8.8
```

The `ipconfig /all` output also confirmed:

```text id="pc2all"
Connection-specific DNS Suffix..: kiarisitlab.com
DHCP Servers....................: 192.168.2.1
DNS Servers.....................: 8.8.8.8
```

### PC2 Result

PC2 successfully obtained its address from **POOL2**:

```text id="pc2summary"
192.168.2.11/24
Gateway: 192.168.2.1
DNS: 8.8.8.8
```

---

# 7. Request an IP Address on PC1

PC1 is located on the `192.168.1.0/24` network.

Because R2 is the DHCP server and is on another subnet, PC1's DHCP request must be relayed by R1.

The following command was used:

```text id="pc1renew"
C:\>ipconfig /renew
```

The first two attempts failed:

```text id="pc1failed"
DHCP request failed.

DHCP request failed.
```

A subsequent request succeeded:

```text id="pc1success"
IP Address......................: 192.168.1.12
Subnet Mask.....................: 255.255.255.0
Default Gateway.................: 192.168.1.1
DNS Server......................: 8.8.8.8
```

The `ipconfig /all` output confirmed:

```text id="pc1all"
Connection-specific DNS Suffix..: karisitlab.com
DHCP Servers....................: 203.0.113.1
DNS Servers.....................: 8.8.8.8
```

### PC1 Result

PC1 successfully obtained its address from **POOL1** through the DHCP relay on R1:

```text id="pc1summary"
192.168.1.12/24
Gateway: 192.168.1.1
DHCP Server: 203.0.113.1
DNS: 8.8.8.8
```

---

# 8. Why PC1 Shows R2 as the DHCP Server

One important observation from PC1's `ipconfig /all` output is:

```text id="pc1dhcpserver"
DHCP Servers....................: 203.0.113.1
```

This is expected.

PC1 is connected to R1, but R1 is acting only as the **DHCP relay agent**. The actual DHCP server is R2 at `203.0.113.1`.

The process is:

```text id="relayprocess"
PC1
 |
 | DHCP Discover
 v
R1
 |
 | ip helper-address 203.0.113.1
 v
R2 DHCP Server
 |
 | DHCP Offer
 v
R1
 |
 v
PC1
```

Therefore, PC1 correctly identifies **203.0.113.1** as its DHCP server.

---

# 9. Final DHCP Address Allocation

| Device | DHCP Pool | Assigned IP | Gateway | DNS | DHCP Server |
|---|---|---|---|---|---|
| R1 G0/0 | POOL3 | `203.0.113.2` | — | — | R2 |
| PC1 | POOL1 | `192.168.1.12` | `192.168.1.1` | `8.8.8.8` | `203.0.113.1` |
| PC2 | POOL2 | `192.168.2.11` | `192.168.2.1` | `8.8.8.8` | `192.168.2.1` |

---

# 10. Final R1 Configuration

The important R1 configuration is:

```text id="r1final"
interface GigabitEthernet0/0
 ip address dhcp
 duplex auto
 speed auto

interface GigabitEthernet0/1
 ip address 192.168.1.1 255.255.255.0
 ip helper-address 203.0.113.1
 duplex auto
 speed auto

ip route 192.168.2.0 255.255.255.0 GigabitEthernet0/0
```

---

# 11. Final R2 Configuration

The important R2 configuration is:

```text id="r2final"
ip dhcp excluded-address 192.168.1.1 192.168.1.10
ip dhcp excluded-address 192.168.2.1 192.168.2.10
ip dhcp excluded-address 203.0.113.1

ip dhcp pool POOL1
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.1
 dns-server 8.8.8.8
 domain-name karisitlab.com

ip dhcp pool POOL2
 network 192.168.2.0 255.255.255.0
 default-router 192.168.2.1
 dns-server 8.8.8.8
 domain-name kiarisitlab.com

ip dhcp pool POOL3
 network 203.0.113.0 255.255.255.252

interface GigabitEthernet0/0
 ip address 203.0.113.1 255.255.255.252

interface GigabitEthernet0/1
 ip address 192.168.2.1 255.255.255.0

ip route 192.168.1.0 255.255.255.0 GigabitEthernet0/0
```

---

# 12. Verification Commands

Useful commands for verifying the lab include:

### R1

```text id="r1verify"
show ip interface brief
show running-config
show ip route
```

### R2

```text id="r2verify"
show ip interface brief
show running-config | section dhcp
show ip dhcp binding
show ip dhcp pool
show ip route
```

### PCs

```text id="pcverify"
ipconfig /all
ipconfig /renew
```

---

# 13. Troubleshooting

### PC1 Initially Failed to Obtain an Address

PC1 initially displayed:

```text id="fail"
DHCP request failed.
```

The subsequent request succeeded after the DHCP relay and server configuration were in place.

When troubleshooting DHCP relay problems, verify:

```text id="troubleshoot"
R1(config)#interface g0/1
R1(config-if)#ip helper-address 203.0.113.1
```

Also verify that R1 can reach R2 and that the DHCP pool exists on R2.

### Check DHCP Bindings

On R2:

```text id="bindings"
R2#show ip dhcp binding
```

This can be used to confirm which addresses have been leased to DHCP clients.

---

# 14. Lab Answers

### Question 1: What DHCP pools were configured?

- **POOL1:** `192.168.1.0/24`
- **POOL2:** `192.168.2.0/24`
- **POOL3:** `203.0.113.0/30`

The first ten addresses were reserved in POOL1 and POOL2, while `203.0.113.1` was reserved in POOL3.

### Question 2: What IP address did R1 configure on G0/0?

R1 received:

```text id="answer2"
203.0.113.2/30
```

### Question 3: How was R1 configured as a DHCP relay?

R1 G0/1 was configured with:

```text id="answer3"
ip helper-address 203.0.113.1
```

### Question 4: What addresses did PC1 and PC2 receive?

**PC1:**

```text id="answerpc1"
192.168.1.12/24
Gateway: 192.168.1.1
DNS: 8.8.8.8
```

**PC2:**

```text id="answerpc2"
192.168.2.11/24
Gateway: 192.168.2.1
DNS: 8.8.8.8
```

---

# Conclusion

This lab demonstrated the configuration of **Cisco IOS DHCP pools**, **DHCP address exclusions**, **DHCP clients**, and **DHCP relay agents**.

R2 was configured as the central DHCP server for three networks. R1 obtained its WAN address dynamically from R2 using POOL3. R1 was then configured as a DHCP relay for the `192.168.1.0/24` LAN, allowing PC1 to obtain an address from R2 even though the DHCP server was on a different subnet.

The final results confirmed that:

- R1 successfully received `203.0.113.2` through DHCP.
- PC1 successfully received `192.168.1.12` through the R1 DHCP relay.
- PC2 successfully received `192.168.2.11` directly from R2.
- Both PCs received `8.8.8.8` as their DNS server.
- The correct default gateways were supplied by the respective DHCP pools.
- DHCP relay functionality was successfully demonstrated.