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


