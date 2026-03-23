# 🖥️ Windows Server 2022 Administration Lab

> **Full Active Directory domain deployment with DNS, DHCP, Group Policy, IIS, file shares, audit policy, system state backup, and PowerShell automation — built from scratch on VMware.**

---

## 📋 Project Overview

This project simulates a single-site enterprise Windows Server environment. A Windows Server 2022 Domain Controller (DC01) manages a domain-joined Windows 10 client (CLIENT1) with full identity management, network services, security hardening, and automation using PowerShell scripts.

Every component was configured, tested, and documented following Microsoft best practices.

---

## 🏗️ Lab Architecture

```
Internet (VMware NAT - VMnet8)
          |
     DC01 (Windows Server 2022)
     192.168.10.2 — Domain Controller
     ├── AD DS:    lab.local forest
     ├── DNS:      lab.local forward zone
     ├── DHCP:     192.168.10.50–100
     ├── GPO:      5 Group Policy Objects
     ├── IIS:      http://DC01.lab.local
     ├── Backup:   System State (~8.1 GB)
     └── Scripts:  7 PowerShell automation scripts
          |
     VMnet1 (Host-only — 192.168.10.0/24)
          |
     CLIENT1 (Windows 10 Pro)
     192.168.10.50 — Domain Member
```

---

## 🛠️ Technologies & Features

| Category | Implementation |
|---|---|
| Active Directory | New forest `lab.local`, DC01 promoted, `IT_DEPARTMENT` OU |
| DNS | Forward lookup zone, A records, auto-registration |
| DHCP | Scope .50–.100, 8-day lease, MAC reservation for CLIENT1 |
| Group Policy | 5 GPOs — password policy, lockout, wallpaper, control panel restriction, IE zone |
| File Shares | SharedFiles (read-only), IT_Share (modify), Wallpaper (GPO source), Backup |
| NTFS Permissions | Dual-layer Share + NTFS permissions, inheritance disabled on IT_Share |
| IIS Web Server | Internal site at `http://DC01.lab.local` — port 80, custom HTML |
| Remote Desktop | RDP enabled, `vasanth.it` granted access via GPO (not local admin) |
| Audit Policy | 4 audit categories — logon, account management, policy change tracked in Event Viewer |
| System State Backup | AD database (ntds.dit), SYSVOL, DNS zones, registry — ~8.1 GB |
| PowerShell | 7 automation scripts for service monitoring, bulk user creation, reporting, and more |

---

## 📡 Network Configuration

| Device | Interface | IP | Role |
|---|---|---|---|
| DC01 | Ethernet0 (VMnet1) | 192.168.10.2 | Static — Domain Controller, DNS, DHCP |
| DC01 | Ethernet1 (VMnet8) | 192.168.58.x | DHCP — Internet access only |
| CLIENT1 | Ethernet0 (VMnet1) | 192.168.10.50 | DHCP reserved by MAC |
| VMware Host | VMnet1 | 192.168.10.1 | Host access |

---

## 🔒 Group Policy Objects

| GPO Name | Linked To | Settings |
|---|---|---|
| Default Domain Policy | `lab.local` | Password: min 8 chars, complexity, 90-day max |
| Default Domain Policy | `lab.local` | Lockout: 3 attempts, 30 min lock, 15 min reset |
| IT_DEPT_Wallpaper | `IT_DEPARTMENT` OU | Desktop wallpaper via UNC path |
| IT_DEPT_Restrictions | `IT_DEPARTMENT` OU | Block access to Control Panel & PC Settings |
| IT_DEPT_IEZone | `IT_DEPARTMENT` OU | Add DC01.lab.local to IE Intranet Zone |

---

## 🤖 PowerShell Automation Scripts

| Script | Purpose |
|---|---|
| `ServiceCheck.ps1` | Checks 7 critical Windows services — color-coded green/red output |
| `BulkCreateUsers.ps1` | Creates multiple AD users from CSV — full account provisioning |
| `ADUserReport.ps1` | Exports all OU users to CSV — Name, SAM, enabled status, last password set |
| `UnlockAccounts.ps1` | Finds and unlocks all locked AD accounts — most common helpdesk task |
| `InactiveUsers.ps1` | Finds users inactive 30+ days — security audit and cleanup |
| `AddUsersToGroup.ps1` | Adds all OU users to security group — role-based access control |
| `UpdateDepartment.ps1` | Updates Department attribute for all OU users in bulk |

---

## ✅ Testing Results

| Test | Result |
|---|---|
| Domain join — CLIENT1 to lab.local | ✅ Pass |
| DNS resolution — CLIENT1 → DC01.lab.local | ✅ Pass |
| DHCP lease + MAC reservation | ✅ Pass |
| Domain user login (vasanth.it) | ✅ Pass |
| GPO — Password policy enforced | ✅ Pass |
| GPO — Account lockout after 3 fails | ✅ Pass |
| GPO — Wallpaper applied to CLIENT1 | ✅ Pass |
| GPO — Control Panel blocked | ✅ Pass |
| File share — Read-only (SharedFiles) | ✅ Pass |
| File share — Modify (IT_Share) | ✅ Pass |
| Remote Desktop from CLIENT1 to DC01 | ✅ Pass |
| Event ID 4740 (account lockout) logged | ✅ Pass |
| IIS — hostname access via DC01.lab.local | ✅ Pass |
| System State backup — ntds.dit + SYSVOL present | ✅ Pass |
| All 7 PowerShell scripts executed | ✅ Pass |

---

## 🐛 Real Issues Solved

These are real troubleshooting scenarios encountered during the build — not lab book exercises:

- **GPO not applying:** DC01 had a duplicate DNS A record pointing to NAT IP (192.168.58.x). CLIENT1 was resolving DC01 to an unreachable IP. Fixed by deleting the stale DNS record.
- **Account Lockout not working:** Policy was applied at OU level. Windows only applies lockout policy from Default Domain Policy — not OUs. Moved policy to correct level.
- **Static IP reverting:** VMware IP conflict between host adapter (.1) and DC01 (.2). Fixed with PowerShell instead of GUI to force correct binding.
- **RDP denied for domain user:** Domain Controllers restrict RDP to Administrators by default. Granted access via GPO — User Rights Assignment in Default Domain Controllers Policy.

---

## 📁 Project Files

```
windows-server-2022-lab/
├── docs/
│   └── Windows_Server_2022_Complete_Documentation.pdf
├── scripts/
│   ├── ServiceCheck.ps1
│   ├── BulkCreateUsers.ps1
│   ├── ADUserReport.ps1
│   ├── UnlockAccounts.ps1
│   ├── InactiveUsers.ps1
│   ├── AddUsersToGroup.ps1
│   ├── UpdateDepartment.ps1
│   └── NewUsers.csv
└── README.md
```

---

## 🚀 How to Replicate This Lab

1. Install **VMware Workstation** — create VMnet1 (Host-only, 192.168.10.0/24) and VMnet8 (NAT)
2. Create DC01 VM: Windows Server 2022, 4GB RAM, dual NIC (VMnet1 + VMnet8)
3. Create CLIENT1 VM: Windows 10 Pro, 4GB RAM, single NIC (VMnet1)
4. On DC01: set static IP 192.168.10.2, install AD DS, promote to domain controller for `lab.local`
5. Follow documentation for DNS, DHCP, GPO, file shares, IIS, backup, and PowerShell scripts

---

## 📚 Windows Server Topics Demonstrated

`AD DS` `DNS` `DHCP` `Group Policy (GPO)` `Password Policy` `Account Lockout` `File Shares` `NTFS Permissions` `Share Permissions` `RDP/RDS` `Audit Policy` `Event Viewer` `IIS` `System State Backup` `PowerShell Automation` `VMware Networking`

---

## 👨‍💻 Author

**Vasanth Kumar**
BCA — Vivekananda Institute of Management, Bengaluru, Karnataka
📧 vasanthkumarvk2855@gmail.com
🔗 [LinkedIn](https://linkedin.com/in/vasanth-kumar-vk55) | [GitHub](https://github.com/vasanth-kumar-vk)

---

*This project is part of a hands-on IT infrastructure portfolio built to demonstrate entry-level Windows Server / System Administrator skills.*
