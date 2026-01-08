# Linux Attack Report (Phase 1 – Host-Level Analysis)
## Table of Contents
- [Purpose & Scope of Analysis](#1-purpose--scope-of-analysis)
- [System Context](#2-system-context)
- [Initial Access](#3-initial-access)
  - [SSH Authentication Using Valid Credentials](#31-ssh-authentication-using-valid-credentials)
  - [Session Establishment](#32-session-establishment)
- [Post-Access Activity (Noise Generation Test)](#4-post-access-activity-noise-generation-test)
  - [Authentication Flooding](#41-authentication-flooding)
  - [Failed Privilege Transitions](#42-failed-privilege-transitions)
  - [Privileged Command Execution](#43-privileged-command-execution)
- [Behavioral Comparison to Baseline](#5-behavioral-comparison-to-baseline)
- [Conclusion](#6-conclusion)

## 1. Purpose & Scope of Analysis

This report documents malicious activity observed on the Linux server (ubuntu-srv) during Phase 1 of the red team operation.  
The analysis is strictly limited to host-level evidence collected directly from the Linux system and focuses on authentication, authorization, and privilege-related activity recorded in native Linux logs.

The purpose of this report is to:
- identify attacker actions performed on the Linux host,
- correlate observed behavior with Linux authentication and authorization logs,
- compare attack activity against the previously established Linux baseline.

---

## 2. System Context

Hostname: ubuntu-srv  
Operating System: Ubuntu Server (24.04 LTS)  
Role: Internal Linux server / pivot host  

Relevant log sources:
- /var/log/auth.log
- Rotated authentication logs (/var/log/auth.log.2.gz)

At the time of the attack, the system was operational and accessible via SSH from internal network segments.

---

## 3. Initial Access

### 3.1 SSH Authentication Using Valid Credentials

Analysis of rotated authentication logs confirms a successful SSH login using valid credentials for user webadmin.

Evidence from /var/log/auth.log.2.gz:

2025-12-17T19:52:47 ubuntu-srv sshd[1881]:  
Accepted password for webadmin from 10.10.0.50 port 50213 ssh2

This activity is visually confirmed in the screenshot `log_attact.png`, which shows the extracted SSH authentication event from the rotated Linux authentication log.

This event confirms:
- use of a legitimate user account,
- successful remote access via SSH,
- absence of brute-force or authentication flooding prior to login.

This activity represents credential-based initial access consistent with baseline authentication mechanisms.

---

### 3.2 Session Establishment

Immediately following authentication, an interactive session was established.

Evidence:

pam_unix(sshd:session): session opened for user webadmin (uid=1001)

This behavior is illustrated in the screenshot `session.png`, which shows the PAM session creation event associated with the SSH login.

This confirms full shell access to the Linux system under the compromised user account.

---

## 4. Post-Access Activity (Noise Generation Test)

After establishing initial access, the attacker intentionally generated noisy activity on the Linux host in order to test logging and detection boundaries.

Observed actions include:
- repeated failed SSH authentication attempts,
- failed local user switching attempts using su,
- execution of privileged commands via sudo,
- access to sensitive system files,
- creation and execution of a non-standard systemd service.

---

### 4.1 Authentication Flooding

Multiple failed SSH authentication attempts were observed.

Example evidence:

Failed password for webadmin from 127.0.0.1 port 36246 ssh2

This authentication noise is visible in the screenshot `logs.png`, which captures repeated failed authentication attempts recorded in the Linux authentication logs.

These events exceed baseline authentication behavior and indicate intentional authentication abuse.

---

### 4.2 Failed Privilege Transitions

Failed attempts to switch users using su were recorded.

Example evidence:

FAILED SU (to webadmin) vbubuntu on pts/0

This behavior indicates unauthorized privilege transition attempts outside normal operational patterns.

---

### 4.3 Privileged Command Execution

Successful execution of privileged commands was observed, including access to sensitive credential storage files and service manipulation.

Example evidence:

sudo: webadmin : COMMAND=/usr/bin/cat /etc/shadow  
sudo: webadmin : COMMAND=/usr/bin/systemctl start rt_malware.service

These actions represent clear deviations from baseline behavior and demonstrate malicious intent, including credential access and persistence-related activity.

---

## 5. Behavioral Comparison to Baseline

Compared to the previously established Linux baseline, the following deviations were identified:

- repeated authentication failures,
- unauthorized user switching attempts,
- privileged access to credential storage files,
- execution of non-standard system services.

None of these behaviors were observed during baseline data collection.

---

## 6. Conclusion

The Linux server was successfully compromised during Phase 1 using valid user credentials, allowing the attacker to establish an initial foothold without immediately triggering suspicious behavior.

Subsequent noisy activity produced clear host-level indicators of malicious behavior, including authentication abuse, privilege escalation attempts, and persistence-related actions.

This confirms that native Linux logs alone are sufficient to reconstruct attacker activity and establish a reliable forensic timeline.
