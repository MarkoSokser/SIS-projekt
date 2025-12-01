# Windows Server 2019 Domain Controller Setup – Detailed Guide

This document covers all steps performed to set up Windows Server 2019 as a Domain Controller, including:
- VM setup in VirtualBox
- Network configuration
- Vulnerabilities and why they were implemented
- Windows Server 2019 Core installation
- Configuration via sconfig
- Active Directory domain setup
- User creation
- Validation and snapshots
- All problems encountered and solutions

---

## 1. Creating Windows Server 2019 VM

### 1.1. Why This Server?

- This is the **Domain Controller (DC)** – the heart of an enterprise network.
- Handles authentication, DNS, security policies, and user management.
- Red Team attacks the DC, Blue Team defends it.

### 1.2. VM Specifications

```
OS: Windows Server 2019 Core (Eval)
CPU: 2 cores
RAM: 2–4 GB
Disk: 40GB VDI
Firmware: BIOS
Display: minimal (Core edition has no GUI)
```

**Reasoning:**
- Lightweight enough for laptop deployment
- Realistic for enterprise simulation

---

## 2. VirtualBox Network Configuration

### 2.1. Adapter 1 — Internal Network "TechNovaNet"

**Purpose:** Production network where DC + Workstations reside.

```
Attached to: Internal Network
Name: TechNovaNet
Promiscuous Mode: Allow All
Connected: ✔
```

![Network Adapter Configuration](./images/pfSense/network-adapters/Internal.png)
*Figure 9: Internal Network adapter configuration for Domain Controller*

**Why?**
- This is your enterprise network simulated in VirtualBox.
- All servers must be connected exclusively through it.


## 3. Windows Server 2019 Core Installation

### 3.1. Boot from ISO

Started VM, enabled boot from ISO disk.

### 3.2. Unattended Installation

VirtualBox automatically configured:
- Local user: `vboxuser`
- Password: WinServer2019

**Purpose:** Quick automated installation.

### 3.3. Removing ISO

After installation, removed:
- Optical disk (ISO)

**Why?**
- If ISO isn't removed, server might boot into setup again.

---

## 4. Initial Startup and Configuration

After booting into cmd screen, launched:

```cmd
sconfig
```

This is the official Microsoft configuration menu.

![sconfig Menu](./images/WinServer2019/conf_menu/conf_menu.png)
*Figure 10: Server Configuration (sconfig) main menu*

---

## 5. Network Configuration (Critical Step)

Selected option:
```
8) Network settings
```

Windows Server had APIPA address (`169.254.x.x`) because it didn't receive DHCP.

### 5.1. Set Static IP Address

```
IP: 10.10.0.10
Mask: 255.255.255.0
Gateway: 10.10.0.1
```

![Static IP Configuration](./images/WinServer2019/ipconf/IP-config.png)
*Figure 11: Static IP configuration for Domain Controller*

**Why?**
- Standard IP scheme for enterprise networks.
- Gateway is pfSense.

### 5.2. Set DNS

```
DNS1: 10.10.0.10 (the DC itself)
```

**Why must DC have itself as DNS?**
- AD DNS zones reside on the Domain Controller.
- All AD functionality depends on DNS.

### 5.3. Reboot

After changes – server rebooted.

---

## 6. Installing Active Directory Domain Services

Launched PowerShell.

### Adding AD Role:

```powershell
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
```

### Creating New Forest:

```powershell
Install-ADDSForest -DomainName "technova.local"
```

Server rebooted again.

**Why "technova.local"?**
- Internal domain name – realistic and appropriate for the project.


---

## 7. Implementing Weak Passwords (+ Why It's Important)

As part of realistic simulation, we implemented weaknesses:

### Weak Passwords:
- `Administrator1209!!`
- `BlueTeam2025!`
- `RedTeam2025!`

**Why?**
- Red Team will use password spraying / brute-force / Pass-the-Hash.
- MITRE ATT&CK includes techniques targeting weak passwords.
- In real environments, passwords are often weak.

**Security Note:**
These weaknesses are **intentional** for training purposes and map to:
- **T1078** (Valid Accounts)
- **T1110** (Brute Force)
- **T1003** (Credential Dumping)

---

## 8. Creating Users

Created users:
- `admin_lab` (Domain Admins)
- `blue_team`
- `red_team`

### PowerShell Commands:

```powershell
New-ADUser -Name "Admin Lab" -SamAccountName "admin_lab" `
  -AccountPassword (ConvertTo-SecureString "Administrator1209!!" -AsPlainText -Force) `
  -Enabled $true

New-ADUser -Name "Blue Team" -SamAccountName "blue_team" `
  -AccountPassword (ConvertTo-SecureString "BlueTeam2025!" -AsPlainText -Force) `
  -Enabled $true

New-ADUser -Name "Red Team" -SamAccountName "red_team" `
  -AccountPassword (ConvertTo-SecureString "RedTeam2025!" -AsPlainText -Force) `
  -Enabled $true
```

### Adding admin_lab to Domain Admins:

```powershell
Add-ADGroupMember -Identity "Domain Admins" -Members "admin_lab"
```

**Why?**
- `admin_lab` → used for AD administration
- `blue_team` → used for SIEM/EDR testing
- `red_team` → used for attack techniques

![User Creation](./images/WinServer2019/users/Users.png)
![User Creation](./images/WinServer2019/users/User-admin.png)
*Figure 12: Active Directory users created via PowerShell*

---

## 9. Problems and Solutions

###  Problem: "Account already exists"

Attempted to create user `admin` which already exists on DC.

**Solution:** Used `admin_lab` instead of `admin`.

---

###  Problem: DC cannot ping pfSense

**Reason:** pfSense VM was not running.

**Solution:** Start pfSense → ping works.

```cmd
ping 10.10.0.1
```

**Result:**  Reply from 10.10.0.1

---

## 10. Validation (Verifying Everything Works)

###  Ping pfSense LAN:

```cmd
ping 10.10.0.1
```

**Result:** Successful

---

###  DNS Works:

```cmd
nslookup technova.local
ipconfig /all
```

![DNS Validation](./images/WinServer2019/DNS_validation/nslookup.png)
![DNS Validation](./images/WinServer2019/DNS_validation/ipconfig.png)
*Figure 13: DNS resolution verification*

**Result:** DNS resolves correctly to `10.10.0.10`

---

### AD Structure Stable:

Server successfully rebooted after domain creation.

---

## 11. Snapshot (Golden State)

### Created Snapshot:

```
DC-SERVER – clean AD + users
```

**Why?**
- Return to initial state
- Repeat CALDERA campaigns
- Team work distribution

![Snapshot Creation](./images/WinServer2019/snap/Snapshoot.png)
*Figure 14: VirtualBox snapshot of clean Domain Controller state*

---

## 12. Current Infrastructure State

The network now looks like this:

```
pfSense (10.10.0.1)
   |
   +--- Windows Server 2019 DC (technova.local) – 10.10.0.10
```
