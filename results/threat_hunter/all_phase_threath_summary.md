# Final Security Assessment Report  
## TechNova Lab – Baseline, Attack Simulation, and Hardening Validation
## Table of Contents
1. [Executive Overview](#1-executive-overview)
2. [Environment Overview](#2-environment-overview)
3. [Phase 0 – Pre-Attack Baseline Summary](#3-phase-0--pre-attack-baseline-summary)
4. [Phase 1 – Adversary Emulation (Baseline / Weak Security)](#4-phase-1--adversary-emulation-baseline--weak-security)
  4.1. [Linux Host – Phase 1 Summary](#41-linux-host--phase-1-summary)
  4.2. [Windows Host – Phase 1 Summary](#42-windows-host--phase-1-summary)
  4.3. [Wazuh SIEM – Phase 1 Summary](#43-wazuh-siem--phase-1-summary)
5. [Phase 2 – Hardening Verification](#5-phase-2--hardening-verification)
  5.1. [Linux Host – Phase 2 Summary](#51-linux-host--phase-2-summary)
  5.2. [Windows Host – Phase 2 Summary](#52-windows-host--phase-2-summary)
  5.3. [Wazuh SIEM – Phase 2 Summary](#53-wazuh-siem--phase-2-summary)
6. [Detection Gaps and Visibility Limitations](#6-detection-gaps-and-visibility-limitations)
7. [Cross-Phase Comparative Assessment](#7-cross-phase-comparative-assessment)
8. [Key Lessons Learned](#8-key-lessons-learned)
9. [Final Conclusion](#9-final-conclusion)

---

## 1. Executive Overview

This report presents the **final consolidated security assessment** of the TechNova lab environment.  
The assessment was conducted through a structured, three-phase approach:

1. **Pre-Attack Baseline Establishment**
2. **Phase 1 – Adversary Emulation (Baseline / Weak Security)**
3. **Phase 2 – Hardening Verification and Re-Attack**

Each phase was documented independently and is referenced throughout this report for traceability and verification:

- Baseline summary: [baseline/summary.md](../../implementation/src/threat_hunter/baseline/summary.md)
- Phase 1 summary: [first_phase/summary.md](../../implementation/src/threat_hunter/first_phase/summary.md)
- Phase 2 summary: [second_phase/summary.md](../../implementation/src/threat_hunter/second_phase/summary.md)



This final report synthesizes findings from all three phases into a single, end-to-end security assessment.

---

## 2. Environment Overview

The TechNova lab consists of the following components:

- **Linux Server (`ubuntu-srv`)**  
  Internal infrastructure host and initial foothold during Phase 1  
  *(see `baseline/summary.md`, `first_phase/summary.md`)*

- **Windows Workstation (`WIN-CLIENT`)**  
  Lateral movement and persistence target  
  *(see `first_phase/summary.md`, `second_phase/summary.md`)*

- **pfSense Firewall**  
  Network segmentation and enforcement point  
  *(baseline and Phase 2 validation documented in `baseline/summary.md` and `second_phase/summary.md`)*

- **Wazuh SIEM**  
  Central detection and correlation platform  
  *(detection behavior analyzed across all three summaries)*

This architecture enabled realistic multi-host attack simulation and layered defensive validation.

---

## 3. Phase 0 – Pre-Attack Baseline Summary

A comprehensive baseline was established before any adversary activity.  
This baseline serves as the **ground truth reference** for all later analysis.

Key characteristics included:
- Stable system operation
- Verified log ingestion into Wazuh
- Documented intentional weaknesses
- No malicious persistence or artifacts

The full baseline context and observations are documented in:  
 **`baseline/summary.md`**

This baseline enabled accurate differentiation between:
- normal system behavior,
- background noise,
- and true attack artifacts observed in later phases.

---

## 4. Phase 1 – Adversary Emulation (Baseline / Weak Security)

Phase 1 simulated a realistic multi-stage attack using **MITRE CALDERA**, targeting the environment in its intentionally weakened state.

### 4.1 Linux Host – Phase 1 Summary

- Initial access achieved using valid credentials
- Privilege escalation succeeded due to misconfigured sudo
- Credential material accessed
- Persistence established
- Detection noise deliberately generated

Detailed attack execution and evidence are documented in:  
 **`first_phase/summary.md`**

---

### 4.2 Windows Host – Phase 1 Summary

- Lateral movement from Linux to Windows succeeded
- CALDERA agent deployed and executed
- Persistence and discovery activities confirmed
- Full kill chain execution achieved

Windows-side compromise details and evidence are available in:  
 **`first_phase/summary.md`**

---

### 4.3 Wazuh SIEM – Phase 1 Summary

- Early Phase 1 detection was limited due to configuration state
- Credential-based techniques demonstrated low-noise characteristics
- Once enabled, Wazuh successfully detected authentication, escalation, and persistence artifacts

Phase 1 detection behavior and limitations are documented in:  
 **`first_phase/summary.md`**

---

## 5. Phase 2 – Hardening Verification

Phase 2 re-executed the same attack techniques against the hardened environment to validate defensive effectiveness.

### 5.1 Linux Host – Phase 2 Summary

- SSH authentication attempts failed
- Privilege escalation via sudo failed
- Credential dumping attempts were blocked
- Attack chain terminated early

Linux hardening validation is documented in:  
 **`second_phase/summary.md`**

---

### 5.2 Windows Host – Phase 2 Summary

- SMB/RPC access blocked by network segmentation
- No agent deployment
- No persistence or execution artifacts
- Windows host remained uncompromised

Windows containment validation is documented in:  
 **`second_phase/summary.md`**

---

### 5.3 Wazuh SIEM – Phase 2 Summary

- Failed authentication attempts correctly detected
- Analyst activity correctly distinguished from attacker activity
- Network telemetry confirmed absence of lateral movement
- Correlation aligned with CALDERA execution output

SIEM behavior and validation details are documented in:  
 **`second_phase/summary.md`**

---

## 6. Detection Gaps and Visibility Limitations

Across phases, a consistent detection gap was identified:

- Lack of Linux process execution telemetry (auditd / execve)

This gap:
- did not affect prevention or containment,
- limited visibility into failed, non-privileged process execution,
- represents a clear future improvement opportunity.

This limitation and its impact are discussed in detail in:  
 **`second_phase/summary.md`**

---

## 7. Cross-Phase Comparative Assessment

| Security Aspect              | Phase 1 | Phase 2 |
|-----------------------------|---------|---------|
| Initial Access              | Allowed | Blocked |
| Privilege Escalation        | Allowed | Blocked |
| Credential Access           | Allowed | Blocked |
| Lateral Movement            | Successful | Contained |
| Persistence                 | Established | Prevented |
| Windows Compromise          | Yes | No |
| SIEM Detection Accuracy     | Partial | Reliable |

(Individual phase details: `first_phase/summary.md`, `second_phase/summary.md`)

---

## 8. Key Lessons Learned

1. Credential-based attacks are low-noise and difficult to detect without correlation  
2. Preventive controls significantly reduce reliance on detection  
3. SIEM effectiveness is directly tied to telemetry coverage  
4. Accurate baselines are essential for correct attribution  
5. Defense-in-depth provides measurable resilience improvements  

Lessons learned per phase are documented across:
- `baseline/summary.md`
- `first_phase/summary.md`
- `second_phase/summary.md`

---

## 9. Final Conclusion

This assessment demonstrates a complete security lifecycle:

- Phase 0 established a trusted baseline
- Phase 1 validated exploitability of documented weaknesses
- Phase 2 confirmed that hardening eliminated those attack paths

Preventive controls, network segmentation, and SIEM correlation collectively delivered a hardened and resilient environment, with clearly identified next steps for detection maturity.

**Final Assessment Result:**  
Security objectives achieved  
Attack paths eliminated  
Detection visibility improvements identified  

This report represents the consolidated conclusion of all assessment phases.

---
