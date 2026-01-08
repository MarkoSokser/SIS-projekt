# Final Phase 2 Summary – Hardening Verification  
*(Linux + Windows + Wazuh SIEM)*

## Objective

The objective of **Phase 2 (Hardening Verification)** was to re-execute previously successful attack techniques against the hardened environment and verify whether implemented security controls effectively **prevent, contain, and detect** malicious activity.

This phase focused on validating:
- Prevention of initial access and privilege escalation
- Containment of lateral movement
- Visibility and detection capabilities across endpoints and SIEM
- Identification of remaining detection gaps

---

## Linux Host (ubuntu-srv) – Summary

### Prevention Outcome
All attack attempts against the Linux host were **unsuccessful**:

- **Initial Access:**  
  Failed SSH authentication attempts targeting the `webadmin` user were observed and blocked.

- **Privilege Escalation:**  
  Attempts to abuse `sudo` privileges failed due to proper privilege restrictions and password enforcement.

- **Credential Access:**  
  Access to `/etc/shadow` was not achieved; credential dumping was prevented.

As a result, the attack chain was **terminated early** and did not progress to discovery, lateral movement, or exfiltration stages.

### Detection Outcome
- SSH authentication failures were successfully logged and correlated.
- Failed sudo attempts were visible and correctly interpreted.
- No false positives related to advanced attack stages were generated.

### Identified Detection Gap
- Execution of failed attack tooling (e.g., `python3 psexec.py`, `nc`) was not visible at the SIEM level.
- Root cause: lack of Linux process execution telemetry (auditd / execve logging).

This gap does **not** affect prevention but represents an opportunity for improved visibility.

---

## Windows Host (WIN-CLIENT) – Summary

### Prevention Outcome
The Windows workstation was **not compromised** during Phase 2:

- No lateral movement was successful
- No CALDERA agent was deployed
- No malicious process execution occurred
- No persistence mechanisms were established

Firewall segmentation and network hardening prevented SMB/RPC access from the Linux host.

### Detection Outcome
- No Windows-side security alerts were generated because no attack traffic or execution reached the system.
- The absence of Windows telemetry is an **expected and correct outcome**, confirming effective containment.

---

## Wazuh SIEM – Summary

### Successful Detection
Wazuh successfully detected and correlated:
- Failed SSH authentication attempts
- Authentication-related sudo activity
- Relevant security events aligned with Phase 2 attack attempts

Analyst activity (e.g., log inspection via sudo) was correctly logged and distinguishable from attack activity.

### Lateral Movement Validation
Network telemetry (pfSense `filterlog`) confirmed:
- No TCP connections to SMB/RPC ports (445/135)
- No traffic directed toward the Windows host
- Firewall rules effectively blocked lateral movement attempts

Correlation with Red Team tooling confirmed failed PsExec execution due to network segmentation (`No route to host`).

### Detection Gap
- Wazuh did not observe execution of failed Linux attack processes.
- Cause: missing process execution telemetry on Linux endpoints.
- Impact: limited visibility into failed, non-privileged attack tooling.

This gap is clearly scoped and does not undermine overall security posture.

---

## Overall Phase 2 Assessment

| Security Objective          |              Result               |
|-----------------------------|-----------------------------------|
| Initial Access Prevention   | ✅ Successful |
| Privilege Escalation Block  | ✅ Successful |
| Lateral Movement Containment| ✅ Successful |
| Endpoint Compromise         | ❌ Not Achieved |
| SIEM Detection Accuracy     | ✅ Reliable |
| Detection Coverage Gaps     | ⚠️ Identified (Process telemetry) |

---

## Final Conclusion

Phase 2 confirms that the hardened environment is **resilient against the previously successful attack paths**. Preventive controls effectively blocked escalation and movement, while Wazuh provided accurate detection for authentication-based attack attempts and reliable validation of network containment.

The only identified limitation relates to **process execution visibility on Linux**, representing a clear and actionable improvement area rather than a security failure.