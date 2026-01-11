# Team Coordination Guide

This document outlines the responsibilities and tasks for each team member during the attack simulation exercise.

## Table of Contents

- [Timeline Overview](#timeline-overview)
- [Roles and Responsibilities](#roles-and-responsibilities)
  - [Red Team Engineer](#red-team-engineer)
  - [Threat Analyst](#threat-analyst)
  - [Blue Team Engineer](#blue-team-engineer)
- [Communication Protocol](#communication-protocol)

## Timeline Overview

1.  **Phase 0: Blue Team Setup**  Blue Team Engineer prepares monitoring.
2.  **Phase 1: Attack Execution**  Red Team executes attacks.
3.  **Phase 2: Detection & Logging**  Threat Analyst monitors and documents.
4.  **Phase 3: Remediation**  Blue Team Engineer fixes vulnerabilities.
5.  **Phase 4: Verification**  All teams verify fixes work.

---

## Roles and Responsibilities

### Red Team Engineer
**Responsibility:** Execute the 4 planned attack scenarios against the infrastructure.

*   **Scenario 1 (SSH Brute Force):** Attempt to brute-force the Linux server using weak credentials.
*   **Scenario 2 (C2 Exfiltration):** Simulate data exfiltration from the Windows Workstation using PowerShell.
*   **Scenario 3 (Privilege Escalation):** Exploit misconfigurations on the Linux server to gain root access.
*   **Scenario 4 (Lateral Movement):** Attempt to scan and connect to the Windows Workstation from the CALDERA server.
*   **Post-Attack:** Notify the team upon successful execution of all scenarios.

### Threat Analyst
**Responsibility:** Monitor security logs in real-time and collect evidence of the attacks.

*   **Monitoring:** Watch Wazuh logs on the central server during the attack phase.
*   **Detection:** Identify specific alerts for SSH failures, PowerShell network connections, sudo abuse, and lateral movement attempts.
*   **Evidence Collection:** Capture screenshots of alerts and export relevant logs for the final report.
*   **Reporting:** Provide the Blue Team with a summary of detections and evidence.

### Blue Team Engineer
**Responsibility:** Prepare the environment, remediate vulnerabilities, and verify security fixes.

*   **Setup (Phase 0):** Ensure all agents are active, monitoring is functional, and intentional vulnerabilities are in place.
*   **Remediation (Phase 3):**
    *   **Scenario 1:** Enforce strong passwords and harden SSH configuration.
    *   **Scenario 2:** Configure pfSense firewall rules to block malicious outbound traffic.
    *   **Scenario 3:** Fix sudoers misconfigurations and remove unauthorized cron jobs.
    *   **Scenario 4:** Enable and configure the Windows Host Firewall to block unauthorized IPs.
*   **Verification (Phase 4):** Confirm that all attacks are successfully blocked after remediation.

---

## Communication Protocol

*   **Blue Team:** Announce when the environment is ready for attacks.
*   **Red Team:** Announce the start and completion of each attack scenario.
*   **Threat Analyst:** Confirm when evidence has been successfully captured.
*   **Blue Team:** Announce when remediation is complete and ready for verification.
