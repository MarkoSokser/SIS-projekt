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
| Ubuntu Ranjivi Server | 10.10.0.51 |
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