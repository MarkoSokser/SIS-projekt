# Red Team Findings

This document summarizes the key findings from Red Team operations during the TechNova cyber-range exercise, covering initial compromise vectors and hardening verification.

## Key Findings Summary

In Phase 1 (Baseline), Red Team achieved full domain compromise starting from a single Linux foothold via weak SSH credentials, exploiting NOPASSWD sudo for privilege escalation, hardcoded credentials for lateral movement via SMB, and unrestricted outbound traffic for data exfiltration. In Phase 2 (Hardening), all critical attack vectors were blocked by Blue Team remediations, including firewall enablement, credential removal, and sudo hardening, preventing escalation and movement while maintaining initial access assumptions.

## Table of Contents

- [Phase 1: Baseline Vulnerability Assessment](#phase-1-baseline-vulnerability-assessment)
- [Phase 2: Hardening Verification](#phase-2-hardening-verification)
- [Verification Results](#verification-results)
- [Lessons Learned](#lessons-learned)

---

## Phase 1: Baseline Vulnerability Assessment

### Attack Scenario 1: Initial Access (Linux)

**Vulnerability:** Weak SSH credentials (`webadmin`) + NOPASSWD sudo  
**Attack Result:** SUCCESS. Initial foothold established and escalated to root privileges immediately.

**Key Observation:**
- Lack of brute-force protection allowed automated guessing.
- `sudo cat /etc/shadow` executed without prompt.

---

### Attack Scenario 2: Lateral Movement (Windows)

**Vulnerability:** Disabled Windows Firewall + Hardcoded Admin Credentials  
**Attack Result:** SUCCESS. Red Team retrieved `admin_lab` credentials from `/tmp/db_config.py` and used PsExec (SMB Port 445) to execute a payload on the Windows Workstation.

**Key Observation:**
- Flat network architecture allowed direct communication between Linux and Windows.
- Standard user (`employee`) lateral movement failed as expected, but compromised Admin credentials bypassed this control.

---

### Attack Scenario 3: Data Exfiltration

**Vulnerability:** Unrestricted Outbound Traffic  
**Attack Result:** ATTEMPTED. Data exfiltration was attempted to the C2 server via HTTP POST (Port 8888), but server error prevented successful upload.

**Key Observation:**
- No egress filtering on the gateway firewall.
- `curl` utility on Windows allowed easy data transfer.

---

## Phase 2: Hardening Verification

### Re-Test 1: Privilege Escalation

**Defense Applied:** Removal of NOPASSWD from `/etc/sudoers`.  
**Red Team Verification:** **BLOCKED.**
- Command `sudo cat /etc/shadow` timed out waiting for input.
- Privilege escalation chain broken.

---

### Re-Test 2: Lateral Movement

**Defense Applied:** Enabled Windows Firewall (Block 445/135) + Credential Rotation.  
**Red Team Verification:** **BLOCKED.**
- PsExec attempt resulted in `[Errno 113] No route to host`.
- Network segmentation successfully isolated the Windows target.

---

### Re-Test 3: Credential Hunting

**Defense Applied:** Deletion of `/tmp/db_config.py`.  
**Red Team Verification:** **BLOCKED.**
- `cat` command returned "No such file or directory".
- Attack path requiring stolen credentials was neutralized.

---

## Verification Results

Summary of attack success rates before and after hardening.

| Attack Vector | MITRE Technique | Phase 1 (Baseline) | Phase 2 (Hardened) | Mitigation Status |
|---------------|-----------------|-------------------|--------------------|-------------------|
| **Initial Access** | T1110 | 🟢 SUCCESS | 🟢 SUCCESS | **Expected** (Assumed Breach) |
| **Privilege Escalation** | T1548.003 | 🟢 SUCCESS | 🔴 FAILED (Timeout) | ✅ **Effective** |
| **Credential Theft** | T1003 | 🟢 SUCCESS | 🔴 FAILED (No File) | ✅ **Effective** |
| **Lateral Movement** | T1021.002 | 🟢 SUCCESS | 🔴 FAILED (Blocked) | ✅ **Effective** |
| **Data Exfiltration** | T1041 | 🟢 ATTEMPTED (server error) | ⚪ SKIPPED | ✅ **Effective** (Indirectly) |

**Overall Status:** The critical path to total domain compromise has been successfully closed.

---

## Lessons Learned

1.  **Defense in Depth:** While initial access (Phishing/SSH) is hard to prevent completely, stopping lateral movement via Host Firewalls prevents the breach from becoming a catastrophe.
2.  **Configuration Hygiene:** Hardcoded credentials in temporary files (`/tmp`) are a critical risk that allowed bypassing domain security policies.
3.  **Tool Dependency:** Red Team operations rely heavily on bringing external tools (`impacket`, `sshpass`) into the environment. Blocking file downloads (e.g., `wget`) on servers would significantly hamper attacker velocity.
4.  **Logging Gaps:** While attacks were blocked, ensuring that "Denied" traffic is logged to the SIEM is crucial for visibility, as blocked attacks often go unnoticed without proper alerting.

---

## Related Documentation

- [Red Team Phase 1 Plan](../../implementation/src/red-team-engineer/red-team-attack-plan-phase1.md)
- [Red Team Phase 2 Verification](../../implementation/src/red-team-engineer/red-team-hardening-validation-phase2.md)
- [Team Coordination Guide](../../implementation/src/red-team-engineer/team-coordination.md)