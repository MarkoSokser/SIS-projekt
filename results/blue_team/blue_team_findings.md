# Blue Team Findings

This document summarizes the key findings from Blue Team operations during the TechNova cyber-range exercise.

## Table of Contents

- [Phase 0: Pre-Attack Monitoring Setup](#phase-0-pre-attack-monitoring-setup)
- [Phase 3: Post-Attack Remediation](#phase-3-post-attack-remediation)
- [Verification Results](#verification-results)
- [Lessons Learned](#lessons-learned)

---

## Phase 0: Pre-Attack Monitoring Setup

### Detection Rules Deployed

| Rule ID | Description | MITRE Technique |
|---------|-------------|-----------------|
| 100001 | SSH brute force attack (5+ failed attempts in 60s) | T1110 |
| 100002 | SSH login success after multiple failures | T1110 |
| 100011 | Sudo executed without password (NOPASSWD) | T1548.003 |
| 100012 | Sudo used to access sensitive system files | T1003 |
| 100020 | PowerShell process execution | T1059.001 |
| 100021 | PowerShell making network connection | T1071 |
| 100023 | PowerShell web request detected | T1041 |
| 100040 | Critical system file modified | T1222 |
| 100041 | Cron job configuration modified | T1053.003 |
| 100050 | Multiple RDP login failures | T1021.001 |
| 100060 | pfSense firewall blocked traffic | Firewall |

### Intentional Vulnerabilities Documented

| System | Vulnerability | Purpose |
|--------|---------------|---------|
| Linux Server (10.10.0.51) | `webadmin` weak password | SSH Brute Force scenario |
| Linux Server (10.10.0.51) | `NOPASSWD` in sudoers | Privilege Escalation scenario |
| Linux Server (10.10.0.51) | Hardcoded credentials in `/tmp/db_config.py` | Credential Hunting scenario |
| Windows Workstation (10.10.0.50) | Firewall disabled | Lateral Movement scenario |

---

## Phase 3: Post-Attack Remediation

### Scenario 1: SSH Brute Force

**Vulnerability:** Weak/missing password for `webadmin` user  
**Attack Result:** Red Team gained access immediately without brute force

**Remediation Applied:**
- Changed password to strong passphrase: `W3bAdm1nS3cure2025!`
- Hardened `/etc/ssh/sshd_config`:
  - `PermitRootLogin no`
  - `MaxAuthTries 3`
  - `PermitEmptyPasswords no`
  - `LoginGraceTime 60`

**Verification:** SSH login now requires correct password; brute force blocked after 3 attempts

---

### Scenario 2: C2 Exfiltration

**Vulnerability:** No outbound traffic filtering to C2 server  
**Attack Result:** Red Team exfiltrated data via HTTP POST to 10.10.0.53:8888

**Remediation Applied:**
- Created pfSense firewall rule:
  - Action: Block
  - Source: 10.10.0.50 (Windows Workstation)
  - Destination: 10.10.0.53 (C2 Server)
  - Port: 8888 TCP
  - Logging: Enabled

**Verification:** Connection timeout when attempting exfiltration; Wazuh shows "Firewall Block" event

---

### Scenario 3: Privilege Escalation

**Vulnerability:** `NOPASSWD` entry in `/etc/sudoers` + hardcoded credentials  
**Attack Result:** Red Team read `/etc/shadow` without password and found `admin_lab` credentials

**Remediation Applied:**
- Removed `NOPASSWD` entry using `visudo`
- Deleted `/tmp/db_config.py` containing hardcoded credentials
- Rotated `admin_lab` password to `Adm1nL4bS3cure2025!`

**Verification:** `sudo` now prompts for password; credential file returns "No such file"

---

### Scenario 4: Lateral Movement

**Vulnerability:** Windows Firewall disabled  
**Attack Result:** Red Team used PsExec via SMB (port 445) to deploy CALDERA agent

**Remediation Applied:**
- Enabled Windows Defender Firewall on all profiles
- Created inbound block rules:
  - Block all from 10.10.0.53 (CALDERA C2)
  - Block SMB (445) from 10.10.0.51
  - Block RPC (135) from 10.10.0.51
  - Block WMI (5985/5986) from 10.10.0.51, 10.10.0.53
- Removed `splunkd.exe` CALDERA agent

**Verification:** PsExec connection returns "No route to host"; C2 agent cannot reconnect

---

### Scenario 5: Linux Server Cleanup

**Artifacts Removed:**
- `/home/vbubuntu/psexec.py`
- `/home/vbubuntu/wmiexec.py`
- `/home/vbubuntu/smbclient.py`
- `/home/vbubuntu/splunkd` (CALDERA agent)

**Verification:** `ls -la /home/vbubuntu/*.py` returns "No such file"; no active C2 connections

---

## Verification Results

| Scenario | Vulnerability | Remediation | Phase 2 Re-Test |
|----------|---------------|-------------|-----------------|
| SSH Brute Force | Weak Password | Password + SSH Hardening | BLOCKED |
| C2 Exfiltration | Open Outbound | pfSense Block Rule | BLOCKED |
| Privilege Escalation | NOPASSWD Sudo | Entry Removed | BLOCKED |
| Lateral Movement | Firewall Disabled | Firewall + Block Rules | BLOCKED |
| Artifact Cleanup | Attack Tools | Removed | CLEAN |

**Overall Status:** Environment secured against Phase 1 attack techniques

---

## Lessons Learned

1. **Password Policies:** Even internal lab users need strong passwords; weak credentials are the easiest entry point
2. **Least Privilege:** NOPASSWD sudo entries should never exist in production environments
3. **Credential Management:** Never store credentials in plaintext files; use secrets management
4. **Host Firewalls:** Windows Firewall provides critical defense against lateral movement
5. **Network Segmentation:** pfSense rules can effectively contain compromised hosts
6. **Detection Coverage:** Wazuh custom rules successfully detected all attack phases

---

## Related Documentation

- [Blue Team Pre-Attack Setup](../implementation/src/blue-team-engineer/blue-team-pre-attack-setup.md)
- [Blue Team Post-Attack Remediation](../implementation/src/blue-team-engineer/blue-team-post-attack-remediation.md)
- [Planned Attack Scenarios](../implementation/src/blue-team-engineer/planned-attacks.md)
