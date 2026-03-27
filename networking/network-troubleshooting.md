---
Title: Network Connectivity Troubleshooting Guide
---

# Network Connectivity Troubleshooting Guide

**Category:** Network
**Audience:** Help Desk Techs (Tier 1 & Tier 2)
**Last Updated:** 2026-03-26

---

## Table of Contents

1. [Overview](#overview)
2. [Step 1 — Check Physical Connection](#step-1--check-physical-connection)
3. [Step 2 — Check IP Configuration](#step-2--check-ip-configuration)
4. [Step 3 — Test Connectivity with Ping](#step-3--test-connectivity-with-ping)
5. [Step 4 — Check DNS Resolution](#step-4--check-dns-resolution)
6. [Step 5 — Flush DNS & Renew IP](#step-5--flush-dns--renew-ip)
7. [Step 6 — Check Network Adapter](#step-6--check-network-adapter)
8. [Step 7 — Reset Network Stack](#step-7--reset-network-stack)
9. [Common Error Messages](#common-error-messages)
10. [Quick Command Reference](#quick-command-reference)
11. [Escalation](#escalation)

---

## Overview

This guide walks through the standard troubleshooting process for network connectivity issues on Windows machines in a domain environment. Follow each step in order before escalating to Tier 2.

> **Important:** Always open a ticket before making changes. Document every step taken and its result.

---

## Step 1 - Check Physical Connection

Before touching any settings, rule out physical issues.

**For wired connections:**

1. Check that the **ethernet cable is firmly plugged in** to both the PC and the wall/switch
2. Look at the **NIC port LED lights** on the back of the PC:
    - Solid or blinking green = connected
    - No light = no link detected
3. Try a **different ethernet cable**
4. Try a **different wall port** if available
5. Check if other users on the same switch are affected 

**For wireless connections:**

1. Confirm **Wi-Fi is turned on** (check the taskbar icon)
2. Make sure **Airplane Mode is off**
3. Try forgetting the network and reconnecting:
    - `Settings` -> `Network & Internet` -> `Wi-Fi` -> `Manage known networks`
    - Select the network -> **Forget** -> reconnect

> **Note:** If multiple users on the same switch or access point are affected, this is likely an infrastructure issue - escalate immediately.

---

## Step 2 - Check IP Configuration

Verify the machine has a Valid IP address.

1. Open **Command Prompt** (`cmd`) as Administrator 
2. Run the following command;

```powershell
ipconfig /all
```

3. Check the output for the active adapter:

| Field | Expected Value |
|---|---|
| IPv4 Address | Should match your subnet (e.g. `192.168.1.x`) |
| Subnet Mask | Typically `255.255.255.0` |
| Default Gateway | Should be your router/gateway IP |
| DHCP Enabled | `Yes` (unless static IP is set) |
| DNS Servers | Should point to your domain controller |

> **APIPA Address Warning:** If the IPv4 address starts with `169.254.x.x`, the machine **failed to get an IP from DHCP**. Proceed to [Step 5](#step-5--flush-dns--renew-ip)

---

## Step 3 - Test Connectivity with Ping

Use ping to identify exactly where the connection breaks down.

Run each command in order and note which one fails:

```powershell
# 1. Ping the loopback (tests if TCP/IP stack is working)
ping 127.0.0.1

# 2. Ping the default gateway (test local network)
ping 192.168.1.1

# 3. Ping a known external IP (tests internet without DNS)
ping 8.8.8.8

# 4. Ping a domain name (tests DNS resolution)
ping google.com
```

**Interpreting results:**

| Ping fails at... | Likely cause |
|---|---|
| `127.0.0.1` | TCP/IP stack corrupted - proceed to step 7 |
| Default gateway | Local network issue - check cable/switch/router |
| `8.8.8.8` | ISP or firewall issue - escalate |
| `google.com` only | DNS issue - proceed to step 4 |

---

## Step 4 - Check DNS Resolution

If pinging an IP works but domain names fail, the issue is DNS.

1. Open **Command Prompt** as Administrator
2. Run:

```powershell
# Test DNS resolution manually
nslookup google.com

# check which DNS server is being used
nslookup google.com 8.8.8.8
```

3. If `nslookup` returns an error or wrong IP:
    - Verify the DNS server IP in `ipconfig /all`
    - Confirm the DNS server is the **domain controller** IP (in a domain environment)
    - Try setting DNS manually to `8.8.8.8` temporarily to isolate the issue:
        - `Control Panel` -> `Network & Internet` -> `Network Connections`
        - Right-click adapter -> `Properties` -> `IPv4` -> set DNS to `8.8.8.8`

> **Note:** In and Active Directory environment, DNS should always point to the domain controller - not a public DNS server permanently. Only use `8.8.8.8` for testing.

---

## Step 5 - Flush DNS & Renew IP

Clear cached DNS entries and request a fresh IP from DHCP.

1. Open **Command Prompt** as Administrator
2. Run the following commands **in order**:

```powershell
# Release the current IP address
ipconfig /release

# Flush the DNS cache 
ipconfig  /flushdns

# Request a new IP from DHCP
ipconfig /renew
```

3. Run `ipconfig /all` again to confirm a valid IP was assigned
4. Test connectivity again with `ping google.com`

> **Note:** ipconfig /release` will briefly disconnect the machine from the network.

---

## Step 6 - Check Network Adapter

If the above steps haven't resolved the issue, check the adapter itself.

**Via Device Manager:**

1. Right-click **Start** -> **Device Manager**
2. Expand **Network Adapter**
3. Look for any adapter with a yellow warning icon
4. Right click the adapter -> **Disable Device** -> wait 5 seconds -> **Enable device**
5. If that doesn't work, right-click -> **Uninstall device** -> restart the PC (Window will reinstall the driver)

**Via PowerShell:**

```powershell
# View all network adapters and their status
Get-NetAdapter

# Restart a specific adapter (release "Ethernet" with adapter name)
Restart-NetAdapter -Name "Ethernet"

# Check adapter status
Get-NetAdapter | Select-object Name, Status, Linkspeed
```

---

## Step 7 - Reset Network Stack

Use this as  a last resort before escalating. This resets all network settings to default. 

1. Open **Command Prompt** as Administrator
2. Run the following commands **in order**:

```powershell
# Reset Winsock catalog 
netsh winsock reset

# Reset TCP/IP stack
netsh int ip reset

# Reset Windows Firewall rules
netsh advfirewall reset

# Flush DNS one more time
ipconfig /flushdns 
```

3. **Restart the computer**
4. Test connectivity after reboot

> **Warning:** `netsh winsock reset` can affect third-party network software (VPNs, firewalls). Document this step in the ticket and notify the user.

---

## Common Error Messages 

| Error Message | Meaning | Action |
|---|---|---|
| `Request timed out` | Host is unreachable or blocking ICMP | Check firewall, try a different target |
| `Destination host unreachable` | No route to the host | Check gateway and routing |
| `DNS request timed out` | DNS server not responding | Check DNS  settings, try Step 4 |
| `169.254.x.x IP address` | DHCP failed (APIPA address) | Check DHCP server, run Step 5 |
| `Media disconnected` | No physical link detected | Check cable and NIC (Step 1) |
| `General failure` | TCP/IP stack issue | Run Step 7 (network stack reset) |

---

## Quick Command Reference

| Task | Command |
|---|---|
| View IP Configuration | `ipconfig /all` |
| Ping loopback | `ping 127.0.0.1` |
| Ping external IP | `ping 8.8.8.8` |
| Test  DNS | `nslookup google.com` |
| Release IP | `ipconfig /release` |
| Renew IP | `ipconfig /renew` |
| Flush DNS cache | `ipconfig /flushdns` |
| View adapters | `Get-NetAdapter` |
| Restart adapters | `Restart-NetAdapter -Name "Ethernet"` |
| Reset Winsock | `netsh winsock reset` |
| Reset TCP/IP | `netsh int ip reset` |
| Trace route to host | `tracert google.com` |

---

## Escalation

Escalate to **Tier 2 / Network Admin** if:

- Multiple users are affected on the same switch or VLAN
- The DHCP server is not issuing addresses
- DNS server is unreachable from multiple machines
- Network stack reset did not resolve the issue
- The issue is intermittent and cannot be reproduced
- Hardware failure is suspected (switch, router, NIC)

**Escalation Path:** Help Desk Tier 1 -> Tier 2 -> Network Admin -> IT Manager

---

*Document maintained by Luis Vazquez*
