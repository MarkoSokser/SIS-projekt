#  TechNova Cyber Range – Adversary Simulation Lab

A comprehensive cybersecurity training environment built for adversary emulation, detection engineering, and blue team/red team exercises using the **MITRE ATT&CK** framework.

---

##  Table of Contents

- [Project Overview](#project-overview)
- [Key Features](#-key-features)
- [Network Architecture](#-network-architecture)
- [Team Roles](#team-roles)
- [Project Structure](#-project-structure)
- [Technologies Used](#-technologies-used)
- [Getting Started](#-getting-started)
- [Documentation](#-documentation)
- [Disclaimer](#-disclaimer)

---

## Project Overview

**TechNova Cyber Range** is an isolated virtual lab environment designed for hands-on cybersecurity training. The project simulates a realistic enterprise network with intentional vulnerabilities, enabling:

- **Red Team** operations using MITRE CALDERA for automated adversary emulation
- **Blue Team** defense, detection, and incident response using Wazuh SIEM
- **Threat Hunting** through baseline analysis and anomaly detection
- **Security Hardening** validation through multi-phase attack simulations

The environment follows a two-phase methodology:
1. **Phase 1 (Baseline)**: Attack simulation against a deliberately vulnerable network
2. **Phase 2 (Hardening)**: Re-test after applying security controls to validate improvements

---

##  Key Features

| Feature | Description |
|---------|-------------|
|  **Realistic Enterprise Environment** | Active Directory domain, Windows workstations, Linux servers |
|  **Automated Red Team Operations** | MITRE CALDERA framework for repeatable attack chains |
|  **Centralized Monitoring** | Wazuh SIEM with custom detection rules |
|  **MITRE ATT&CK Mapping** | All attacks mapped to official techniques |
|  **Snapshot Recovery** | Quick reset to clean baseline state |
|  **Remote Access** | Tailscale VPN for secure team collaboration |

---

##  Network Architecture

All systems operate within the **TechNovaNet** internal network (`10.10.0.0/24`):

```
┌─────────────────────────────────────────────────────────────────┐
│                        TechNovaNet                              │
│                       10.10.0.0/24                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌──────────────┐    ┌──────────────┐    ┌──────────────┐     │
│   │   pfSense    │    │   Windows    │    │   Windows    │     │
│   │   Firewall   │    │   Server DC  │    │  Workstation │     │
│   │  10.10.0.1   │    │  10.10.0.10  │    │  10.10.0.20  │     │
│   └──────────────┘    └──────────────┘    └──────────────┘     │
│                                                                 │
│   ┌──────────────┐    ┌──────────────┐    ┌──────────────┐     │
│   │    Ubuntu    │    │   CALDERA    │    │    Wazuh     │     │
│   │    Server    │    │    Server    │    │     SIEM     │     │
│   │  10.10.0.51  │    │  10.10.0.53  │    │  10.10.0.70  │     │
│   └──────────────┘    └──────────────┘    └──────────────┘     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

| Component | IP Address | Purpose |
|-----------|------------|---------|
| pfSense Firewall | 10.10.0.1 | Network gateway, routing, traffic control |
| Windows Server 2019 DC | 10.10.0.10 | Domain Controller (technova.local), DNS |
| Windows 10 Workstation | 10.10.0.20 | Client endpoint, attack target |
| Ubuntu Server | 10.10.0.51 | Initial foothold target, vulnerable services |
| CALDERA Server | 10.10.0.53 | Red Team C2, adversary emulation platform |
| Wazuh SIEM | 10.10.0.70 | Centralized logging, detection, alerting |

---

## Team Roles

###  Infrastructure Engineer
Responsible for designing and deploying the complete virtual lab environment:
- Network architecture and pfSense configuration
- Active Directory and domain setup
- Vulnerability implementation for Red Team operations
- SIEM and monitoring tool integration

###  Red Team Engineer
Executes adversary emulation campaigns using MITRE CALDERA:
- Multi-stage attack chains (Initial Access → Persistence → Lateral Movement → Exfiltration)
- Custom ability development for lab-specific scenarios
- Attack documentation with MITRE ATT&CK mapping
- Hardening validation testing

###  Blue Team Engineer
Configures defenses and monitors for malicious activity:
- Wazuh SIEM configuration and custom detection rules
- Pre-attack security baseline establishment
- Real-time attack detection and alerting
- Post-attack remediation and hardening

###  Threat Hunter
Establishes baselines and hunts for anomalies:
- Pre-attack environment documentation
- Normal behavior profiling across all systems
- Attack artifact identification
- Detection gap analysis

---

##  Project Structure

```
SIS-projekt-1/
├── docs/                          # Project documentation
│   ├── plan.md                    # Implementation plan
│   ├── report.md                  # Final summary report
│   ├── theory.md                  # Theoretical background
│   └── references.md              # Reference materials
│
├── implementation/src/            # Role-specific documentation
│   ├── infrastructure-engineer/   # Lab setup guides
│   ├── red-team-engineer/         # Attack plans and execution logs
│   ├── blue-team-engineer/        # Defense setup and remediation
│   └── threat_hunter/             # Baseline reports
│
├── presentation/                  # Project presentations
│
└── results/                       # Attack results and findings
    ├── findings.md                # Summary of discoveries
    └── red_team/                  # MITRE ATT&CK layers and logs
        ├── phase1-baseline/       # Pre-hardening attack results
        └── phase2-hardening/      # Post-hardening attack results
```

---

##  Technologies Used

| Category | Technology |
|----------|------------|
| **Virtualization** | VirtualBox |
| **Firewall/Router** | pfSense |
| **Domain Services** | Windows Server 2019, Active Directory |
| **Endpoints** | Windows 10, Ubuntu Server 24.04 LTS |
| **Adversary Emulation** | MITRE CALDERA |
| **SIEM/Monitoring** | Wazuh |
| **Remote Access** | Tailscale VPN |
| **Framework** | MITRE ATT&CK |

---

##  Getting Started

### Prerequisites
- VirtualBox 7.0+ installed
- Minimum 16GB RAM (32GB recommended)
- 150-200GB+ free disk space
- Host machine with virtualization enabled (VT-x/AMD-V)


##  Documentation

| Document | Description |
|----------|-------------|
| [Implementation Plan](docs/plan.md) | Detailed role responsibilities and components |
| [Theoretical Background](docs/theory.md) | Concepts behind the lab design |
| [Final Report](docs/report.md) | Project summary and accomplishments |
| [Baseline Summary](implementation/src/threat_hunter/baseline/summary.md) | Pre-attack environment state |

---

##  Disclaimer

This environment is designed for **educational purposes only**. All vulnerabilities are intentional and exist solely within an isolated lab network. Never use these techniques, credentials, or configurations in production environments.

---
