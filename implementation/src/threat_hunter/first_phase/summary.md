# Final Attack Summary and Cross-System Analysis

## 1. Overview

This report presents a consolidated analysis of a multi-stage attack conducted within the Technova lab environment, spanning three core components:

- Linux server (`ubuntu-srv`)
- Windows workstation (domain-joined endpoint)
- Wazuh SIEM platform

The objective of this summary is to correlate observations across all systems, explain detection outcomes, and provide a holistic view of attacker behavior, detection effectiveness, and visibility gaps.

---

## 2. Attack Chain Summary

The attack followed a realistic adversary workflow aligned with common post-exploitation techniques:

1. **Initial Access (Linux)**  
   The attacker gained access to the Linux server via SSH using valid credentials.  
   No brute-force activity was observed prior to the successful login, indicating credential-based access rather than password guessing.

2. **Privilege Escalation and Local Discovery (Linux)**  
   After login, the attacker leveraged misconfigured privileges (`NOPASSWD sudo`) to escalate to root.  
   Sensitive files such as `/etc/shadow` were accessed, and persistence was established via a custom systemd service.

3. **Lateral Movement (Windows)**  
   Credentials obtained from the Linux server were used to authenticate to the Windows workstation.  
   Multiple failed authentication attempts preceded a successful login, consistent with credential validation and enumeration.

4. **Persistence (Windows)**  
   The attacker created multiple services using randomized names, executed under the `LocalSystem` account, indicating deliberate persistence mechanisms.

---

## 3. Linux Host Findings

- Successful SSH authentication for user `webadmin` was confirmed in rotated authentication logs.
- Extensive authentication failures were later observed during noise generation, including:
  - SSH authentication failures
  - `su` failures
- Privilege escalation to root occurred via sudo.
- Sensitive file access (`/etc/shadow`) was explicitly logged.
- A malicious systemd service was created and started.

All activities were recorded in `/var/log/auth.log` and its rotated counterparts and were successfully detected by Wazuh once archiving and alerting were validated.

---

## 4. Windows Host Findings

- Repeated authentication failures (Event ID 4625) targeting valid domain users were observed.
- Successful authentication events (Event ID 4624, LogonType 3) confirmed lateral movement.
- Authentication mechanisms included NTLM and Kerberos.
- Persistence was established via service creation (Event ID 7045).
- Service names were non-descriptive and inconsistent with legitimate software installations.

Despite the absence of real-time SIEM monitoring during parts of the attack, Windows native logs fully captured the attack lifecycle.

---

## 5. Wazuh SIEM Analysis

### 5.1 Initial Detection Gap Explanation

During the first attack iteration:
- Log archiving (`logall`) was disabled.
- The attack relied primarily on:
  - valid credentials,
  - standard system utilities,
  - native authentication mechanisms.

As a result:
- No high-severity alerts were generated.
- No archived raw events were stored for retrospective analysis.

This behavior is consistent with Wazuh operating as designed: it prioritizes anomalous or rule-matching behavior rather than all legitimate activity.

### 5.2 Post-Reconfiguration Results

After confirming agent connectivity and validating alerting:
- Authentication failures
- Successful login sessions
- Privilege escalation events
were all detected and logged by Wazuh.

This confirms that Wazuh was functional and capable of detecting noisy or misconfigured behavior, but not stealthy credential-based activity by default.

---

## 6. Key Lessons Learned

1. **Credential-Based Attacks Are Low-Noise by Nature**  
   Attacks using valid credentials can bypass default SIEM alerting without additional correlation rules.

2. **Endpoint Logs Are Critical**  
   Both Linux and Windows native logs independently provided sufficient evidence to reconstruct the full attack chain.

3. **SIEM Visibility Depends on Configuration and Use Case**  
   Wazuh did not fail; it operated within its default detection scope. Enhanced detection would require:
   - brute-force correlation rules,
   - privilege escalation thresholds,
   - persistence-focused custom rules.

4. **Infrastructure Constraints Affect Detection, Not Evidence**  
   Temporary unavailability of the SIEM did not prevent forensic reconstruction, as logs remained intact on endpoints.

---

## 7. Final Conclusion

The attack scenario successfully demonstrated a realistic end-to-end compromise involving initial access, privilege escalation, lateral movement, and persistence across heterogeneous systems.

While Wazuh did not initially generate alerts for stealthy phases of the attack, subsequent validation confirmed that detection gaps were attributable to configuration choices and attack methodology rather than system failure.

This exercise highlights the importance of:
- layered detection,
- endpoint log retention,
- and correlation-based threat hunting
in modern enterprise environments.

The combined analysis of Linux, Windows, and SIEM telemetry provides a complete and coherent view of attacker behavior and defensive visibility.

