# Implementation plan

## Role 1: Infrastructure Engineer

### 1. Project Role

As Infrastructure Engineer, the responsibilities included:
- design and construction of the entire virtual network for attack simulation
- operating system deployment
- implementation of vulnerabilities needed by the Red Team
- integration of monitoring and security tools
- ensuring the environment is stable, accessible, and ready for other team members


###  2. Main Components Implemented

A complete enterprise-lab was built consisting of 6 interconnected virtual machines.

####  pfSense Firewall
- configured as central network gateway
- LAN and WAN configuration
- static routes
- enabled access to internal network

####  Windows Server 2019 – Domain Controller
- created technova.local domain
- configured DNS
- set up vulnerable users (weak passwords)
- prepared AD environment for Lateral Movement

####  Windows 10 Workstation
- joined to domain
- prepared for attack scenarios (CALDERA agent, Wazuh agent)
- realistic client workstation

####  Ubuntu Vulnerable Server
- configured as target for initial foothold
- implemented vulnerabilities:
  - weak users
  - SSH password login
  - sudoers NOPASSWD
  - apache + directory listing
  - cron backdoor script
  - lack of hardening

####  CALDERA Operator VM
- installed MITRE CALDERA framework
- resolved DNS issues, emergency-shell bug, chown errors
- connected Linux and Windows agents
- environment ready for Red Team operations

####  Mini-Wazuh SIEM 
- configured minimal SIEM server
- connected Windows and Linux agents
- enabled Blue Team to monitor real attacks



### 3. Network Architecture

All VMs are connected to the same internal network **TechNovaNet** (10.10.0.0/24) with corresponding static IP addresses:

| Component | IP Address |
|-----------|-----------|
| pfSense | 10.10.0.1 |
| Domain Controller | 10.10.0.10 |
| Windows Workstation | 10.10.0.20 |
| Ubuntu Vulnerable Server | 10.10.0.51 |
| CALDERA Server | 10.10.0.53 |
| Wazuh SIEM | 10.10.0.70 |

All components communicate within this network as a real corporate LAN.



###  4. Vulnerabilities and Intentional Weaknesses

To allow the Red Team to execute MITRE ATT&CK techniques, the following were intentionally left:

✔ Weak passwords and password reuse  
✔ Password-based SSH access  
✔ Weak sudoers (NOPASSWD)  
✔ Directory listing on Apache server  
✔ Cron backdoor  
✔ Minimal network segmentation (flat network)  
✔ Default Defender and minimal Windows hardening  
✔ Unhardened AD  

This makes the network realistic and vulnerably weak, which is the project's goal.


###  5. Security Tools

#### Wazuh SIEM
- monitors Windows and Linux logs
- detects brute force, privilege escalation, file tampering
- enables Blue Team analytics

#### CALDERA Sandcat agents
- enable Red Team to simulate multi-stage attacks

####  Syslog / journald integration
- Linux logs forwarded to SIEM



### 6. Snapshot and Baseline Preparation

A "golden state" snapshot was created for all VMs:
- clean system state
- all agents installed
- all vulnerabilities active
- ready for attack repetition

This allows the team to always return to the starting point without errors.

---

## Role 2: Blue Team Engineer

### 1. Project Role

As Blue Team Engineer, the responsibilities include:
- designing monitoring and detection strategy for the TechNova lab
- preparing SIEM (Wazuh) for ingesting logs from all hosts
- defining use cases and alerts for key ATT&CK techniques
- coordinating with Red Team to understand planned attack paths
- documenting detection coverage and gaps for the final report
 - performing post-attack remediation and hardening to fix identified vulnerabilities

### 2. Main Planned Activities

- onboard all project endpoints (DC, workstation, Ubuntu, CALDERA, pfSense where possible) into Wazuh
- standardize time sync and hostname conventions to simplify correlation
- create base dashboards for authentication events, process creation and network connections
- prepare rules for detecting common attack behaviors (brute force, privilege escalation, lateral movement)
- validate that logs from CALDERA activities are visible and correctly tagged in SIEM

### 3. Pre‑Attack Monitoring Setup

Before Red Team operations start, the Blue Team will:
- verify agents are installed and sending logs from Windows and Linux hosts
- configure log retention appropriate for the exercise window
- tune out obvious noise (benign system events) to focus on attack indicators
- test a few synthetic events (failed logons, sudo misuse) to confirm alerts trigger

### 4. Coordination With Other Roles

- work with Infrastructure Engineer to ensure critical logs (Windows Security, Sysmon if available, Linux auth/journald, web server logs) are forwarded
- agree with Red Team on a high‑level attack timeline so monitoring can be focused during exercises
- share detection expectations and feedback so attacks can be replayed if needed for evidence collection

### 5. Planned Deliverables

- baseline dashboards and alert ruleset in Wazuh
- short runbook for triaging common alerts during exercises
- summary of which MITRE ATT&CK techniques are covered by detections and which remain gaps

### 6. Planned Remediation Steps

After the Red Team completes their attacks, the Blue Team will:
- harden SSH access by changing weak passwords used in brute-force scenarios
- contain compromised Windows hosts by adding pfSense firewall rules to block outbound C2 traffic
- remove insecure sudoers settings (NOPASSWD) on the Linux server to prevent easy privilege escalation
- re-enable and configure Windows firewall rules to block lateral movement from attacker infrastructure

This plan ensures the environment is not only vulnerable and attack‑ready, but also observable and measurable from the Blue Team perspective.