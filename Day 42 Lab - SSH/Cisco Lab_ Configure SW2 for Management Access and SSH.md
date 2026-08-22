# Cisco Lab: Configure SW2 for Management Access and SSH

## Lab Objective

Configure a newly added Cisco switch (**SW2**) for basic management, secure console access, and remote SSH access. SSH access must be restricted so that **PC1 is the only permitted source**.

---

## Topology

- **Laptop1** → SW2 Console Port
- **PC1** → Network → SW2
- **R2** → SW2
- **SW2 VLAN 1:** `192.168.2.253/24`
- **R2:** `192.168.2.254/24`

---

## Requirements

### 1. Basic SW2 Configuration

Configure the following:

| Setting | Required Value |
|---|---|
| Hostname | `SW2` |
| Enable secret | `ccna` |
| Local username | `jeremy` |
| Local password | `ccna` |
| VLAN 1 SVI | `192.168.2.253/24` |
| Default gateway | `192.168.2.254` |

### 2. Console Line Security

Configure the console line with:

- Local user authentication
- Exec timeout of **5 minutes**

### 3. SSH Remote Access

Configure SW2 for SSH with:

- Domain name: `jeremysitlab.com`
- RSA key size: **2048 bits**
- Local user authentication
- Exec timeout: **5 minutes**
- SSH only
- Access restricted to **PC1 only**

---

# Configuration

## Step 1 — Connect to SW2 Using the Console

Connect **Laptop1** to SW2's console port.

Enter privileged EXEC mode:

```cisco
SW2> enable
```

Configure global settings:

```cisco
SW2# configure terminal
SW2(config)# hostname SW2
SW2(config)# enable secret ccna
SW2(config)# username jeremy secret ccna
```

---

## Step 2 — Configure the VLAN 1 Management Interface

Configure the SVI:

```cisco
SW2(config)# interface vlan 1
SW2(config-if)# ip address 192.168.2.253 255.255.255.0
SW2(config-if)# no shutdown
SW2(config-if)# exit
```

Configure R2 as the default gateway:

```cisco
SW2(config)# ip default-gateway 192.168.2.254
```

---

## Step 3 — Secure Console Access

Configure the console line to authenticate against the local user database and disconnect inactive sessions after 5 minutes:

```cisco
SW2(config)# line console 0
SW2(config-line)# login local
SW2(config-line)# exec-timeout 5 0
SW2(config-line)# exit
```

The console will now prompt for the configured local username and password.

Example:

```text
User Access Verification

Username: jeremy
Password:
SW2>
```

---

# Step 4 — Configure the SSH Domain Name

SSH RSA key generation requires a domain name to be configured first.

```cisco
SW2(config)# ip domain-name jeremysitlab.com
```

Verify:

```cisco
SW2# show running-config | include domain
```

Expected:

```text
ip domain-name jeremysitlab.com
```

---

# Step 5 — Generate a 2048-bit RSA Key

Generate the RSA keys:

```cisco
SW2(config)# crypto key generate rsa
```

When prompted for the modulus size, enter:

```text
How many bits in the modulus [512]: 2048
```

The 2048-bit key is required for the lab.

Verify the keys:

```cisco
SW2# show crypto key mypubkey rsa
```

> **Important:** The original lab attempt generated a **512-bit RSA key**. The router/switch output showed that this was insufficient for SSH version 2:
>
> ```text
> RSA key size needs to be at least 768 bits for ssh version 2
> ```
>
> Therefore, the correct configuration should use **2048 bits**, not 512 bits.

---

# Step 6 — Configure SSH Version 2

Enable SSH version 2:

```cisco
SW2(config)# ip ssh version 2
```

Verify:

```cisco
SW2# show ip ssh
```

Expected output should indicate that SSH version 2 is enabled.

---

# Step 7 — Configure VTY Lines for SSH

Configure all VTY lines to:

- Use the local user database
- Allow SSH only
- Timeout after 5 minutes

```cisco
SW2(config)# line vty 0 15
SW2(config-line)# login local
SW2(config-line)# exec-timeout 5 0
SW2(config-line)# transport input ssh
```

---

# Step 8 — Restrict SSH Access to PC1

PC1 has the IP address:

```text
192.168.1.1
```

Create a standard numbered ACL that permits only PC1:

```cisco
SW2(config)# access-list 1 permit host 192.168.1.1
```

Apply the ACL inbound to all VTY lines:

```cisco
SW2(config)# line vty 0 15
SW2(config-line)# access-class 1 in
```

This prevents devices other than PC1 from establishing a remote VTY session.

---

# Step 9 — Save the Configuration

Return to privileged EXEC mode:

```cisco
SW2(config-line)# end
```

Save the configuration:

```cisco
SW2# write memory
```

or:

```cisco
SW2# copy running-config startup-config
```

Expected:

```text
Building configuration...

[OK]
```

---

# Verification

## Verify the Running Configuration

```cisco
SW2# show running-config
```

Important sections should resemble:

```cisco
hostname SW2

enable secret ccna

username jeremy secret ccna

ip ssh version 2
ip domain-name jeremysitlab.com

interface Vlan1
 ip address 192.168.2.253 255.255.255.0
 no shutdown

ip default-gateway 192.168.2.254

access-list 1 permit host 192.168.1.1

line con 0
 login local
 exec-timeout 5 0

line vty 0 15
 access-class 1 in
 exec-timeout 5 0
 login local
 transport input ssh
```

---

## Verify the VLAN 1 SVI

```cisco
SW2# show ip interface brief
```

Expected:

```text
Interface              IP-Address      OK? Method Status
Vlan1                  192.168.2.253   YES manual up
```

The VLAN 1 interface must be **up/up** for management connectivity.

---

## Test Connectivity From R2

From R2:

```cisco
R2# ping 192.168.2.253
```

Expected:

```text
Success rate is 100 percent
```

Some Packet Tracer simulations may lose the first packet while ARP is resolved.

---

## Test SSH From PC1

From PC1:

```text
C:\> ssh -l jeremy 192.168.2.253
```

Enter the password:

```text
Password: ccna
```

Successful authentication should result in:

```text
SW2>
```

Then enter privileged EXEC mode:

```cisco
SW2> enable
Password: ccna
SW2#
```

---

# Verify SSH Configuration

On SW2:

```cisco
SW2# show ip ssh
```

Confirm that SSH is enabled and using **version 2**.

Also verify the VTY configuration:

```cisco
SW2# show running-config | section line vty
```

Expected:

```cisco
line vty 0 4
 access-class 1 in
 exec-timeout 5 0
 login local
 transport input ssh
line vty 5 15
 access-class 1 in
 exec-timeout 5 0
 login local
 transport input ssh
```

---

# Verify the Access Control List

```cisco
SW2# show access-lists
```

Expected:

```text
Standard IP access list 1
    10 permit host 192.168.1.1
```

This confirms that **PC1 (192.168.1.1)** is the only source permitted to access the VTY lines.

---

# Troubleshooting

## SSH Connection Refused

If you see:

```text
% Connection refused by remote host
```

check the following:

### 1. Verify RSA keys

```cisco
SW2# show crypto key mypubkey rsa
```

If keys do not exist, generate them again using **2048 bits**.

### 2. Verify SSH version

```cisco
SW2# show ip ssh
```

Configure:

```cisco
SW2(config)# ip ssh version 2
```

### 3. Verify VTY configuration

```cisco
SW2# show running-config | section line vty
```

Make sure:

```cisco
login local
transport input ssh
```

are configured.

### 4. Verify the local username

```cisco
SW2# show running-config | include username
```

The configured username should match the credentials you use for SSH.

### 5. Verify the ACL

```cisco
SW2# show access-lists
```

The ACL must permit PC1's IP address:

```cisco
access-list 1 permit host 192.168.1.1
```

---

# Important Lessons From This Lab

- A switch requires an **SVI** for Layer 3 management.
- The switch's **default gateway** should point toward the router on the management subnet.
- `login local` makes Cisco IOS authenticate against locally configured usernames.
- SSH requires a configured **domain name** before RSA keys can be generated.
- A **2048-bit RSA key** provides the required key size for this lab.
- `transport input ssh` disables Telnet and allows SSH connections only.
- `access-class 1 in` restricts VTY access using a standard ACL.
- The ACL can be used to ensure that **only PC1** can remotely manage SW2.
- Always save the configuration with `write memory` or `copy running-config startup-config`.

---

# Final Verification Checklist

- [ ] SW2 hostname configured
- [ ] Enable secret `ccna` configured
- [ ] Local user `jeremy` with password `ccna` configured
- [ ] VLAN 1 configured as `192.168.2.253/24`
- [ ] VLAN 1 is up
- [ ] Default gateway configured as `192.168.2.254`
- [ ] Console uses local authentication
- [ ] Console exec timeout set to 5 minutes
- [ ] Domain name `jeremysitlab.com` configured
- [ ] 2048-bit RSA keys generated
- [ ] SSH version 2 enabled
- [ ] VTY lines use local authentication
- [ ] VTY exec timeout set to 5 minutes
- [ ] VTY lines accept SSH only
- [ ] ACL permits PC1 (`192.168.1.1`) only
- [ ] ACL applied inbound to VTY lines
- [ ] PC1 can successfully SSH into SW2
- [ ] Configuration saved to startup-config