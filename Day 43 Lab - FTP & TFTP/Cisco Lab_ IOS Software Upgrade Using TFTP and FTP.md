# Cisco Lab: IOS Software Upgrade Using TFTP and FTP

## Lab Objective

In this lab, routers **R1** and **R2** are upgraded to a newer Cisco IOS image stored on **SRV1**.

The lab demonstrates:

- IP addressing and routing
- Verifying network connectivity
- Transferring an IOS image using **TFTP**
- Transferring an IOS image using **FTP**
- Configuring the router to boot from the new IOS image
- Reloading the router to apply the upgrade
- Deleting the old IOS image from flash
- Verifying the new IOS version

---

# Lab Requirements

## Devices

- R1 — Cisco 2911
- R2 — Cisco 2911
- SRV1 — Server providing TFTP and FTP services

## IOS Image

Both routers must be upgraded to:

```text
c2900-universalk9-mz.SPA.155-3.M4a.bin
```

## TFTP

R1 retrieves the IOS image from:

```text
SRV1: 10.0.0.1
```

## FTP

R2 retrieves the IOS image from:

```text
SRV1: 10.0.0.1
```

FTP credentials:

```text
Username: jeremy
Password: ccna
```

> **Note:** The FTP transfer may take approximately one minute in Packet Tracer. Do not interrupt the transfer.

---

# 1. Configure IP Addresses and Routing

The first requirement is to configure the appropriate IP addresses on every device.

After configuring the interfaces, verify them with:

```cisco
R1# show ip interface brief
R2# show ip interface brief
```

Interfaces that are being used should show:

```text
Status     up
Protocol   up
```

If an interface is administratively down, enable it:

```cisco
R1(config)# interface g0/0
R1(config-if)# no shutdown
```

Use the appropriate interface and IP address for the topology.

---

# 2. Configure Routing for Full Connectivity

The routers must be able to reach SRV1 before the file transfers can work.

First verify the routing table:

```cisco
R1# show ip route
R2# show ip route
```

Configure the required static routes according to the topology.

For example:

```cisco
R1(config)# ip route <destination-network> <subnet-mask> <next-hop>
```

and:

```cisco
R2(config)# ip route <destination-network> <subnet-mask> <next-hop>
```

If the topology uses a default route, configure it as appropriate:

```cisco
R1(config)# ip route 0.0.0.0 0.0.0.0 <next-hop>
```

---

# 3. Verify Connectivity to SRV1

Before attempting TFTP or FTP, verify that both routers can reach SRV1.

From R1:

```cisco
R1# ping 10.0.0.1
```

From R2:

```cisco
R2# ping 10.0.0.1
```

The ping should succeed.

For example:

```text
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.0.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent
```

> **Important:** If the ping to `10.0.0.1` fails, fix IP addressing or routing before attempting the file transfer. A TFTP/FTP timeout usually indicates a connectivity or server-service problem.

---

# 4. Verify the IOS Image on SRV1

Open **SRV1** in Packet Tracer.

Go to:

**Services → TFTP**

Make sure the TFTP service is:

```text
On
```

Confirm that the following file exists:

```text
c2900-universalk9-mz.SPA.155-3.M4a.bin
```

The screenshot for this lab shows the file available on SRV1.

For FTP, go to:

**Services → FTP**

Make sure the FTP service is enabled and that the user exists:

```text
Username: jeremy
Password: ccna
```

---

# 5. Check R1's Current IOS Version

Before upgrading R1, check the currently running IOS:

```cisco
R1# show version
```

The original R1 configuration shows:

```text
Cisco IOS Software, C2900 Software (C2900-UNIVERSALK9-M),
Version 15.1(4)M4
```

The currently booted image is:

```text
flash0:c2900-universalk9-mz.SPA.151-1.M4.bin
```

Also check the flash contents:

```cisco
R1# show flash:
```

The lab environment may contain multiple IOS files.

---

# 6. Copy the New IOS Image to R1 Using TFTP

From R1, use:

```cisco
R1# copy tftp: flash:
```

When prompted:

```text
Address or name of remote host []?
```

Enter:

```text
10.0.0.1
```

For the source filename:

```text
c2900-universalk9-mz.SPA.155-3.M4a.bin
```

For the destination filename, press **Enter** to accept the same filename.

The complete interaction should look similar to:

```text
R1# copy tftp: flash:
Address or name of remote host []? 10.0.0.1
Source filename []? c2900-universalk9-mz.SPA.155-3.M4a.bin
Destination filename [c2900-universalk9-mz.SPA.155-3.M4a.bin]?
```

Packet Tracer should then begin transferring the file.

---

# 7. Verify the New IOS File on R1

After the transfer completes:

```cisco
R1# show flash:
```

Confirm that the new image exists:

```text
c2900-universalk9-mz.SPA.155-3.M4a.bin
```

Check the available flash space as well.

The new IOS image must be completely transferred before continuing.

---

# 8. Configure R1 to Boot the New IOS

Configure the new image as the boot image:

```cisco
R1# configure terminal
R1(config)# boot system flash:c2900-universalk9-mz.SPA.155-3.M4a.bin
R1(config)# end
```

Verify the boot variable:

```cisco
R1# show running-config | include boot
```

You should see:

```text
boot system flash:c2900-universalk9-mz.SPA.155-3.M4a.bin
```

Save the configuration:

```cisco
R1# copy running-config startup-config
```

---

# 9. Reload R1

Reload the router:

```cisco
R1# reload
```

Confirm the reload when prompted.

R1 will restart and load the new IOS image.

---

# 10. Verify R1's IOS Upgrade

After R1 finishes booting:

```cisco
R1# show version
```

Verify that the router is now running the new IOS version.

The image should correspond to:

```text
c2900-universalk9-mz.SPA.155-3.M4a.bin
```

You can also verify the boot image:

```cisco
R1# show version | include System image
```

---

# 11. Delete the Old IOS Image From R1

After confirming that R1 successfully boots using the new image, check flash:

```cisco
R1# show flash:
```

Identify the old IOS image.

For example:

```text
c2900-universalk9-mz.SPA.151-1.M4.bin
```

Delete the old image:

```cisco
R1# delete flash:c2900-universalk9-mz.SPA.151-1.M4.bin
```

Confirm the deletion when prompted.

Then verify:

```cisco
R1# show flash:
```

The old IOS image should no longer be present.

> **Important:** Do not delete the currently running IOS image. Always verify `show version` and `show flash:` before deleting an image.

---

# 12. Check R2's Current IOS Version

On R2:

```cisco
R2# show version
```

The original R2 configuration shows:

```text
Cisco IOS Software, C2900 Software (C2900-UNIVERSALK9-M),
Version 15.1(4)M4
```

The currently booted image is:

```text
flash0:c2900-universalk9-mz.SPA.151-1.M4.bin
```

Check the flash contents:

```cisco
R2# show flash:
```

---

# 13. Configure FTP Credentials on R2

R2 needs the FTP username and password before retrieving the IOS image.

Enter configuration mode:

```cisco
R2# configure terminal
```

Configure the FTP username:

```cisco
R2(config)# ip ftp username jeremy
```

Configure the FTP password:

```cisco
R2(config)# ip ftp password ccna
```

Exit configuration mode:

```cisco
R2(config)# end
```

Verify:

```cisco
R2# show running-config | include ip ftp
```

---

# 14. Copy the New IOS Image to R2 Using FTP

Use:

```cisco
R2# copy ftp: flash:
```

When prompted for the remote host:

```text
Address or name of remote host []? 10.0.0.1
```

Source filename:

```text
c2900-universalk9-mz.SPA.155-3.M4a.bin
```

Destination filename:

```text
c2900-universalk9-mz.SPA.155-3.M4a.bin
```

The process should resemble:

```text
R2# copy ftp: flash:
Address or name of remote host []? 10.0.0.1
Source filename []? c2900-universalk9-mz.SPA.155-3.M4a.bin
Destination filename [c2900-universalk9-mz.SPA.155-3.M4a.bin]?
```

The FTP transfer will begin.

> **Important:** The lab specifically warns that this transfer may take about one minute. Allow the transfer to finish.

---

# 15. Verify the New IOS File on R2

After the FTP transfer completes:

```cisco
R2# show flash:
```

Confirm that:

```text
c2900-universalk9-mz.SPA.155-3.M4a.bin
```

is present.

---

# 16. Configure R2 to Boot the New IOS

Configure the new IOS as the boot image:

```cisco
R2# configure terminal
R2(config)# boot system flash:c2900-universalk9-mz.SPA.155-3.M4a.bin
R2(config)# end
```

Verify:

```cisco
R2# show running-config | include boot
```

Expected:

```text
boot system flash:c2900-universalk9-mz.SPA.155-3.M4a.bin
```

Save the configuration:

```cisco
R2# copy running-config startup-config
```

---

# 17. Reload R2

Reload the router:

```cisco
R2# reload
```

Confirm the reload when prompted.

Wait for R2 to completely reboot.

---

# 18. Verify R2's IOS Upgrade

After R2 boots:

```cisco
R2# show version
```

Verify that the new IOS image is running:

```text
c2900-universalk9-mz.SPA.155-3.M4a.bin
```

You can also use:

```cisco
R2# show version | include System image
```

---

# 19. Delete the Old IOS Image From R2

Check the flash contents:

```cisco
R2# show flash:
```

Identify the old IOS image.

For example:

```text
c2900-universalk9-mz.SPA.151-1.M4.bin
```

Delete it:

```cisco
R2# delete flash:c2900-universalk9-mz.SPA.151-1.M4.bin
```

Confirm the deletion.

Then verify:

```cisco
R2# show flash:
```

The old IOS image should be removed while the new IOS image remains.

---

# Troubleshooting

## TFTP Transfer Times Out

During the attempted R1 transfer, the following error occurred:

```text
Accessing tftp://10.0.0.1/c2900-universalk9-mz.SPA.155-3.M4a.bin........
%Error opening tftp://10.0.0.1/c2900-universalk9-mz.SPA.155-3.M4a.bin (Timed out)
```

A TFTP timeout does **not** necessarily mean the filename is wrong.

Check these items in order.

### 1. Ping SRV1

```cisco
R1# ping 10.0.0.1
```

If this fails, fix routing/connectivity first.

### 2. Verify TFTP Is Enabled

On SRV1:

**Services → TFTP → On**

### 3. Verify the Filename

The filename must exactly match:

```text
c2900-universalk9-mz.SPA.155-3.M4a.bin
```

Cisco IOS filenames are case-sensitive.

### 4. Verify the Server IP

The lab uses:

```text
10.0.0.1
```

### 5. Check Routing

```cisco
R1# show ip route
```

R1 must have a route to the SRV1 network.

---

# FTP Transfer Timeout Troubleshooting

The attempted R2 transfer produced:

```text
Accessing ftp://10.0.0.1/c2900-universalk9-mz.SPA.155-3.M4a.bin...
%Error opening ftp:///c2900-universalk9-mz.SPA.155-3.M4a.bin (Timed out)
```

Check the following.

## 1. Verify R2 Can Reach SRV1

```cisco
R2# ping 10.0.0.1
```

The ping should succeed.

## 2. Verify FTP Is Enabled

On SRV1:

**Services → FTP → On**

## 3. Verify the FTP User

The credentials must be:

```text
Username: jeremy
Password: ccna
```

## 4. Verify R2's FTP Configuration

```cisco
R2# show running-config | include ip ftp
```

Expected configuration:

```text
ip ftp username jeremy
ip ftp password ccna
```

## 5. Verify the File Exists on SRV1

Confirm:

```text
c2900-universalk9-mz.SPA.155-3.M4a.bin
```

exists under the server's FTP files.

## 6. Wait for the Transfer

The lab explicitly warns:

```text
THE TRANSFER MAY TAKE ABOUT A MINUTE
```

Do not cancel the transfer prematurely.

---

# Useful Verification Commands

## Check Interfaces

```cisco
show ip interface brief
```

## Check Routing

```cisco
show ip route
```

## Check Connectivity

```cisco
ping 10.0.0.1
```

## Check IOS Version

```cisco
show version
```

## Check Flash

```cisco
show flash:
```

## Check Boot Variable

```cisco
show running-config | include boot
```

## Check FTP Configuration

```cisco
show running-config | include ip ftp
```

---

# Final Verification Checklist

- [ ] IP addresses configured on all required devices
- [ ] All required interfaces are `up/up`
- [ ] Routing configured between networks
- [ ] R1 can ping SRV1
- [ ] R2 can ping SRV1
- [ ] TFTP service enabled on SRV1
- [ ] FTP service enabled on SRV1
- [ ] IOS file exists on SRV1
- [ ] R1 retrieves the IOS image using TFTP
- [ ] New IOS image exists in R1 flash
- [ ] R1 boot variable points to the new IOS
- [ ] R1 configuration saved
- [ ] R1 successfully boots the new IOS
- [ ] Old R1 IOS image deleted from flash
- [ ] FTP username `jeremy` configured on R2
- [ ] FTP password `ccna` configured on R2
- [ ] R2 retrieves the IOS image using FTP
- [ ] New IOS image exists in R2 flash
- [ ] R2 boot variable points to the new IOS
- [ ] R2 configuration saved
- [ ] R2 successfully boots the new IOS
- [ ] Old R2 IOS image deleted from flash
- [ ] Final `show version` confirms the new IOS image

---

# Key Commands Summary

### R1 — TFTP Upgrade

```cisco
R1# ping 10.0.0.1
R1# show flash:
R1# copy tftp: flash:
Address or name of remote host []? 10.0.0.1
Source filename []? c2900-universalk9-mz.SPA.155-3.M4a.bin
Destination filename [c2900-universalk9-mz.SPA.155-3.M4a.bin]?
R1# configure terminal
R1(config)# boot system flash:c2900-universalk9-mz.SPA.155-3.M4a.bin
R1(config)# end
R1# copy running-config startup-config
R1# reload
R1# show version
R1# show flash:
```

### R2 — FTP Upgrade

```cisco
R2# ping 10.0.0.1
R2# configure terminal
R2(config)# ip ftp username jeremy
R2(config)# ip ftp password ccna
R2(config)# end
R2# copy ftp: flash:
Address or name of remote host []? 10.0.0.1
Source filename []? c2900-universalk9-mz.SPA.155-3.M4a.bin
Destination filename [c2900-universalk9-mz.SPA.155-3.M4a.bin]?
R2# configure terminal
R2(config)# boot system flash:c2900-universalk9-mz.SPA.155-3.M4a.bin
R2(config)# end
R2# copy running-config startup-config
R2# reload
R2# show version
R2# show flash:
```

> **Lesson learned:** The most important prerequisite for TFTP/FTP IOS upgrades is end-to-end connectivity. Before troubleshooting the transfer command itself, verify `ping 10.0.0.1`, routing, the server service, and the exact filename. Once the image is successfully copied, configure the boot variable, save the configuration, reload, verify the new IOS, and only then delete the old image.