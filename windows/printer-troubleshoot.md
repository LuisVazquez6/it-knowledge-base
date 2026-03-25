---
title: Printer Troubleshooting Guide
---

# Printer Troubleshooting Guide

## Overview
This Guide covers the most common printer issues reported to the IT help desk

## Apploes to
- All network and locally connected printers
- Windows 10 and Windows 11
- Wired and wireless printer connections 

---

## Issue 1 - Printer Shows as Offline
1. Click **Start** and open **Settings > Bluetooth and Devices > Printers and Scanners** 
2. Click on the Printer showing as Offline 
3. Click **Open printer queue**
4. Click **Printer** in the menu bar and uncheck **"Use Printer Offline"** if it is checked
5. Restart the printer by turning it off waiting 30 seconds, and turning it back on
6. Restart the Print Spooler service: Open **Services** > find **Print Spooler** > restart

---

## Issue 2 - Print Job Stuck in Queue

1. Open **Settings > Bluetooth and Devices > Printers and Scanners**
2. Click the printer and select **Open print queue**
3. Right Clickthe stuck job and select **Cancel**
4. If the job cannot be cancelled, open the Run dialog (**Win+R**) and type: `services.msc`
5. Find **Print Spooler**, right-click and select **Stop**
6. Navigate to `C:\Windows\System32\spool\PRINTERS` and **delete all files inside** (not the folder itself).
7. Go back to Services, right-click **Printer Spooler** and select **Start**
8. Try printing again.

> ⚠️ **Warning:** Do NOT delete the PRINTERS folder itself — only delete the files inside it.

---

## Issue 3 - Reinstalling a Printer 

1. Go to **Settings > Bluetooth and Devices > Printers and Scanners**
2. Click the Problem printer and select **Remove**
3. Clikc **Add a printer or scanner** and wait for Windows to search
4. Select the correct printer from the list and clikc **Add Device**
5. If the printer does not appear, click **"The printer i want is not listed"** and enter the IP address manually

---

## Common Errors & Fixes
| Error | Fix |
|---|---|
| Driver unavailabe | Download latest driver from manufacturer website and reinstall |
| Access Denied when printing | Check user permissions in print server or local printer |
| Printer prints blank pages | Check ink/toner levels and run printer self-test |
| Wrong printer is set as default | Go to Printers and Scanners, click the correct printer |

---

# Escalation 
Escalate to **Tier2** if:
- The printer requires a new driver installation from a print server
- The printer is a network printer with IP configuration issues
- The issue affects multiple users on the name printer 
- Hardware inspection or replacement is required

---

## Related Articles
- [Ticket Escalation Policy](../ticketing/escalation-policy.md)
