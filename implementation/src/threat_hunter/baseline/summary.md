# Pre-Attack Baseline – Environment Summary

## 1. Purpose and Context

This document represents the **final consolidated pre-attack baseline summary** for the Technova lab environment.
Its purpose is to provide a detailed yet readable overview of the environment state *before any red team activity*,
serving as a single reference point for later attack analysis, detection validation, and post-incident reporting.

While each system has its own dedicated baseline report, this summary:
- consolidates key findings across all systems,
- highlights interdependencies between components,
- documents known weaknesses and expected behavior,
- and establishes a clear reference against which attack activity can be evaluated.

This document should be read together with the individual baseline reports for deeper technical detail.

---

## 2. Environment Composition and Architecture

The Technova lab consists of four primary components:

1. **Windows Workstation**
   - Represents a typical end-user endpoint
   - Serves as a likely initial access vector
   - Used by both red and blue teams for operational tasks

2. **Linux Server (ubuntu-srv)**
   - Internal infrastructure system
   - Potential lateral movement target
   - Hosts privileged accounts and services

3. **pfSense Firewall**
   - Central network enforcement and routing device
   - Controls traffic between internal networks and the WAN
   - Provides network-level visibility and logging

4. **Wazuh SIEM**
   - Centralized logging and detection platform
   - Aggregates host and network telemetry
   - Used for monitoring, alerting, and analysis

The interaction between these systems forms the basis for realistic attack and detection scenarios.

---

## 3. Windows Workstation Baseline Summary

The Windows workstation baseline confirms a clean and stable endpoint state prior to attacks.

Key observations include:
- Normal user activity under the `read_team` account
- No evidence of unauthorized processes or persistence mechanisms
- Active endpoint security tooling, including:
  - Windows Defender
  - Sysmon
  - Wazuh agent
  - Tailscale client
- Stable network behavior with expected outbound traffic patterns
- Dual connectivity via internal network and Tailscale, enabling controlled remote access

The workstation baseline establishes a reliable reference for detecting:
- credential abuse,
- suspicious process creation,
- malicious persistence,
- anomalous outbound connections.

For detailed evidence and artefacts, see:
- `baseline/workstation/workstation_baseline_report.md`

---

## 4. Linux Server Baseline Summary

The Linux server baseline documents the state of an internal system that may be targeted after initial access.

Baseline findings show:
- Normal authentication activity recorded in a pre-attack snapshot of the authentication log
- Legitimate sudo usage patterns by authorized users
- An intentional privilege escalation configuration allowing passwordless sudo access for the `webadmin` user
- No indicators of brute-force attacks, persistence mechanisms, or unauthorized account creation
- Active Wazuh agent ensuring visibility of authentication and privilege escalation events

The documented privilege escalation configuration represents a known and deliberate weakness
that is expected to be leveraged during attack simulations.

This baseline enables clear identification of:
- abnormal privilege escalation,
- suspicious authentication behavior,
- post-exploitation persistence.

Detailed findings are documented in:
- `baseline/linux/linux_baseline_report.md`

---

## 5. pfSense Firewall Baseline Summary

The pfSense firewall baseline establishes expected network behavior and enforcement rules.

Key baseline characteristics include:
- Default deny posture on the WAN interface
- Explicit allow rules for trusted internal traffic
- No inbound NAT port forwarding, preventing exposure of internal services
- Normal outbound traffic patterns including HTTPS, DNS, and NTP
- Repeated blocked multicast and SSDP traffic on the WAN, identified as expected background noise
- Firewall logs enabled and configured for forwarding to the Wazuh SIEM

This baseline provides critical context for identifying:
- anomalous inbound connections,
- unexpected NAT rule changes,
- suspicious outbound communication patterns.

Full firewall configuration and log observations are available in:
- `baseline/pfsense/pfsense_baseline_report.md`

---

## 6. Wazuh SIEM Baseline Summary

The Wazuh SIEM baseline confirms that detection and monitoring capabilities are operational prior to attacks.

Baseline observations include:
- All core Wazuh services required for log ingestion and analysis are running
- All intended agents (Windows workstation, Linux server, local SIEM agent) are active and communicating
- File Integrity Monitoring (FIM) and Security Configuration Assessment (SCA) modules are functioning
- IndexerConnector warnings observed during startup are documented and understood as non-malicious
- No confirmed malicious alerts present in pre-attack alert logs

This baseline ensures confidence that alerts observed during attack phases can be attributed
to red team activity rather than pre-existing conditions.

Detailed SIEM observations are documented in:
- `baseline/wazuh/wazuh_baseline_report.md`

---

## 7. Cross-System Baseline Observations

Across the environment, the following baseline characteristics were established:

- Systems are operational, stable, and reachable
- Logging and telemetry are active at host and network levels
- Endpoint and network data converge at the SIEM
- Background noise and expected warnings are documented
- Known and intentional weaknesses are explicitly identified

These cross-system observations are critical for effective threat hunting,
as they allow analysts to distinguish between normal behavior and true anomalies.

---

## 8. Known Weaknesses and Assumptions

The following weaknesses are documented as part of the baseline and are considered in-scope
for attack simulations:

- Passwordless sudo access for a Linux administrative user
- Broad outbound network access from internal systems
- Reliance on centralized SIEM visibility
- Single firewall acting as a network choke point

These conditions are intentional within the lab and are expected to be exercised during attacks.

---

## 9. Baseline Usage During Attack Phase

This baseline summary will be used to:
- Compare pre- and post-attack system states
- Validate detections and alerts generated by Wazuh
- Support attack timeline reconstruction
- Provide context for false positives and background noise
- Serve as authoritative reference during reporting and presentation

Any deviation from the documented baseline should be treated as potentially suspicious
until validated.

---

## 10. Conclusion

This document represents the authoritative pre-attack baseline for the Technova lab environment.
Together with the individual system baseline reports, it provides a comprehensive and reliable
reference point for all subsequent attack analysis and detection evaluation activities.

All future findings, alerts, and conclusions should be evaluated against this documented state.
