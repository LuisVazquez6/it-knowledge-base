---
title : Active Directory User Management 
---

# Active Directory User Management 

**Category:** Windows / Active Directory
**Audience:** Help Desk Technicians (Tier 1 & Tier 2)
**Last Update:** 2026-03-25

---

## Table of Contents

1. [Overview]
2. [Creating a New User]
3. [Disabling a User Account]
4. [Unlocking a User Account]
5. [Resetting a User Account]
6. [Adding a User to a Group]
7. [Moving a User to a Different OU]
8. [Deleting a User Account]
9. [PowerShell Quick Reference]
10. [Escalation]

---

## Overview

This guide covers common Active Directory user management tasks performed by Help Desk staff. All tasks can be completed via **Active Directory User and Computers (ADUC)** or **PowerShell**. Always follow the principle of least privilege and verify requests through proper ticketing channels before making changes.

**Important:** Never make AD changes without and approved ticket. Document all actions taken in the ticket before closing.

---

## Creating a New User

**When to use:** New employee onboarding request received from HR or manager

### Via ADUC (GUI)

1. Open **Active Directory Users and Computers** 
2. Navigate to the appropriate **Organizational Unit (OU)**
3. Right-click the OU -> **New** -> **User**
4. Fill in the following fields:
    - First name / Last name
    - User logan name (follow naming convention: firstname.lastname)
5. Click **Next**
6. Set a temporary password (follow password policy)
7. Check **"User must change password at next logon"**
8. Click **Next** -> **Finish**
9. Add the user to the appropriate **security groups**

### Via PowerShell

```powershell
New-ADUser `
    -Name "Jane Doe" `
    -GivenName "Jane" `
    -Surname "Doe" `
    -SamAccountName "jane.doe" `
    -UserPrincipalName "jane.doe@yourdomain.com" `
    -Path "OU=HR,DC=yourdomain,DC=com" `
    -AccountPassword (ConvertTo-SecureString "TempPass123!" -AsPlainText -Force) `
    -ChangePasswordAtLogon $true `
    -Enabled $true
```

---

## Disabling a User Account 

**When to use:** Employee offboarding, suspension, or extended leave.

### Via ADUC (GUI)

1. Open **ADUC**
2. Search for the user account.
3. Right Click the User -> **Disable Account**
4. Confirm the action
5. Move the user to the **Disabled User OU** (if applicable)

### Via PowerShell

```powershell
Disable-ADAccount -Identity "jane.doe"
```

> **Note:** Do not delete accounts immediately upon offboarding. Disable and retain for at least 30 days per company policy.

---

## Unblocking a User Account

**When to use:** User is locked out after too many failed login attempts.

## Via ADUC (GUI)

1. Open **ADUC**
2. Search for the user account
3. Double-click to open **Properties**
4. Go to the **Account** tab
5. Check **"Unlock account"** checkbox
6. Click **Apply** -> **OK**

### Via PowerShell

```powershell
# Unlock a single account
Unlock-ADAccount -Identity "jane.doe"

# Find ALL locked out accounts in the domain
Search-ADAccount -LockedOut | Select-Object Name, SamAccountName
```

---

## Resetting a User Password

**When to use:** User forgot their password or account requires a forced reset.

## Via ADUC (GUI)

1. Open **ADUC**
2. Search for an right-click the user -> **Reset Password**
3. Enter a new temporary password (meet complexity requirements)
4. Check **"User must change password at next logon"**
5. Click **OK**

### Via PowerShell

```powershell
Set-ADAccountPassword -Identity "jane.doe" `
    -NewPassword (ConvertTo-SecureString "NewTempPass123!" -AsPlainText -Force) `
    -Reset

# Force password change at next logon
Set-ADUser -Identity "jane.doe" -ChangePasswordAtLogon $true
```

> **Security:** Never send passwords via email. User a secure method (phone call, in-person, or secure password portal)

---

## Adding a User to a Group

**When to use:** User needs access to shared drive, application, or resource.

### Via ADUC (GUI)

1. Open **ADUC**
2. Navigate to the user -> double-click to open **Properties**
3. Go to the **Member Of** tab
4. Click **Add** -> type the group name -> **Check Names** -> **OK**
5. Click **Apply** -> **OK**

## Via PowerShell

```powershell
Add-ADGroupMember -Identity "HelpDesk_Agents" -Members "jane.doe"

# Verify Group membership
Get-ADGroupMember -Identity "HelpDesk_Agents" | Select-Object Name
```

---

## Moving a User to a Different OU

**When to use:** Employee changes department or role.

### Via ADUC (GUI)

1. Open **ADUC** 
2. Right-Click the user -> **Move**
3. Select the destination OU -> **OK**

### Via PowerShell

```powershell
Move-ADObject `
    -Identity "CN=jane Doe, OU=HR,DC=yourdomain,DC=com" `
    -TargetPath "OU=Finance,DC=yourdomain,DC=com"
```

---

## Deleting a User Account

**When to use:** Only after the 30-day retention period post-offboarding has passed.

### Via ADUC (GUI)

1. Open **ADUC**
2. Search for the disabled user account
3. Right-Click -> **Delete**
4. Confirm the deletion

### Via PowerShell

```powershell
Remove-ADUser -Identity "jane.doe" -Confirm:$false
```

> **Warning:** This action is irreversible. Always confirm with your supervisor before deleting accounts.

---

## PowerShell Quick Reference

| Task | Command |
|---|---|
| Create new User | `New-ADUser` |
| Disable account | `Disable-ADAccount -Identity "username"` |
| Enable account | `Enable-ADAccount -Identity "username"` |
| Unlock account | `Unlock-ADAccount -Identity "username"` |
| Reset password | `Set-ADAccountPassword -Identity "username"` |
| Add to group | `Add-ADGroupMember -Identity "group" -Members "username"` |
| Remove from group | `Remove-ADGroupMember -Identity "group" -Members "username"` |
| Find locked accounts | `Search-ADAccount -LockedOut` |
| Find disabled accounts | `Search-ADAccount -AccountDisabled` |
| Get user details | `Get-ADUser -Identity "username" -Properties *` |

---

## Escalation

Escalate to **Tier 2 / Sysadmin** if:

- You cannot locate the user account in AD
- The account shows signs of compromise (unusual login times, locations)
- Group Policy changes are required
- Domain Controller issues are suspected
- You are unsure about any change that could affect multiple users

**Escalation path:** Help Desk Tier 1 -> Tier 2 -> Sysadmin -> IT Manager

---

*Document Maintained by Luis Vazquez*
