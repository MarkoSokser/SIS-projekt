# pfSense Baseline Report
## Table of Contents
- [System Role and Context](#1-system-role-and-context)
- [Interface and Network Overview](#2-interface-and-network-overview)
- [Firewall Ruleset Baseline (pfctl -sr)](#3-firewall-ruleset-baseline-pfctl--sr)
- [NAT Configuration Baseline (pfctl -sn)](#4-nat-configuration-baseline-pfctl--sn)
- [Logging and Visibility](#5-logging-and-visibility)
- [Observed Baseline Traffic Patterns](#6-observed-baseline-traffic-patterns)
- [Security Observations and Potential Weaknesses](#7-security-observations-and-potential-weaknesses)
- [Relevance for Threat Hunting](#8-relevance-for-threat-hunting)
- [Baseline Status](#9-baseline-status)

## 1. System Role and Context
pfSense acts as the central firewall and routing component within the lab environment. It controls traffic between WAN, LAN (10.10.0.0/24), and OPT networks, and provides visibility into network-level events prior to any simulated attacks.

This baseline represents the state of pfSense before red team activity and is intended to support later correlation during threat hunting.

---

## 2. Interface and Network Overview
Interface configuration and segmentation were reviewed using the firewall rule interface.
Screenshots `Firewall_rules1.png`, `Firewall_rules2.png`, and `Firewall_rules3.png` show:
Observed interfaces:
- WAN (em0): Internet-facing interface with strict ingress filtering
- LAN (em1): Internal network (10.10.0.0/24)
- OPT1 (em2): Secondary internal segment (192.168.56.0/24)

Private and bogon networks are explicitly blocked on the WAN interface, indicating a hardened perimeter configuration.

---

## 3. Firewall Ruleset Baseline (pfctl -sr)
Key observations:
- Default deny policy for both IPv4 and IPv6 traffic
- Explicit anti-lockout rules allowing HTTPS/HTTP access to pfSense management interface
- LAN network allowed outbound access ("Default allow LAN to any")
- Strict blocking of traffic with invalid ports (port 0)
- SSH access restricted via sshguard and brute-force protection rules
- ICMP traffic tightly controlled with explicit allowances

The ruleset reflects a defensive-first posture with minimal exposed services.
These behaviors are visible in the firewall rules screenshots
(`Firewall_rules1.png`, `Firewall_rules2.png`, `Firewall_rules3.png`).

---

## 4. NAT Configuration Baseline (pfctl -sn)
NAT behavior observed:
- Outbound NAT for LAN (10.10.0.0/24) and OPT (192.168.56.0/24) networks
- Static-port NAT rules for ISAKMP/IPsec traffic
- No inbound port forwarding (no rdr rules defined)

This confirms that no internal services are directly exposed to the WAN.
This can be seen in the screenshot `nat.png`.
---

## 5. Logging and Visibility
Logging configuration:
- Firewall events logged to /var/log/filter.log
- Logs include both pass and block decisions
- Syslog service enabled (newyslog configured)
- pfSense logs intended to be forwarded to Wazuh SIEM for centralized analysis

This provides sufficient telemetry for network-based detection during attacks.

---

## 6. Observed Baseline Traffic Patterns
From filter.log:
- Regular outbound HTTPS traffic from internal hosts (10.10.0.x → TCP/443)
- DNS queries (UDP/53) to both internal resolver and external servers
- NTP traffic (UDP/123) from internal systems
- Frequent blocked multicast/SSDP traffic on WAN (expected background noise)

No anomalous inbound connections were observed in the baseline window.

---

## 7. Security Observations and Potential Weaknesses
Positive findings:
- Strong default deny posture
- No exposed services on WAN
- Extensive logging enabled
- SSH brute-force protection active

Potential risks:
- LAN allowed unrestricted outbound access (expected in lab, risky in production)
- Reliance on pfSense as single choke point increases impact if compromised
- No explicit east-west segmentation between internal subnets

---

## 8. Relevance for Threat Hunting
This pfSense baseline establishes:
- Normal firewall decision patterns
- Expected background noise and blocked traffic
- Legitimate outbound service usage

During attack simulations, deviations such as:
- New allowed inbound connections
- Unexpected NAT mappings
- Abnormal DNS or NTP behavior
can be reliably identified and correlated with host-based alerts from Wazuh.

---

## 9. Baseline Status
pfSense baseline successfully captured.
Configuration and logs provide a reliable reference point for subsequent attack-phase analysis.
