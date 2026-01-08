# Workstation Baseline Report (Pre-Attack)
## Table of Contents
- [Purpose & Role of the System](#1-purpose--role-of-the-system)
- [Baseline Overview (Pre-Attack)](#2-baseline-overview-pre-attack)
  - [Operating System and Identity](#21-operating-system-and-identity)
  - [Users and Groups](#22-users-and-groups)
  - [Processes and Services](#23-processes-and-services)
  - [Network Configuration](#24-network-configuration)
- [Collected Evidence](#3-collected-evidence)
- [Baseline Behavioral Characteristics](#4-baseline-behavioral-characteristics)
- [Key Detection Points](#5-key-detection-points)
- [Identified Vulnerabilities and Limitations](#6-identified-vulnerabilities-and-limitations)

## 1. Purpose & Role of the System

The Windows workstation (`DESKTOP-LBF7A9D`) serves as a primary user endpoint within the Technova lab environment and represents a realistic corporate workstation scenario.

Its role within the project is especially important because user workstations are commonly targeted as the initial entry point during real-world cyber attacks. This system interacts both with internal infrastructure components (domain services, SIEM, firewall) and with external networks, making it a valuable observation point for threat hunting and incident detection.

The workstation is domain-joined (`technova.local`) and accessed remotely via Remote Desktop Protocol (RDP), primarily through a Tailscale overlay network. This access model reflects a modern remote-access scenario often seen in enterprise environments.

---

## 2. Baseline Overview (Pre-Attack)

### 2.1 Operating System and Identity

- Operating System: Windows 10 (Build 19044)
- Hostname: `DESKTOP-LBF7A9D`
- Domain: `technova.local`
- System Role: Standard domain-joined user workstation

At the time of baseline collection, the system was operating normally with no signs of instability or compromise.

---

### 2.2 Users and Groups

The following baseline user configuration was identified:

- `read_team` – standard domain user account  
  - Used for non-privileged baseline data collection  
  - Not a member of the local Administrators group

In addition, several privileged local and domain groups were present, including:

- Administrators  
- Remote Desktop Users  
- Domain Users  

This separation between privileged and non-privileged accounts confirms the use of a least-privilege model and also explains visibility limitations encountered during baseline collection (e.g., restricted access to Security Event Logs).

---

### 2.3 Processes and Services

Process and service enumeration showed only expected and legitimate activity. Key security-relevant processes observed during baseline include:

- `wazuh-agent.exe` – endpoint telemetry forwarding to the SIEM
- `Sysmon64.exe` – enhanced host-based event logging
- `tailscaled.exe` – overlay network connectivity
- `MsMpEng.exe` – Microsoft Defender Antivirus engine

No unknown, unsigned, or suspicious processes were observed at baseline. All core Windows and security services were running as expected.

---

### 2.4 Network Configuration

The workstation operates with a dual network configuration:

- Internal LAN interface: `10.10.0.50/24`, routed through the pfSense firewall
- Tailscale overlay interface: `100.73.38.37/32`

Baseline network activity consisted primarily of:

- outbound HTTPS traffic,
- DNS resolution,
- RDP connections,
- continuous communication between the Wazuh agent and the SIEM manager.

This configuration is consistent with the intended lab design and provides multiple telemetry sources for later correlation during attack scenarios.

---

## 3. Collected Evidence

As part of the pre-attack baseline phase, the following artefacts were collected from the workstation:

- System information snapshot (`systeminfo.txt`)
- User account listing (`users.txt`)
- Local and domain group listing (`groups.txt`)
- Running process snapshots:
  - `processes.txt`
  - `wmic_process_snapshot.txt`
- Service state listings (`services.txt`)
- Startup program configuration (`startup_programs.txt`)
- Network configuration and active connections:
  - `network_config.txt`
  - `netstat.txt`
- Windows Defender status and configuration (`defender_status.txt`)
- Loaded driver inventory (`drivers.txt`)
- Environment variable snapshot (`env.txt`)
- Registry hive exports:
  - `reg_SAM.reg`
  - `reg_SYSTEM.reg`
  - `reg_SOFTWARE.reg`
- Windows Event Logs:
  - Application (`Application.evtx`)
  - System (`System.evtx`)
  - Security (`Security.evtx`)
- File integrity baseline:
  - SHA-256 hashes of `C:\Windows\System32` files (`system32_hashes.txt`)

These artefacts represent a comprehensive snapshot of the workstation state prior to any red team activity and will be used as reference material for post-attack comparison.

---

## 4. Baseline Behavioral Characteristics

During baseline observation, the workstation exhibited stable and predictable behavior consistent with a normal user endpoint:

- No abnormal spikes in CPU or memory usage
- No unusual process creation events
- No excessive authentication failures
- RDP usage consistent with administrative or analyst access
- No anomalous outbound network connections

This behavior establishes a clean reference state and provides a reliable foundation for identifying deviations during attack execution.

---

## 5. Key Detection Points

Based on the baseline analysis, the following areas were identified as high-value detection points for future threat hunting:

- Process creation events, especially involving PowerShell, command shells, or scripting engines
- RDP authentication and session activity
- Changes to startup programs or Windows services
- Modifications to critical registry locations
- Windows Defender status changes or tampering
- Outbound network connections to unusual or unexpected destinations

These points will be directly referenced during attack-phase analysis and post-incident review.

---

## 6. Identified Vulnerabilities and Limitations

The following weaknesses and limitations were identified as part of the baseline assessment:

- RDP exposure on the workstation, which is intentional for lab access but represents a realistic attack surface
- Dual-homed network configuration (LAN and overlay network), increasing exposure complexity
- Standard users lack access to Security Event Logs without administrative privileges
- Detection capabilities rely on host-level telemetry and centralized SIEM correlation

These observations are not considered misconfigurations within the lab context but are important to document for accurate threat modeling and analysis.
