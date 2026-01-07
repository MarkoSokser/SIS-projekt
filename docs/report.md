# Final report / summary

## Table of Contents
- [Role 1: Infrastructure Engineer – Summary](#role-1-infrastructure-engineer--summary)

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