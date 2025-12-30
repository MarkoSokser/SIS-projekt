# TechNova Lab – Credentials and Access Control Guide

This document contains all credentials, access rules, and role-based permissions for the complete TechNova cybersecurity lab infrastructure.

** IMPORTANT SECURITY NOTE:**
All passwords in this document are **intentionally weak** for educational and training purposes. These credentials are designed to facilitate Red Team vs Blue Team exercises and should **NEVER** be used in production environments.

---

## Table of Contents

1. [Lab Overview](#1-lab-overview)
2. [Network Architecture](#2-network-architecture)
3. [pfSense Firewall Credentials](#3-pfsense-firewall-credentials)
4. [Windows Server 2019 Domain Controller](#4-windows-server-2019-domain-controller)
5. [Active Directory Domain Accounts](#5-active-directory-domain-accounts)
6. [Windows 10/11 Workstation](#6-windows-1011-workstation)
7. [Ubuntu Vulnerable Server](#7-ubuntu-vulnerable-server)
8. [CALDERA Server](#8-caldera-server)
9. [Wazuh SIEM Server](#9-wazuh-siem-server)
10. [Role-Based Access Matrix](#10-role-based-access-matrix)
11. [Red Team Access Guide](#11-red-team-access-guide)
12. [Blue Team Access Guide](#12-blue-team-access-guide)
13. [Threat Hunter Access Guide](#13-threat-hunter-access-guide)
14. [Infrastructure Engineer Access](#14-infrastructure-engineer-access)
15. [Quick Reference Card](#15-quick-reference-card)

---

## 1. Lab Overview

### Project: TechNova Cybersecurity Lab
**Purpose:** MITRE CALDERA adversary simulation environment for Red Team vs Blue Team exercises

### Infrastructure Components:
- **pfSense Firewall** – Network gateway and security
- **Windows Server 2019 DC** – Active Directory domain controller (technova.local)
- **Windows 10/11 Workstation** – Domain-joined client
- **Ubuntu Server** – Vulnerable target with web services
- **CALDERA Server** – Adversary emulation framework
- **Wazuh SIEM** – Security monitoring and detection

### Network: TechNovaNet
**Subnet:** 10.10.0.0/24

---

## 2. Network Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Internet (NAT)                           │
└────────────────────────┬────────────────────────────────────┘
                         │
                    ┌────┴────┐
                    │ pfSense │ 10.0.2.15 (WAN)
                    │ Gateway │ 10.10.0.1 (LAN)
                    └────┬────┘ 192.168.56.2 (Admin)
                         │
        ┌────────────────┴────────────────┐
        │     TechNovaNet (10.10.0.0/24)  │
        └─────────────────────────────────┘
                         │
        ┌────────┬───────┼──────┬──────────┬──────────┐
        │        │       │      │          │          │
    ┌───┴───┐ ┌─┴──┐ ┌──┴──┐ ┌─┴───┐  ┌───┴────┐ ┌───┴────┐
    │  DC   │ │Win │ │Linux│ │CALDR│  │ Wazuh  │ │ Future │
    │.10    │ │.50 │ │.51  │ │.60  │  │  .70   │ │  VMs   │
    └───────┘ └────┘ └─────┘ └─────┘  └────────┘ └────────┘
```

### IP Address Allocation:

| Device | IP Address | Role |
|--------|------------|------|
| pfSense LAN | 10.10.0.1 | Gateway |
| pfSense Admin | 192.168.56.2 | Management |
| Windows Server DC | 10.10.0.10 | Domain Controller |
| Windows Workstation | 10.10.0.50 (DHCP) | Domain client |
| Ubuntu Server | 10.10.0.51 | Vulnerable target |
| CALDERA Server | 10.10.0.60 | Adversary emulation |
| Wazuh SIEM | 10.10.0.70 | Security monitoring |
| DHCP Pool | 10.10.0.50-200 | Dynamic assignments |

---

## 3. pfSense Firewall Credentials

### Management Access:

**Web GUI URL:**
```
https://192.168.56.2
(Host-Only network - accessible only from Windows host)
```

**Console Access:**
- Direct access via VirtualBox VM console

### Credentials:

```
Username: admin
Password: Sig$$2025
```

**Note:** Password was changed from default `pfsense` during initial setup for security.

### Network Interfaces:

**WAN (Adapter 1 - NAT):**
```
IP: 10.0.2.15 (DHCP from VirtualBox NAT)
Purpose: Internet access for updates
```

**LAN (Adapter 2 - Internal Network):**
```
IP: 10.10.0.1/24
Network: TechNovaNet
DHCP Range: 10.10.0.50 - 10.10.0.200
DNS: 10.10.0.10 (Domain Controller)
Purpose: Lab network segment
```

**OPT1/ADMIN (Adapter 3 - Host-Only):**
```
IP: 192.168.56.2/24
Purpose: Secure management access
Firewall Rule: Allow all (for admin access)
```

### Important Services:

**DHCP Server (LAN):**
- Status: Enabled
- Range: 10.10.0.50 - 10.10.0.200
- Gateway: 10.10.0.1
- DNS: 10.10.0.10

**DNS Resolver:**
- Status: Enabled
- Forwards to: 1.1.1.1, 8.8.8.8
- Domain override: technova.local → 10.10.0.10

---

## 4. Windows Server 2019 Domain Controller

### VM Access:

**VirtualBox Console:**
- Direct console access (no RDP configured)

### Local Administrator:

```
Username: vboxuser
Password: WinServer2019
```

**Note:** This is the local administrator created during Windows installation.

### Domain Administrator:

```
Domain: TECHNOVA
Username: Administrator
Password: (default Windows Server password)
Full: TECHNOVA\Administrator
```

### Network Configuration:

```
IP Address: 10.10.0.10
Subnet Mask: 255.255.255.0
Gateway: 10.10.0.1
DNS: 10.10.0.10 (points to itself)
```

### Active Directory Forest:

```
Domain Name: technova.local
Domain NetBIOS: TECHNOVA
Forest Functional Level: Windows Server 2016
Domain Functional Level: Windows Server 2016
```

### Server Configuration Tool:

```cmd
sconfig
```

Use this menu for:
- Network settings
- Computer name
- Windows updates
- Remote management
- Power options

---

## 5. Active Directory Domain Accounts

### Domain: technova.local (TECHNOVA)

All domain accounts follow this format:
```
TECHNOVA\username
```

---

### 5.1. Administrator Account (Infrastructure Management)

```
Display Name: Admin Lab
Username: admin_lab
Password: Administrator1209!!
Domain: TECHNOVA\admin_lab

Group Memberships:
- Domain Admins
- Domain Users

Permissions:
- Full administrative access to domain
- Can manage all domain resources
- Can create/delete users and groups
- Can modify group policies
```

**Purpose:** Infrastructure administration and domain management

**Usage:**
- Joining computers to domain
- Managing Active Directory
- Installing domain services
- Group Policy management

---

### 5.2. Blue Team Account (Defensive Operations)

```
Display Name: Blue Team
Username: blue_team
Password: BlueTeam2025!
Domain: TECHNOVA\blue_team

Group Memberships:
- Domain Users

Permissions:
- Standard user privileges
- Read access to security logs (via delegation)
- Can log into workstations
- No administrative rights
```

**Purpose:** Defensive security operations and monitoring

**Usage:**
- SIEM monitoring (Wazuh)
- Log analysis
- Security event investigation
- Incident response activities

**Intentional Weakness:** Password follows common pattern (Role + Year + !)

**MITRE ATT&CK Mapping:**
- T1078 - Valid Accounts
- T1110.003 - Password Spraying target

---

### 5.3. Red Team Account (Offensive Operations)

```
Display Name: Red Team
Username: red_team
Password: RedTeam2025!
Domain: TECHNOVA\red_team

Group Memberships:
- Domain Users

Permissions:
- Standard user privileges
- Can log into workstations
- No administrative rights
```

**Purpose:** Simulated adversary account for attack scenarios

**Usage:**
- Initial access simulations
- Lateral movement exercises
- Privilege escalation attempts
- Credential harvesting targets

**Intentional Weakness:** Password follows common pattern (Role + Year + !)

**MITRE ATT&CK Mapping:**
- T1078 - Valid Accounts
- T1110.001 - Brute Force: Password Guessing
- T1110.003 - Password Spraying

---

## 6. Windows 10/11 Workstation

### Local Account (Pre-Domain Join):

```
Username: vboxuser
Password: WinServer2019
```

**Note:** Used only during initial setup before domain join

### Domain Status:

```
Computer Name: WIN-CLIENT
Domain: technova.local
Full Computer Name: WIN-CLIENT.technova.local
```

### Network Configuration:

```
IP Address: 10.10.0.50 (DHCP from pfSense)
Subnet Mask: 255.255.255.0
Gateway: 10.10.0.1
DNS: 10.10.0.10
```

### Domain Login Access:

**Any domain account can log in:**

```
TECHNOVA\admin_lab (Administrator1209!!)
TECHNOVA\blue_team (BlueTeam2025!)
TECHNOVA\red_team (RedTeam2025!)
```

**Login Format:**
- Click "Other user" on login screen
- Enter: `TECHNOVA\username`
- Enter password

### Installed Agents:

**Wazuh Agent:**
```
Agent ID: 001
Agent Name: WIN-CLIENT
SIEM Server: 10.10.0.70
Status: Active
```

**CALDERA Sandcat Agent:**
```
Agent Location: C:\Users\Public\splunkd.exe
Server: http://10.10.0.60:8888
Group: red
```

---

## 7. Ubuntu Vulnerable Server

### SSH Access:

```
Hostname: ubuntu-srv
IP Address: 10.10.0.51
SSH Port: 22

Primary User:
Username: vbubuntu
Password: UbuntuServer2025
```

**SSH Connection:**
```bash
ssh vbubuntu@10.10.0.51
```

---

### Vulnerable Account (Intentional Weakness):

```
Username: webadmin
Password: Webadmin123!
Sudo Access: NOPASSWD for ALL commands
```

**Sudo Configuration:**
```
webadmin ALL=(ALL) NOPASSWD:ALL
```

**Security Impact:**
- Immediate root access after compromise
- No password required for sudo
- Perfect privilege escalation target

**MITRE ATT&CK Mapping:**
- T1078 - Valid Accounts
- T1548.003 - Sudo and Sudo Caching
- T1110.001 - Brute Force: Password Guessing

---

### Installed Services:

**Apache2 Web Server:**
```
Service: apache2
Port: 80
Status: active (running)
Web Root: /var/www/html
```

**SSH Server:**
```
Service: ssh
Port: 22
Status: active (running)
Authentication: Password enabled
```

---

### Intentional Vulnerabilities:

**1. Weak User Account:**
- User: `webadmin`
- Password: `Webadmin123!`
- Sudo: NOPASSWD:ALL

**2. Cron Job as Root:**
```
Frequency: Every minute
Script: /usr/local/bin/backup.sh
User: root
```

**3. Directory Listing Enabled:**
- Apache shows directory contents
- Information disclosure

**4. Password Authentication:**
- SSH accepts passwords
- Vulnerable to brute force

---

### Network Configuration:

```
IP Address: 10.10.0.51/24
Gateway: 10.10.0.1
DNS: 10.10.0.10, 8.8.8.8
Domain Search: technova.local
```

---

### Installed Agents:

**Wazuh Agent:**
```
Agent ID: 002
Agent Name: ubuntu-srv
SIEM Server: 10.10.0.70
Config: /var/ossec/etc/ossec.conf
Status: Active
```

**CALDERA Sandcat Agent:**
```
Agent Location: ~/splunkd
Server: http://10.10.0.60:8888
Group: red
```

---

## 8. CALDERA Server

### VM Access:

```
Hostname: CALDERA-SRV
IP Address: 10.10.0.60
OS: Ubuntu 24.04 LTS
```

### SSH Access:

```
Username: ubcaldera
Password: (simple lab password)
```

**SSH Connection:**
```bash
ssh ubcaldera@10.10.0.60
```

---

### CALDERA Operator User:

```
System User: operator
Password: (set during user creation)
Purpose: Running CALDERA services
```

**User was created with:**
```bash
sudo adduser operator
sudo usermod -aG sudo operator
```

---

### CALDERA Web Interface:

**URL:**
```
http://10.10.0.60:8888
```

**Red Team Operator Credentials:**
```
Username: red
Password: admin
Role: Red Team operator
```

**Blue Team Operator Credentials:**
```
Username: blue
Password: admin
Role: Blue Team operator
```

**Administrator Credentials:**
```
Username: admin
Password: admin
Role: Full administrative access
```

---

### Starting CALDERA Server:

```bash
cd /home/ubcaldera/caldera
source venv/bin/activate
python3 server.py --insecure
```

**Note:** Must activate Python virtual environment before starting

---

### Network Configuration:

```
IP Address: 10.10.0.60/24
Gateway: 10.10.0.1
DNS: 10.10.0.10, 8.8.8.8
Domain Search: technova.local
```

---

### Agent Communication:

**Sandcat Agents:**
- Linux agents connect to: `http://10.10.0.60:8888`
- Windows agents connect to: `http://10.10.0.60:8888`
- Default group: `red`

---

## 9. Wazuh SIEM Server

### VM Access:

```
Hostname: WAZUH-SIEM
IP Address: 10.10.0.70
OS: Ubuntu Server 24.04 LTS
```

### SSH Access:

```
Username: vbwazuh
Password: Wazuh!$1
```

**SSH Connection:**
```bash
ssh vbwazuh@10.10.0.70
```

---

### Wazuh Manager:

**Service Control:**
```bash
sudo /var/ossec/bin/wazuh-control status
sudo /var/ossec/bin/wazuh-control start
sudo /var/ossec/bin/wazuh-control stop
sudo /var/ossec/bin/wazuh-control restart
```

**Agent Management:**
```bash
sudo /var/ossec/bin/manage_agents
sudo /var/ossec/bin/agent_control -l
```

---

### Important Log Locations:

**Alert Logs (JSON):**
```
/var/ossec/logs/alerts/alerts.json
```

**Alert Logs (Text):**
```
/var/ossec/logs/alerts/alerts.log
```

**Manager Logs:**
```
/var/ossec/logs/ossec.log
```

**Real-time Monitoring:**
```bash
sudo tail -f /var/ossec/logs/alerts/alerts.log
sudo tail -f /var/ossec/logs/alerts/alerts.json
```

---

### Connected Agents:

**Agent 001 - Windows Workstation:**
```
ID: 001
Name: WIN-CLIENT
IP: 10.10.0.50
Status: Active
```

**Agent 002 - Linux Server:**
```
ID: 002
Name: ubuntu-srv
IP: 10.10.0.51
Status: Active
```

---

### Network Configuration:

```
IP Address: 10.10.0.70/24
Gateway: 10.10.0.1
DNS: 10.10.0.10
Domain Search: technova.local
```

---

## 10. Role-Based Access Matrix

| Resource | Red Team | Blue Team | Threat Hunter | Infrastructure Engineer |
|----------|----------|-----------|---------------|------------------------|
| **pfSense GUI** | ❌ No | ❌ No | ✅ Read-only | ✅ Full Admin |
| **Domain Controller** | ❌ No | ❌ No | ✅ Read-only | ✅ Full Admin |
| **Windows Workstation** | ✅ Login (red_team) | ✅ Login (blue_team) | ✅ Login (any) | ✅ Full Admin |
| **Linux Server** | ✅ SSH (all users) | ✅ SSH (vbubuntu) | ✅ SSH (all users) | ✅ Full Admin |
| **CALDERA GUI** | ✅ Operator (red) | ✅ Operator (blue) | ✅ View-only | ✅ Admin |
| **Wazuh Logs** | ❌ No | ✅ Full Access | ✅ Full Access | ✅ Full Access |
| **Wazuh SSH** | ❌ No | ✅ SSH Access | ✅ SSH Access | ✅ Full Admin |

---

## 11. Red Team Access Guide

### Your Mission:
Simulate real-world adversary attacks using MITRE ATT&CK techniques via CALDERA framework.

---

### 11.1. Initial Access

**Domain Account:**
```
Username: TECHNOVA\red_team
Password: RedTeam2025!
```

**Windows Workstation Login:**
1. Boot WIN-CLIENT VM
2. Click "Other user"
3. Enter: `TECHNOVA\red_team`
4. Password: `RedTeam2025!`

---

### 11.2. CALDERA Operator Access

**Web Interface:**
```
URL: http://10.10.0.60:8888
Username: red
Password: admin
```

**Capabilities:**
- Deploy Sandcat agents
- Create adversary operations
- Execute MITRE ATT&CK techniques
- View operation results
- Manage agent groups

---

### 11.3. Target Systems

**Primary Targets:**

**Windows Workstation (10.10.0.50):**
- Sandcat agent installed: `C:\Users\Public\splunkd.exe`
- Domain-joined: technova.local
- User accounts: admin_lab, blue_team, red_team

**Linux Server (10.10.0.51):**
- SSH access: `ssh vbubuntu@10.10.0.51` (password: UbuntuServer2025)
- Vulnerable user: `webadmin` (password: Webadmin123!)
- Sandcat agent: `~/splunkd`
- Services: Apache (port 80), SSH (port 22)

---

### 11.4. Attack Techniques Available

**Discovery:**
- Network enumeration
- User enumeration
- Process discovery
- System information gathering

**Credential Access:**
- Password spraying (T1110.003)
- Brute force (T1110.001)
- Credential dumping (T1003)

**Privilege Escalation:**
- Sudo abuse (T1548.003) on Linux
- Valid accounts (T1078)

**Persistence:**
- Scheduled tasks (T1053.003)
- Cron jobs

**Lateral Movement:**
- RDP
- SSH
- Pass-the-hash

---

### 11.5. Red Team Workflow

**Step 1: Deploy Agents**
- Agents already installed on WIN-CLIENT and ubuntu-srv
- Verify in CALDERA → Agents

**Step 2: Create Operation**
- CALDERA → Campaigns → New Operation
- Select adversary profile (e.g., Hunter, Discovery)
- Select agent group: red

**Step 3: Execute**
- Start operation
- Monitor execution in real-time

**Step 4: Review Results**
- Check command outputs
- Verify successful techniques
- Document findings

---

### 11.6. Credentials for Testing

**Domain Credentials (for spraying):**
```
admin_lab : Administrator1209!!
blue_team : BlueTeam2025!
red_team : RedTeam2025!
```

**Linux Credentials (for brute force):**
```
vbubuntu : UbuntuServer2025
webadmin : Webadmin123!
```

---

## 12. Blue Team Access Guide

### Your Mission:
Detect, analyze, and respond to Red Team attacks using Wazuh SIEM and log analysis.

---

### 12.1. Initial Access

**Domain Account:**
```
Username: TECHNOVA\blue_team
Password: BlueTeam2025!
```

**Windows Workstation Login:**
1. Boot WIN-CLIENT VM
2. Click "Other user"
3. Enter: `TECHNOVA\blue_team`
4. Password: `BlueTeam2025!`

---

### 12.2. Wazuh SIEM Access

**SSH to SIEM Server:**
```bash
ssh vbwazuh@10.10.0.70
Password: Wazuh!$1
```

**Monitor Alerts in Real-Time:**
```bash
sudo tail -f /var/ossec/logs/alerts/alerts.log
```

**Monitor High-Severity Alerts:**
```bash
sudo tail -f /var/ossec/logs/alerts/alerts.log | grep -E 'rule.level: (7|8|9|10|11|12|13|14|15)'
```

**View JSON Format:**
```bash
sudo tail -f /var/ossec/logs/alerts/alerts.json
```

---

### 12.3. CALDERA Blue Team Interface

**Web Access:**
```
URL: http://10.10.0.60:8888
Username: blue
Password: admin
```

**Capabilities:**
- View Red Team operations (read-only)
- Understand attack chains
- Analyze TTPs used
- Correlate with SIEM alerts

---

### 12.4. Detection Priorities

**High-Priority Events:**

1. **Failed Login Attempts:**
   - Multiple failures → brute force (T1110)
   - Windows Event 4625
   - Linux auth.log failures

2. **Privilege Escalation:**
   - Sudo usage by webadmin
   - New administrative group members
   - Process execution as SYSTEM/root

3. **Lateral Movement:**
   - New SSH connections
   - RDP sessions
   - Process injection

4. **Credential Access:**
   - LSASS access
   - SAM/shadow file reads
   - Mimikatz indicators

5. **Persistence:**
   - New scheduled tasks
   - Cron job modifications
   - Registry autoruns

---

### 12.5. Investigation Workflow

**Step 1: Alert Triage**
- Monitor Wazuh alerts
- Identify high-severity events
- Correlate multiple alerts

**Step 2: Investigation**
- Check source IP/hostname
- Identify user account
- Review command execution

**Step 3: Containment**
- Document findings
- Escalate if needed
- (In this lab: observe only)

**Step 4: Reporting**
- Document attack chain
- Map to MITRE ATT&CK
- Create incident report

---

### 12.6. Key Log Locations

**Wazuh Alerts:**
```
/var/ossec/logs/alerts/alerts.log
/var/ossec/logs/alerts/alerts.json
```

**Windows Event Logs (on WIN-CLIENT):**
```
Security Events: Event Viewer → Security
Application Events: Event Viewer → Application
```

**Linux Logs (on ubuntu-srv):**
```
/var/log/auth.log (SSH, sudo)
/var/log/syslog (system events)
/var/log/apache2/access.log (web access)
```

---

## 13. Threat Hunter Access Guide

### Your Mission:
Proactively hunt for threats, analyze attack patterns, and identify indicators of compromise.

---

### 13.1. Access Credentials

**Domain Account:**
```
Username: TECHNOVA\admin_lab
Password: Administrator1209!!
```

**Reason for Admin Access:**
- Need to access all systems
- Review security configurations
- Analyze system artifacts
- Investigate suspicious activity

---

### 13.2. Full System Access

**pfSense (Read-Only Recommended):**
```
URL: https://192.168.56.2
Username: admin
Password: pfsense
Purpose: Network traffic analysis, firewall logs
```

**Windows Server DC:**
```
Console access via VirtualBox
Username: TECHNOVA\admin_lab
Password: Administrator1209!!
Purpose: AD logs, authentication events
```

**Windows Workstation:**
```
Username: TECHNOVA\admin_lab
Password: Administrator1209!!
Purpose: Endpoint forensics, process analysis
```

**Linux Server:**
```
ssh vbubuntu@10.10.0.51
Password: UbuntuServer2025
Sudo: sudo su - (if needed)
Purpose: Log analysis, file integrity checks
```

**CALDERA Server:**
```
URL: http://10.10.0.60:8888
Username: admin
Password: admin
Purpose: Understanding attack operations
```

**Wazuh SIEM:**
```
ssh vbwazuh@10.10.0.70
Password: Wazuh!$1
Purpose: SIEM analysis, alert correlation
```

---

### 13.3. Threat Hunting Methodology

**Hypothesis-Driven Hunting:**

1. **Formulate Hypothesis**
   - "Attacker may have gained persistence via cron jobs"
   - "Credentials may have been dumped from LSASS"

2. **Collect Data**
   - Check cron configurations
   - Review Wazuh alerts
   - Analyze process creation logs

3. **Analyze Indicators**
   - Look for anomalies
   - Correlate events
   - Timeline reconstruction

4. **Document Findings**
   - Map to MITRE ATT&CK
   - Create detection rules
   - Share with Blue Team

---

### 13.4. Hunting Focus Areas

**Credential Abuse:**
```bash
# Check failed logins
sudo grep "Failed password" /var/log/auth.log

# Check sudo usage
sudo grep "sudo:" /var/log/auth.log

# Review Wazuh authentication alerts
sudo grep "authentication" /var/ossec/logs/alerts/alerts.log
```

**Persistence Mechanisms:**
```bash
# Linux cron jobs
sudo crontab -l
sudo cat /etc/crontab

# Windows scheduled tasks (on WIN-CLIENT)
schtasks /query /fo LIST /v
```

**Process Execution:**
```bash
# Wazuh process monitoring
sudo grep "execve" /var/ossec/logs/alerts/alerts.log

# Suspicious command patterns
sudo grep -E "(wget|curl|powershell|certutil)" /var/ossec/logs/alerts/alerts.log
```

**Network Connections:**
```bash
# Active connections on Linux
netstat -tulpn

# Wazuh network alerts
sudo grep "connection" /var/ossec/logs/alerts/alerts.log
```

---

### 13.5. MITRE ATT&CK Mapping

Map all findings to MITRE ATT&CK framework:

**Example Mapping:**
- Failed SSH logins → T1110.001 (Brute Force: Password Guessing)
- Sudo NOPASSWD abuse → T1548.003 (Sudo and Sudo Caching)
- Cron job execution → T1053.003 (Scheduled Task/Job: Cron)
- Apache exploitation → T1190 (Exploit Public-Facing Application)

---

## 14. Infrastructure Engineer Access

### Your Mission:
Maintain, configure, and troubleshoot all infrastructure components.

---

### 14.1. Full Administrative Access

**All systems require full admin access for:**
- VM management (VirtualBox)
- Network configuration
- Service troubleshooting
- Snapshot management
- Security updates

---

### 14.2. System Credentials

**pfSense:**
```
Console: Direct VirtualBox access
GUI: https://192.168.56.2
Username: admin
Password: pfsense
```

**Windows Server DC:**
```
Local Admin: vboxuser / WinServer2019
Domain Admin: TECHNOVA\admin_lab / Administrator1209!!
```

**Windows Workstation:**
```
Local Admin: vboxuser / WinServer2019
Domain Login: TECHNOVA\admin_lab / Administrator1209!!
```

**Linux Server:**
```
SSH: vbubuntu@10.10.0.51 / UbuntuServer2025
Sudo: Full access
```

**CALDERA Server:**
```
SSH: ubcaldera@10.10.0.60
Web: admin / admin
```

**Wazuh SIEM:**
```
SSH: vbwazuh@10.10.0.70 / Wazuh!$1
Manager: sudo access
```

---

### 14.3. Management Tasks

**VirtualBox Snapshot Management:**
- Create snapshots before exercises
- Restore to clean state after attacks
- Maintain golden images

**Network Configuration:**
- pfSense firewall rules
- DHCP scope management
- DNS configuration

**Service Management:**
- Start/stop VMs
- Monitor resource usage
- Apply updates

**User Management:**
- Create/delete AD accounts
- Reset passwords
- Modify permissions

**Backup and Recovery:**
- Export VMs
- Document configurations
- Test restore procedures

---

## 15. Quick Reference Card

### Essential IPs

```
pfSense Admin:     192.168.56.2
DC:                10.10.0.10
Windows Client:    10.10.0.50
Linux Server:      10.10.0.51
CALDERA:           10.10.0.60
Wazuh SIEM:        10.10.0.70
```

---

### Domain Credentials

```
Admin:      TECHNOVA\admin_lab    / Administrator1209!!
Blue Team:  TECHNOVA\blue_team    / BlueTeam2025!
Red Team:   TECHNOVA\red_team     / RedTeam2025!
```

---

### CALDERA Access

```
URL:        http://10.10.0.60:8888
Red Ops:    red / admin
Blue Ops:   blue / admin
Admin:      admin / admin
```

---

### Wazuh SIEM

```
SSH:        ssh vbwazuh@10.10.0.70
Password:   Wazuh!$1
Alerts:     sudo tail -f /var/ossec/logs/alerts/alerts.log
```

---

### Linux Server

```
SSH:        ssh vbubuntu@10.10.0.51
Password:   UbuntuServer2025
Vulnerable: webadmin / Webadmin123!
```

---

## 16. Security Notices

### ⚠️ Intentional Vulnerabilities

This lab contains **intentional security weaknesses** for training:

1. **Weak Passwords:**
   - All passwords follow predictable patterns
   - Designed for password spraying exercises
   - NEVER use these patterns in production

2. **Excessive Privileges:**
   - webadmin has NOPASSWD sudo
   - admin_lab has Domain Admin rights
   - Root cron jobs execute scripts

3. **No Network Segmentation:**
   - All systems on single flat network
   - No firewall rules between systems
   - Lateral movement is trivial

4. **Password Authentication:**
   - SSH accepts passwords
   - RDP accepts passwords
   - No MFA/2FA implemented

5. **Service Exposure:**
   - Apache with directory listing
   - Weak SSH configuration
   - No fail2ban or rate limiting

### ✅ These Are Features, Not Bugs

All weaknesses are documented and mapped to MITRE ATT&CK techniques for educational purposes.

---

## 17. Troubleshooting Access Issues

### Cannot Access pfSense GUI

**Problem:** Cannot reach https://192.168.56.2

**Solutions:**
1. Verify pfSense VM is running
2. Check Host-Only adapter IP: `192.168.56.1`
3. Ping test: `ping 192.168.56.2`
4. Disable Windows Firewall temporarily
5. Check browser accepts self-signed certificate

---

### Domain Login Fails

**Problem:** "Domain not available" or "Trust relationship failed"

**Solutions:**
1. Verify DC VM is running
2. Check DNS points to 10.10.0.10
3. Test: `nslookup technova.local`
4. Verify network connectivity: `ping 10.10.0.10`
5. Check computer is joined to domain

---

### Cannot SSH to Linux Systems

**Problem:** Connection refused or timeout

**Solutions:**
1. Verify target VM is running
2. Check IP address: `ip a`
3. Verify SSH service: `systemctl status ssh`
4. Test from another Linux VM first
5. Check pfSense firewall rules

---

### Wazuh Agents Not Connecting

**Problem:** Agent shows "Never connected"

**Solutions:**
1. Verify Wazuh server IP in agent config
2. Check agent service status
3. Verify firewall allows port 1514/tcp
4. Restart agent service
5. Check manager logs: `/var/ossec/logs/ossec.log`

---

### CALDERA Agents Missing

**Problem:** Agents don't appear in CALDERA

**Solutions:**
1. Verify CALDERA server is running
2. Check agent process is running
3. Verify correct server URL in agent command
4. Check network connectivity to 10.10.0.60:8888
5. Review CALDERA server logs

---

## 18. Emergency Recovery

### Reset All Passwords

If credentials are lost or compromised:

**Windows DC:**
```
Boot to console
Option 3: Reset webConfigurator password
```

**Linux Systems:**
```
Boot to recovery mode
mount -o remount,rw /
passwd username
```

**Active Directory:**
```powershell
Set-ADAccountPassword -Identity username -Reset
```

---

### Restore from Snapshot

**VirtualBox Snapshots Available:**
- `pfSense – operational baseline`
- `DC-SERVER – clean AD + users`
- `WIN-CLIENT – clean domain join`
- `Ubuntu-Server – operational baseline`
- `CALDERA-SRV – operational baseline`
- `WAZUH-SIEM – operational baseline`

**Restore Process:**
1. VirtualBox → Select VM
2. Snapshots → Select snapshot
3. Restore → Yes

---

## 19. Contact and Support

### Lab Administrator:
**Infrastructure Engineer**

### Documentation:
- Full setup guides in `implementation/src/infrastructure-engineer/`
- Troubleshooting in each component guide
- Theory in `docs/theory.md`
- Project plan in `docs/plan.md`

---

## 20. Compliance and Ethics

### Lab Usage Agreement

✅ **Authorized Use:**
- Educational purposes only
- Cybersecurity training
- MITRE ATT&CK technique demonstration
- Security research within lab environment

❌ **Prohibited Use:**
- Attacking systems outside this lab
- Using techniques on production networks
- Sharing credentials externally
- Malicious activity

---

**Document Version:** 1.0  
**Last Updated:** November 30, 2025  
**Maintained By:** Infrastructure Engineer  
**Lab Name:** TechNova Cybersecurity Lab  
**Project:** MITRE CALDERA Adversary Simulation Environment
