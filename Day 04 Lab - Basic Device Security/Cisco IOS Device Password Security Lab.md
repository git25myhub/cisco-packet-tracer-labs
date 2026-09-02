# Cisco IOS Device Password Security Lab

## 📌 Lab Overview

This lab demonstrates basic Cisco IOS device security configuration on a router and switch. The lab focuses on configuring hostnames, setting enable passwords, encrypting passwords, configuring an encrypted enable secret, testing authentication, and saving the device configuration.

### Devices Used

| Device | Hostname |
|---|---|
| Router | R1 |
| Switch | SW1 |

---

## 🎯 Lab Objectives

By completing this lab, you will learn how to:

1. Change Cisco device hostnames.
2. Configure an unencrypted enable password.
3. Test the enable password.
4. View passwords in the running configuration.
5. Enable password encryption.
6. Configure an encrypted enable secret.
7. Understand the difference between `enable password` and `enable secret`.
8. Identify Cisco password encryption types.
9. Save the running configuration to the startup configuration.

---

# 🛠️ Lab Configuration

## Step 1: Configure Device Hostnames

The `hostname` command is used in global configuration mode.

### R1

```bash
Router> enable
Router# configure terminal
Router(config)# hostname R1
R1(config)#
```

### SW1

```bash
Switch> enable
Switch# configure terminal
Switch(config)# hostname SW1
SW1(config)#
```

---

## Step 2: Configure an Unencrypted Enable Password

Configure the password `CCNA` on both devices.

### R1

```bash
R1(config)# enable password CCNA
```

### SW1

```bash
SW1(config)# enable password CCNA
```

At this stage, the enable password is stored in the configuration in an unencrypted/plain-text form.

---

## Step 3: Test the Enable Password

Exit to user EXEC mode:

```bash
R1(config)# end
R1# disable
R1>
```

Return to privileged EXEC mode:

```bash
R1> enable
Password:
R1#
```

Enter:

```text
CCNA
```

Repeat the same process on SW1.

---

## Step 4: View the Password in the Running Configuration

Use:

```bash
R1# show running-config
```

Look for:

```text
enable password CCNA
```

The password is visible because it has not yet been encrypted.

Repeat on SW1:

```bash
SW1# show running-config
```

---

## Step 5: Enable Password Encryption

Enter global configuration mode and enable password encryption:

```bash
R1# configure terminal
R1(config)# service password-encryption
```

Repeat on SW1:

```bash
SW1# configure terminal
SW1(config)# service password-encryption
```

The `service password-encryption` command encrypts passwords stored in the running configuration.

---

## Step 6: View the Password Again

Exit configuration mode:

```bash
R1(config)# end
```

Display the running configuration:

```bash
R1# show running-config
```

The password should now appear similar to:

```text
enable password 7 <encrypted-password>
```

The exact encrypted value may be different.

The `7` indicates that Cisco's **Type 7** reversible password obfuscation is being used.

---

## Step 7: Configure a More Secure Enable Secret

Configure `Cisco` as the encrypted enable secret.

### R1

```bash
R1# configure terminal
R1(config)# enable secret Cisco
```

### SW1

```bash
SW1# configure terminal
SW1(config)# enable secret Cisco
```

The `enable secret` command stores a cryptographically hashed password and is more secure than the legacy `enable password` command.

---

## Step 8: Test the Enable Secret

Exit back to user EXEC mode:

```bash
R1(config)# end
R1# disable
R1>
```

Return to privileged EXEC mode:

```bash
R1> enable
Password:
```

### Which password should you use?

Use:

```text
Cisco
```

The `enable secret` password takes precedence over the `enable password` when both are configured.

Therefore:

```text
enable password CCNA
enable secret Cisco
```

requires:

```text
Cisco
```

to enter privileged EXEC mode.

---

## Step 9: View the Passwords and Identify Encryption Types

Use:

```bash
R1# show running-config
```

You should see entries similar to:

```text
enable password 7 <encrypted-password>
enable secret 5 <encrypted-hash>
```

### Questions and Answers

#### What encryption type number is used for the encrypted `enable password`?

**Type 7**

```text
enable password 7 <encrypted-password>
```

Type 7 is reversible obfuscation and should not be considered strong password protection.

#### What encryption type number is used for the encrypted `enable secret`?

In this classic Cisco IOS lab, the `enable secret` is commonly displayed as:

**Type 5**

```text
enable secret 5 <hash>
```

Type 5 uses an MD5-based hash.

> **Note:** On newer IOS versions, `enable secret` may use stronger hash types such as Type 8 or Type 9 depending on the configuration and platform. If your Packet Tracer output shows Type 5, Type 5 is the expected answer for this lab.

---

# Step 10: Save the Configuration

Save the running configuration to the startup configuration.

Use:

```bash
R1# copy running-config startup-config
```

Press **Enter** when prompted for the destination filename.

You can also use:

```bash
R1# write memory
```

Repeat on SW1:

```bash
SW1# copy running-config startup-config
```

---

# 🔍 Verification Commands

The following commands can be used to verify the configuration.

### Display the current running configuration

```bash
show running-config
```

### Display the saved startup configuration

```bash
show startup-config
```

### Check the hostname

```bash
show running-config | include hostname
```

### Check the enable password/secret

```bash
show running-config | include enable
```

---

# 📋 Expected Security Configuration

The relevant configuration should look similar to:

```text
hostname R1
!
service password-encryption
!
enable secret 5 <encrypted-hash>
enable password 7 <encrypted-password>
```

SW1 should have the equivalent configuration:

```text
hostname SW1
!
service password-encryption
!
enable secret 5 <encrypted-hash>
enable password 7 <encrypted-password>
```

---

# 🧠 Key Concepts Learned

| Concept | Description |
|---|---|
| `hostname` | Changes the device hostname |
| `enable password` | Legacy privileged EXEC password |
| `enable secret` | More secure privileged EXEC password |
| `service password-encryption` | Encrypts eligible plain-text passwords in the configuration |
| Type 7 | Reversible Cisco password obfuscation |
| Type 5 | MD5-based password hash |
| `show running-config` | Displays the active configuration |
| `show startup-config` | Displays the saved configuration |
| `copy running-config startup-config` | Saves the active configuration |

---

# 🔐 Important Security Note

The following two commands are **not equally secure**:

```bash
enable password CCNA
```

and

```bash
enable secret Cisco
```

`enable secret` should be preferred because it uses a password hash rather than the legacy Type 7 reversible obfuscation used by `enable password`.

When both are configured:

```text
enable password CCNA
enable secret Cisco
```

the **enable secret takes precedence**.

Therefore, the password required to enter privileged EXEC mode is:

```text
Cisco
```

---

# ✅ Lab Completion Checklist

- [x] Changed router hostname to `R1`
- [x] Changed switch hostname to `SW1`
- [x] Configured `enable password CCNA`
- [x] Tested the enable password
- [x] Viewed the password in the running configuration
- [x] Enabled `service password-encryption`
- [x] Verified the encrypted enable password
- [x] Configured `enable secret Cisco`
- [x] Tested privileged EXEC access
- [x] Confirmed that `Cisco` takes precedence
- [x] Identified Type 7 for `enable password`
- [x] Identified Type 5 for the classic `enable secret` output
- [x] Saved the running configuration to startup configuration

---

## 📚 Conclusion

This lab demonstrates the fundamentals of securing Cisco IOS privileged EXEC access. It shows why `enable secret` is preferred over the legacy `enable password` and how `service password-encryption` protects eligible passwords displayed in the device configuration.

The main security takeaway is:

> **Use `enable secret` instead of `enable password` whenever possible.**