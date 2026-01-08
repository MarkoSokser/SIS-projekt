# Theoretical background

## Table of Contents
- [Infrastructure Engineering for Cybersecurity Lab Environments](#infrastructure-engineering-for-cybersecurity-lab-environments)
- [Virtual Lab Architecture](#2-virtual-lab-architecture)
- [Operating System Configuration](#3-operating-system-configuration)
- [Vulnerability Implementation](#4-vulnerability-implementation)
- [Monitoring Infrastructure](#5-monitoring-infrastructure)
- [Baseline Management](#6-baseline-management)
- [Red Team & Blue Team Support](#7-red-team--blue-team-support)
- [Common Technical Challenges](#8-common-technical-challenges)
- [Blue Team Engineering for Cybersecurity Defense](#blue-team-engineering-for-cybersecurity-defense)
- [Threat Hunting and SIEM-Based Attack Validation](#threat-hunting-and-siem-based-attack-validation)
---

## Infrastructure Engineering for Cybersecurity Lab Environments

### 1. Introduction

Infrastructure Engineering in cybersecurity involves designing, building, and maintaining controlled environments for adversary simulation. The role creates the foundation for offensive and defensive security operations by balancing realistic vulnerabilities with monitored, controlled systems.


### 2. Virtual Lab Architecture

#### Network Topology

**Flat Networks** are commonly used in training labs because they:
- Reflect real-world small/medium business configurations
- Enable lateral movement demonstrations
- Simplify setup while maintaining realism

The TechNovaNet (10.10.0.0/24) represents this approach with all systems in a single broadcast domain.

#### Virtualization

Key concepts used in lab deployment:
- **Snapshots**: Point-in-time VM states for quick recovery
- **Resource allocation**: CPU, RAM, and storage distribution
- **Network modes**: Internal networking for isolated lab environments



### 3. Operating System Configuration

#### Windows Server & Active Directory

**Active Directory Domain Services (AD DS)** provides:
- Centralized authentication (Kerberos)
- DNS services
- Group Policy management
- User and computer organization

**Intentional AD Vulnerabilities** for training:
- Weak password policies
- Excessive user privileges
- Kerberoasting opportunities
- Pass-the-Hash vulnerabilities

#### Linux Server Configuration

**Security-relevant configurations**:
- User privileges (`/etc/sudoers`)
- SSH access (`/etc/ssh/sshd_config`)
- Web services (Apache directory listing)
- Cron jobs and scheduled tasks

#### Network Appliances

**pfSense** serves as the network gateway providing:
- Routing between network segments
- NAT configuration
- Firewall rules
- Network traffic control


### 4. Vulnerability Implementation

#### Purpose of Intentional Weaknesses

Controlled vulnerabilities enable hands-on practice with:
- Real-world attack techniques
- MITRE ATT&CK framework tactics
- Detection and response procedures

#### Common Vulnerability Types

**Authentication**:
- Weak/reused passwords
- Password-based SSH access

**Authorization**:
- Overly permissive sudo (NOPASSWD)
- Excessive AD privileges

**Configuration**:
- Directory listing enabled
- Unnecessary services running
- Minimal security hardening

#### MITRE ATT&CK Mapping

Implemented vulnerabilities map to specific techniques:
- **Initial Access** (T1078): Weak credentials
- **Privilege Escalation** (T1548): Sudo misconfigurations
- **Lateral Movement** (T1021): SMB, RDP, SSH access
- **Persistence** (T1053): Cron/scheduled tasks



### 5. Monitoring Infrastructure

#### SIEM Implementation (Wazuh)

**Core Functions**:
- Log aggregation from Windows and Linux endpoints
- Real-time event correlation
- Threat detection and alerting
- Forensic analysis capabilities

**Architecture Components**:
- Manager: Central analysis and storage
- Agents: Endpoint data collectors
- Dashboards: Visualization and reporting

#### Key Events to Monitor

- Failed login attempts (brute force detection)
- Privilege escalations
- Process creations and network connections
- File modifications
- Service changes



### 6. Baseline Management

#### Golden State Snapshots

A "golden state" snapshot includes:
- Clean, configured operating system
- All monitoring agents installed (CALDERA, Wazuh)
- Implemented vulnerabilities documented
- Network configuration completed

**Purpose**: Enable rapid reset to known-good state between attack simulations.



### 7. Red Team & Blue Team Support

#### Red Team Requirements

Infrastructure must provide:
- CALDERA framework for automated adversary emulation
- Vulnerable entry points
- Lateral movement pathways
- Realistic enterprise environment

#### Blue Team Requirements

Infrastructure must enable:
- SIEM access for log analysis
- Comprehensive monitoring coverage
- Alert investigation capabilities
- Incident response practice



### 8. Common Technical Challenges

**DNS Issues**: Resolved through proper DNS configuration and static entries  
**Network Connectivity**: Verified via routing tables and firewall rules  
**Resource Constraints**: Managed through optimized VM configurations (minimal Wazuh: 2GB RAM)  
**Agent Installations**: Troubleshot using dependency verification and permission checks

---

## Blue Team Engineering for Cybersecurity Defense

### 1. Introduction to Blue Team Operations

Blue Team Engineering focuses on defensive security operations within enterprise environments. The role encompasses monitoring, detection, incident response, and security hardening to protect systems against adversarial threats.

**Core Responsibilities:**
- Configure and maintain security monitoring infrastructure
- Develop detection rules and alert mechanisms
- Respond to security incidents and remediate vulnerabilities
- Validate security controls through verification testing

### 2. Security Monitoring and Detection

#### SIEM Configuration

**Wazuh Agent Deployment** enables centralized visibility:
- Windows endpoints: Sysmon integration for process/network monitoring
- Linux endpoints: File Integrity Monitoring (FIM) for critical files
- Network appliances: Syslog forwarding from pfSense

#### Custom Detection Rules

Effective detection requires rules mapped to attack techniques:
- **Authentication monitoring**: SSH brute force detection (failed attempts)
- **Privilege escalation**: Sudo abuse and NOPASSWD exploitation
- **Command and Control**: PowerShell network connections, curl/wget activity
- **Lateral movement**: SMB connections, service creation events

#### MITRE ATT&CK Integration

Detection rules should reference ATT&CK techniques:
- T1110 (Brute Force) → Authentication failure alerts
- T1548 (Abuse Elevation Control) → Sudo command monitoring
- T1071 (Application Layer Protocol) → Network connection alerts
- T1021 (Remote Services) → SMB/RDP connection logging

### 3. Security Hardening

#### Host-Based Hardening

**Linux Systems:**
- SSH configuration (`/etc/ssh/sshd_config`): Disable root login, limit auth attempts
- Sudoers cleanup: Remove NOPASSWD entries
- Password policies: Enforce strong, complex passwords
- Artifact removal: Clean up attacker tools and persistence mechanisms

**Windows Systems:**
- Windows Defender Firewall: Enable and configure inbound rules
- Block rules: Restrict connections from known-malicious IPs
- Malware cleanup: Remove C2 agents and attack tools

#### Network-Based Hardening

**Firewall Rules (pfSense):**
- Block outbound traffic to known C2 servers
- Segment internal networks to limit lateral movement
- Enable logging for visibility into blocked traffic

### 4. Incident Response Methodology

#### Attack → Detect → Fix → Verify

The Blue Team follows a structured approach:
1. **Attack**: Understand the threat through Red Team simulation
2. **Detect**: Identify indicators of compromise via SIEM alerts
3. **Fix**: Apply remediation measures (hardening, patching)
4. **Verify**: Confirm fixes block repeat attacks

#### Evidence Collection

Proper documentation supports incident response:
- Screenshot capture of before/after states
- Log exports for forensic analysis
- Configuration snapshots for audit trails

### 5. Verification and Validation

#### Post-Remediation Testing

After applying fixes, Blue Team validates effectiveness:
- Re-run attack scenarios to confirm blocks
- Monitor SIEM for detection of blocked attempts
- Document successful mitigations

#### Defense-in-Depth Validation

Multiple security layers should be verified:
- Network layer: Firewall blocks malicious traffic
- Host layer: Endpoint controls prevent execution
- Application layer: Authentication controls deny access

---

## Threat Hunting and SIEM-Based Attack Validation

### 1. Introduction to Threat Hunting

Threat Hunting represents a proactive security discipline focused on identifying malicious activity that may evade automated detection mechanisms. Unlike traditional alert-driven monitoring, threat hunting relies on analyst-driven investigation, hypothesis testing, and telemetry correlation.

In the context of this cybersecurity lab, threat hunting was used to:
- Validate Red Team attack execution
- Confirm whether attacks succeeded or failed
- Distinguish real attack activity from system noise
- Verify the effectiveness of security hardening measures

---

### 2. Telemetry-Driven Analysis vs Alert-Driven Analysis

A critical distinction in modern security operations is the difference between alerts and telemetry.

**Alerts** represent events that matched predefined detection rules.  
**Telemetry** represents the raw data generated by systems, networks, and applications.

During the project, analysis focused primarily on telemetry sources, including:
- Linux authentication logs (`auth.log`)
- Windows security and system logs
- Firewall network telemetry (pfSense `filterlog`)
- Wazuh archives and alert streams

This approach ensured that conclusions were based on observable system behavior rather than assumptions derived from tool output.

---

### 3. Attack Chain Validation Methodology

Each attack attempt was analyzed by mapping observed telemetry to expected stages of the MITRE ATT&CK kill chain:

1. Initial Access  
2. Privilege Escalation  
3. Credential Access  
4. Discovery  
5. Lateral Movement  
6. Persistence  
7. Exfiltration  

For each stage, the analyst evaluated:
- Whether the stage was attempted
- Whether it was successful
- Whether corresponding evidence existed in logs or network telemetry

If a stage produced no telemetry, the absence itself was treated as meaningful evidence when aligned with preventive controls.

---

### 4. Noise Identification and False Positive Avoidance

A major challenge in threat hunting is distinguishing attack activity from benign system behavior.

During analysis, multiple sources of background noise were identified, including:
- Automated cron jobs
- System maintenance tasks
- Network multicast and broadcast traffic
- Analyst-generated administrative activity (e.g., log inspection via sudo)

These events were carefully excluded from attack attribution to avoid false positives and incorrect conclusions.

---

### 5. Phase 2 Hardening Verification Through Threat Hunting

Threat hunting played a central role in validating Phase 2 hardening measures.

Rather than relying solely on the absence of successful attacks, verification focused on:
- Failed authentication attempts
- Blocked privilege escalation attempts
- Absence of lateral movement network traffic
- Correlation with Red Team execution output

This methodology allowed the analyst to confidently conclude that:
- Attack chains were interrupted early
- Preventive controls were functioning as intended
- Lack of Windows-side telemetry was expected and correct

---

### 6. Detection Gaps and Visibility Limitations

Threat hunting also enabled identification of detection gaps.

One key limitation identified was the lack of Linux process execution telemetry.  
Failed attack tooling execution (e.g., non-privileged process launches) did not generate observable events in the SIEM.

This limitation highlights an important security principle:
- Absence of detection does not imply absence of prevention
- Detection capabilities are directly tied to telemetry collection scope

Such findings provide actionable input for future monitoring improvements.

---

### 7. Role of the Threat Analyst in Security Validation

The threat analyst acts as the final validation layer between offensive simulation and defensive assurance.

Key responsibilities demonstrated in this project include:
- Correlating attack actions with system telemetry
- Validating whether attacks truly succeeded
- Preventing false conclusions based on incomplete data
- Translating technical findings into defensible security assessments

Threat hunting thus bridges the gap between Red Team activity and Blue Team confidence.

---
