# Final report / summary

## Table of Contents
- [Role 1: Infrastructure Engineer – Summary](#role-1-infrastructure-engineer--summary)
- [Role 2: Blue Team Engineer – Summary](#role-2-blue-team-engineer--summary)
- [Role 3: Red Team Engineer – Summary](#role-3-red-team-engineer--summary)
- [Role 4: Threat Analyst – Summary](#role-4-threat-analyst--summary)

---

## Role 1: Infrastructure Engineer – Summary

As the Infrastructure Engineer for the TechNova cyber-range project, I successfully designed and deployed a complete enterprise-grade virtual laboratory environment for adversary simulation and defense training. The infrastructure consists of six interconnected virtual machines forming a realistic corporate network (TechNovaNet - 10.10.0.0/24).

### Key Accomplishments

**1. Network Architecture & Gateway (pfSense)**
- Configured pfSense firewall as the central network gateway with three network adapters (NAT for WAN, Internal Network for LAN, Host-Only for secure management)
- Implemented static routing and DNS forwarding to enable seamless connectivity across all lab components
- Established secure administrative access through dedicated management interface (192.168.56.2)
- Resolved complex VirtualBox networking issues to ensure proper traffic flow and internet connectivity for all VMs

**2. Active Directory Environment (Windows Server 2019)**
- Deployed Windows Server 2019 Core as Domain Controller for technova.local domain
- Configured DNS services and domain integration for enterprise authentication simulation
- Created vulnerable user accounts with weak passwords (alice, bob, manager) to enable realistic attack scenarios
- Prepared the AD environment for lateral movement and privilege escalation techniques

**3. Client Workstation (Windows 10)**
- Successfully joined Windows 10 workstation to the technova.local domain
- Installed and configured both CALDERA Sandcat agent and Wazuh monitoring agent
- Configured minimal security hardening to allow Red Team operations while maintaining realistic enterprise settings
- Established workstation as primary target for lateral movement and endpoint compromise scenarios

**4. Vulnerable Linux Server (Ubuntu 24.04 LTS)**
- Deployed Ubuntu Server with intentional security weaknesses for Red Team initial access
- Implemented multiple attack vectors: weak SSH credentials, password-based authentication, directory listing on Apache web server
- Configured dangerous sudo NOPASSWD settings for privilege escalation scenarios
- Deployed cron-based backdoor script as persistence mechanism
- Set up Apache web server with intentional misconfigurations

**5. MITRE CALDERA Framework (Attack Automation)**
- Installed and configured MITRE CALDERA server for automated adversary emulation
- Resolved critical installation issues: DNS resolution errors, emergency-shell bugs, permission issues
- Successfully deployed Sandcat agents on both Windows and Linux endpoints
- Validated agent connectivity and C2 communication channels
- Prepared platform for Red Team to execute MITRE ATT&CK technique chains

**6. Security Monitoring (Mini-Wazuh SIEM)**
- Deployed lightweight Wazuh SIEM solution optimized for laptop environments (2GB RAM configuration)
- Integrated Windows and Linux agents for centralized log collection and analysis
- Configured log forwarding from all critical endpoints (Domain Controller, Workstation, Ubuntu Server)
- Enabled Blue Team to monitor authentication events, process creation, privilege escalation, and suspicious activities
- Validated alert generation and dashboard functionality

**7. Remote Access & Management (Tailscale VPN)**
- Implemented Tailscale VPN solution for secure remote access to the lab environment
- Configured site-to-site VPN connectivity allowing external access without exposing services directly
- Ensured stable and encrypted management channel for distributed team collaboration

### Technical Challenges Overcome

- Resolved pfSense networking conflicts between VirtualBox NAT, Internal Network, and Host-Only adapters
- Fixed Windows Server 2019 Core DNS issues preventing Active Directory promotion
- Debugged CALDERA server DNS resolution failures and emergency-shell compatibility problems
- Corrected ownership and permission issues affecting CALDERA agent deployment
- Optimized Wazuh deployment for resource-constrained laptop environment
- Ensured consistent network connectivity across heterogeneous operating systems

### Environment Readiness

The final infrastructure provides:
- **Realistic attack surface** with intentional vulnerabilities matching real-world enterprise weaknesses
- **Complete visibility** through SIEM integration for Blue Team detection and response
- **Automation capabilities** via CALDERA for repeatable Red Team operations
- **Snapshot-based recovery** allowing rapid return to clean baseline state
- **Stable and documented** configuration enabling consistent testing across multiple attack phases

All components are fully operational, interconnected, and prepared for Red Team adversary simulation activities following MITRE ATT&CK framework methodologies. The environment successfully balances realism (vulnerable by design) with control (monitored and recoverable), providing an ideal platform for cybersecurity training and technique validation.

---

## Role 2: Blue Team Engineer – Summary

As the Blue Team Engineer for the TechNova cyber-range project, I successfully configured security monitoring, responded to Red Team attack simulations, and implemented comprehensive remediation measures to secure the environment.

### Key Accomplishments

**1. Pre-Attack Security Monitoring Setup**
- Verified Wazuh agents operational on Windows Workstation (10.10.0.50) and Linux Server (10.10.0.51)
- Deployed custom detection rules for SSH brute force, privilege escalation, PowerShell attacks, and lateral movement
- Configured Sysmon on Windows for deep process and network visibility
- Enabled File Integrity Monitoring (FIM) on Linux for critical files (`/etc/sudoers`, `/etc/shadow`, `/etc/crontab`)
- Configured pfSense syslog forwarding to Wazuh SIEM
- Validated telemetry pipeline with signal tests before Red Team engagement

**2. Detection Rules Development**
- Created MITRE ATT&CK-mapped detection rules:
  - T1110 (Brute Force): SSH authentication failure monitoring
  - T1548 (Sudo Abuse): NOPASSWD exploitation detection
  - T1071 (C2 Communication): PowerShell and curl network connection alerts
  - T1021 (Lateral Movement): SMB/RDP connection logging
- Deployed pfSense decoder for firewall log parsing in Wazuh

**3. Post-Attack Remediation (Phase 3)**

*Scenario 1 – SSH Brute Force:*
- Changed weak `webadmin` password to strong passphrase (`W3bAdm1nS3cure2025!`)
- Hardened `/etc/ssh/sshd_config`: `PermitRootLogin no`, `MaxAuthTries 3`, `PermitEmptyPasswords no`

*Scenario 2 – C2 Exfiltration:*
- Created pfSense firewall rule blocking TCP traffic from infected host (10.10.0.50) to C2 server (10.10.0.53) on port 8888

*Scenario 3 – Privilege Escalation:*
- Removed `NOPASSWD` entry from `/etc/sudoers` using visudo
- Deleted hardcoded credentials file (`/tmp/db_config.py`)
- Rotated compromised `admin_lab` domain account password

*Scenario 4 – Lateral Movement:*
- Enabled Windows Defender Firewall on all profiles (Domain, Public, Private)
- Created inbound block rules for CALDERA C2 (10.10.0.53) and compromised Linux server (10.10.0.51)
- Blocked SMB (445), RPC (135), and WMI (5985/5986) ports from external hosts
- Removed `splunkd.exe` CALDERA agent from Windows

**4. Linux Server Cleanup**
- Removed Impacket attack tools (`psexec.py`, `wmiexec.py`, `smbclient.py`)
- Killed and removed CALDERA agent (`splunkd`) from Linux server
- Verified no active C2 connections remained

### Verification Results

| Scenario | Vulnerability | Fix Applied | Status |
|----------|---------------|-------------|--------|
| SSH Brute Force | Weak Password | Password Changed + SSH Hardening | Verified |
| Privilege Escalation | NOPASSWD Sudo | Entry Removed | Verified |
| C2 Exfiltration | Open Outbound | pfSense Block Rule | Verified |
| Lateral Movement | Firewall Disabled | Firewall Enabled + Block Rules | Verified |
| Artifact Cleanup | Attack Tools & C2 Agent | Removed from both systems | Verified |

### Technical Skills Demonstrated

- SIEM configuration and custom rule development (Wazuh)
- Endpoint security hardening (Linux SSH, Windows Firewall)
- Network security controls (pfSense firewall rules)
- Incident response and artifact removal
- Attack → Detect → Fix → Verify methodology
- MITRE ATT&CK framework mapping for detection and response

All remediation measures were successfully validated through Red Team Phase 2 re-testing, confirming that previously successful attacks are now blocked by the implemented security controls.

---

## Role 3: Red Team Engineer – Summary

As the Red Team Engineer for the TechNova cyber-range project, I executed comprehensive adversary simulation campaigns to validate infrastructure vulnerabilities and test Blue Team defensive capabilities. The operations followed MITRE ATT&CK framework methodologies using the CALDERA automation platform.

### Key Accomplishments

**1. Attack Planning & Preparation (Phase 1 Setup)**
- Developed detailed attack scenarios mapping to MITRE ATT&CK techniques (T1078, T1548, T1021, T1041)
- Configured CALDERA adversary profiles with custom abilities for automated execution
- Implemented Fact Sources for dynamic targeting and credential management
- Prepared external tooling (Impacket, SSHpass) for lateral movement and exfiltration

**2. Phase 1: Baseline Attack Execution**
- Achieved full domain compromise starting from single Linux foothold via weak SSH credentials
- Executed privilege escalation using NOPASSWD sudo misconfigurations
- Performed lateral movement via SMB PsExec with harvested admin credentials
- Attempted data exfiltration to C2 server via HTTP POST (server error prevented successful upload)
- Generated attack telemetry to validate SIEM detection gaps and capabilities

**3. Phase 2: Hardening Validation**
- Re-executed identical attack chains against remediated environment
- Verified all critical vulnerabilities were neutralized (firewall blocks, credential removal, sudo hardening)
- Confirmed attack chain interruption at lateral movement stage
- Provided detailed feedback on remediation effectiveness to Blue Team

**4. Documentation & Reporting**
- Maintained detailed operation logs with timestamps and outcomes
- Created MITRE ATT&CK layer mappings for attack visualization
- Documented technical challenges and bypass techniques
- Generated findings reports for project stakeholders

### Attack Chain Validation Results

| Attack Vector | Phase 1 (Baseline) | Phase 2 (Hardened) | Status |
|---------------|-------------------|--------------------|--------|
| Initial Access (SSH) | SUCCESS | SUCCESS | Expected (Assumed Breach) |
| Privilege Escalation (Sudo) | SUCCESS | BLOCKED | Effective |
| Credential Harvesting | SUCCESS | BLOCKED | Effective |
| Lateral Movement (SMB) | SUCCESS | BLOCKED | Effective |
| Data Exfiltration (HTTP) | ATTEMPTED (server error) | BLOCKED | Effective |

### Technical Skills Demonstrated

- Adversary emulation using MITRE CALDERA framework
- MITRE ATT&CK technique implementation and chaining
- External tool integration (Impacket, SSHpass, curl)
- Living-off-the-land tactics and artifact cleanup
- Attack telemetry generation for detection testing
- Post-remediation validation and feedback provision
- Technical documentation and reporting

The Red Team operations successfully demonstrated realistic enterprise compromise scenarios while providing valuable validation data for defensive improvements. All attack chains were effectively neutralized in Phase 2, confirming the effectiveness of implemented security controls.

---

## Role 4: Threat Analyst – Summary

As the Threat Analyst for the TechNova cyber-range project, I was responsible for **analyzing Red Team attack activity**, **validating detection accuracy**, and **verifying the effectiveness of security controls** using telemetry-driven investigation rather than assumptions or tool output alone.

The role focused on correlating **endpoint logs, network telemetry, and SIEM data** to determine whether attacks truly succeeded or were effectively blocked, particularly during Phase 2 (Hardening Verification).

---

### Key Accomplishments

**1. Threat Hunting and Attack Validation**

- Conducted detailed threat hunting across Linux, Windows, and firewall telemetry
- Analyzed raw Wazuh data (`alerts.log`, `archives.log`) rather than relying solely on dashboard alerts
- Validated each stage of the attack chain (Initial Access → Privilege Escalation → Lateral Movement → Exfiltration)
- Confirmed whether attack stages were:
  - attempted,
  - blocked,
  - or not reached
- Distinguished between true attack activity and background system noise (cron jobs, multicast traffic, analyst actions)

---

**2. Phase 1 – Attack Confirmation**

- Verified successful attack execution during Phase 1 using:
  - SSH authentication logs on Linux
  - Sudo privilege escalation evidence
  - Network telemetry indicating lateral movement
  - Windows-side agent execution and persistence
- Correlated Red Team activity with Wazuh alerts to confirm:
  - initial detection gaps,
  - low-noise credential-based attack behavior,
  - and post-compromise visibility

This analysis confirmed that the Phase 1 attack chain was fully successful under baseline conditions.

---

**3. Phase 2 – Hardening Verification Through Telemetry**

- Re-analyzed the same attack techniques after remediation
- Confirmed failed SSH authentication attempts for `webadmin`
- Confirmed blocked sudo privilege escalation attempts
- Verified that `/etc/shadow` access was not achieved
- Analyzed pfSense `filterlog` data to validate:
  - absence of SMB/RPC traffic (ports 445/135)
  - effective network segmentation
- Confirmed that Windows hosts were not reached or compromised

The absence of lateral movement and Windows-side telemetry was correctly interpreted as **successful containment**, not missing data.

---

**4. Detection Accuracy and False Positive Avoidance**

- Identified and excluded analyst-generated events (e.g., sudo log inspection) from attack attribution
- Differentiated ephemeral source ports from true destination service ports in firewall logs
- Prevented misclassification of benign UDP traffic as lateral movement attempts
- Ensured all conclusions were based on observable evidence rather than assumptions

This approach prevented false positives and strengthened the credibility of the final assessment.

---

**5. Detection Gap Identification**

- Identified a detection gap related to Linux process execution visibility
- Confirmed that execution of failed attack tooling (e.g., `python3 psexec.py`, `nc`) was not observable in Wazuh
- Determined the root cause as missing Linux process execution telemetry (auditd / execve logging)
- Correctly classified this limitation as a **visibility gap**, not a failure of prevention or SIEM functionality

This finding provides a clear roadmap for future detection maturity improvements.

---

### Technical Skills Demonstrated

- Threat hunting methodology and hypothesis-driven analysis
- SIEM log analysis (Wazuh alerts and archives)
- Linux and Windows log interpretation
- Firewall telemetry analysis (pfSense filterlog)
- MITRE ATT&CK kill chain validation
- False positive reduction and evidence-based reporting
- Cross-team correlation (Red Team execution vs defensive telemetry)

---

### Final Assessment Contribution

The Threat Analyst role provided the **final validation layer** between offensive simulation and defensive assurance. By correlating attack execution with system telemetry, I confirmed that:

- Phase 1 attacks were genuinely successful under baseline conditions
- Phase 2 hardening measures effectively blocked all critical attack paths
- Detection mechanisms operated correctly within their configured scope
- Remaining detection gaps were accurately identified and scoped

This analysis ensured that the final project conclusions were **technically sound, defensible, and evidence-based**, completing the full security lifecycle of the TechNova cyber-range project.