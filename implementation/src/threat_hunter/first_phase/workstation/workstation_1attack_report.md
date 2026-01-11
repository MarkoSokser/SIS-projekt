# Windows Attack Report (Post-Attack)
## Table of Contents
1. [Purpose & Role of the System](#1-purpose--role-of-the-system)
2. [Attack Overview](#2-attack-overview)
3. [Authentication Failures (Event ID 4625)](#3-authentication-failures-event-id-4625)
4. [Successful Authentication Events (Event ID 4624)](#4-successful-authentication-events-event-id-4624)
5. [Service Creation and Persistence (Event ID 7045)](#5-service-creation-and-persistence-event-id-7045)
6. [Attack Timeline Summary](#6-attack-timeline-summary)
7. [Detection Gaps and Observations](#7-detection-gaps-and-observations)
8. [Key Detection Points for Future Monitoring](#8-key-detection-points-for-future-monitoring)
9. [Conclusion](#9-conclusion)

## 1. Purpose & Role of the System

The Windows workstation serves as a domain-joined endpoint within the Technova lab environment and represents a primary target for lateral movement following initial access to the Linux server.

From an attack lifecycle perspective, this system is relevant because it simulates:
- a compromised user workstation,
- a target for credential-based lateral movement,
- a host for service-based persistence,
- a source of domain and host-level intelligence.

During the attack scenario, this system was accessed after credentials were obtained from the Linux server, and all activity was recorded locally through Windows Event Logging.

---

## 2. Attack Overview

The attack against the Windows workstation occurred after successful lateral movement from the Linux server.  
The attacker used valid credentials to authenticate remotely, followed by multiple failed and successful authentication attempts, and subsequently established persistence via service creation.

Because of infrastructure constraints, the Wazuh server was unavailable during part of the attack. As a result, detection relies entirely on native Windows Security and System event logs.

---

## 3. Authentication Failures (Event ID 4625)

Multiple failed authentication attempts were observed in the Windows Security log, consistent with credential testing and enumeration activity.

### Observed Characteristics

- Event ID: 4625 (Audit Failure)
- Logon Types observed:
  - LogonType 2 (Interactive)
  - LogonType 3 (Network)
- Authentication packages:
  - NTLM
  - Negotiate
- Targeted accounts:
  - `employee`
  - `read_team`

These events indicate repeated failed authentication attempts using legitimate account names, suggesting credential guessing or validation prior to successful access.

### Evidence

![Windows Security Event 4625 – failed authentication (1)](./images/security_4025_1.png)
*Figure 1: failed authentication attempts targeting `employee` account*

![Windows Security Event 4625 – failed authentication (2)](./images/security_4025_2.png)
*Figure 2: 2. failed authentication attempts targeting `employee` account*

![Windows Security Event 4625 – failed authentication (3)](./images/security_4025_3.png)
*Figure 3: failed authentication attempts targeting `read_team` account*

---

## 4. Successful Authentication Events (Event ID 4624)

Following authentication failures, multiple successful logon events were recorded, confirming successful lateral movement.

### Key Successful Logons

- Event ID: 4624 (Audit Success)
- Target account:
  - `admin_lab`
- LogonType:
  - LogonType 3 (Network)
- Authentication method:
  - NTLM
  - Kerberos
- Source workstation:
  - `DESKTOP-K5RR24G`

These events confirm that valid credentials were successfully used to access the workstation remotely.

### Evidence

![Windows Security Event 4624 – successful logon (1)](./images/security_4624_1.png)
*Figure 4: successful logon event targeting `admin_lab` account*

![Windows Security Event 4624 – successful logon (2)](./images/security_4624_2.png)
*Figure 5: 2. successful logon event targeting `admin_lab` account*

![Windows Security Event 4624 – successful logon (3)](./images/security_4624_4.png)
*Figure 6: 3. successful logon event targeting desktop account*

---

## 5. Service Creation and Persistence (Event ID 7045)

After gaining access, the attacker created multiple services on the system, a common persistence technique.

### Observed Service Creation Events

- Event ID: 7045 (Service Control Manager)
- Service characteristics:
  - Randomized service names (e.g. `bQxi`, `ZBNB`, `eHRe`, `QnGx`)
  - Executables located in system paths
  - Services configured for on-demand start
  - Executed under `LocalSystem` account

These patterns are strongly indicative of malicious service-based persistence rather than legitimate software installation.

### Evidence

![Service creation event 7045 – persistence artifact (1)](./images/system_7045_1.png)
*Figure 7: service creation event for `bQxi` service*

![Service creation event 7045 – persistence artifact (2)](./images/system_7045_2.png)
*Figure 8: service creation event for `ZBNB` service*

![Service creation event 7045 – persistence artifact (3)](./images/system_7045_3.png)
*Figure 9: service creation event for `eHRe` service*

![Service creation event 7045 – persistence artifact (4)](./images/system_7045_4.png)
*Figure 10: service creation event for `QnGx` service*

---

## 6. Attack Timeline Summary

1. Multiple failed authentication attempts (Event ID 4625)
2. Successful network-based authentication using valid credentials (Event ID 4624)
3. Establishment of persistence via service creation (Event ID 7045)

This sequence aligns with a realistic post-compromise lateral movement and persistence scenario.

---

## 7. Detection Gaps and Observations

- No Wazuh alerts were generated for this attack phase due to the SIEM server being unavailable.
- All evidence was preserved locally in Windows Security and System logs.
- The attack demonstrates that Windows native logging alone is sufficient to reconstruct the full attack chain when properly analyzed.

---

## 8. Key Detection Points for Future Monitoring

Based on this attack, the following events are critical for detection:

- Repeated Event ID 4625 failures targeting valid domain accounts
- Event ID 4624 LogonType 3 events from unusual workstations
- Event ID 7045 service creation with non-standard service names
- Services running as `LocalSystem` without clear business justification

---

## 9. Conclusion

The Windows workstation was successfully compromised via credential-based lateral movement, followed by persistence through malicious service creation.

Despite the absence of centralized SIEM alerts during the attack window, Windows Event Logs provided sufficient forensic evidence to fully reconstruct attacker actions and intent.

This confirms the importance of endpoint-level log retention and highlights the value of correlating authentication and service management events during incident response.
