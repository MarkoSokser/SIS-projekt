# Blue Team Phase 0: Monitoring Setup Summary

## Overview

Phase 0 established security monitoring infrastructure before Red Team engagement.

## Components Configured

| Component | IP | Configuration |
|-----------|----|--------------| 
| Wazuh SIEM | 10.10.0.70 | Custom rules + pfSense decoder |
| Windows Agent | 10.10.0.50 | Sysmon + Event Channel logging |
| Linux Agent | 10.10.0.51 | FIM for critical files |
| pfSense | 10.10.0.1 | Syslog forwarding to Wazuh |

## Detection Rules Deployed

11 custom detection rules covering:
- SSH brute force (T1110)
- Sudo abuse (T1548)
- PowerShell attacks (T1059, T1071)
- File integrity changes (T1222)
- Lateral movement (T1021)
- Firewall events

## Intentional Vulnerabilities Left for Testing

- `webadmin` user with weak password
- `NOPASSWD` in sudoers
- `/tmp/db_config.py` with hardcoded credentials
- Windows Firewall disabled

## Verification Tests Passed

- Sysmon events ingested by Wazuh
- FIM alerts triggered on file modification
- pfSense syslog packets received
- All agents active and reporting

## Related Files

- Full setup guide: [blue-team-pre-attack-setup.md](../../implementation/src/blue-team-engineer/blue-team-pre-attack-setup.md)
