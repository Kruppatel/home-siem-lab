# Incident Report - INC-001
## Brute Force Attack Detection & Response

**Date:** June 5, 2026  
**Analyst:** Krup Patel  
**Severity:** High  
**Status:** Resolved  
**MITRE ATT&CK:** T1110 - Brute Force  

---

## Executive Summary
On June 4-5, 2026, a brute force attack was detected against a Windows 11 virtual machine (WIN-B4NBUJUI7M6) in the home SOC lab environment. The attack originated from a Kali Linux machine (192.168.64.3) and targeted the victim Windows VM (192.168.64.10). Two attack vectors were used: SSH brute force via a custom Python script and SMB brute force via CrackMapExec. The attack generated 1,132 failed login events (Event ID 4625) and triggered 4 account lockout events (Event ID 4740). A custom Elastic SIEM detection rule successfully detected the attack and generated 2 High severity alerts with a risk score of 75.

---

## Timeline of Events

| Time | Event |
|------|-------|
| Jun 4 @ 23:00 | SSH brute force attack initiated from Kali (192.168.64.3) |
| Jun 5 @ 00:50 | 1,053 failed SSH login events (Event ID 4625) recorded |
| Jun 5 @ 14:47 | SMB brute force attack initiated via CrackMapExec |
| Jun 5 @ 15:03 | 40 additional failed SMB login events recorded |
| Jun 5 @ 15:03 | Account lockout triggered (Event ID 4740) |
| Jun 5 @ 15:09 | First High severity alert fired in Kibana |
| Jun 5 @ 15:30 | Second High severity alert fired in Kibana |

---

## Attack Details

### Attack 1 - SSH Brute Force
- **Tool:** Custom Python script using Paramiko library
- **Target:** Windows 11 VM SSH service (port 22)
- **Method:** Dictionary attack using rockyou.txt wordlist
- **Result:** 1,053 failed login attempts (Event ID 4625)
- **Logon Type:** Network

### Attack 2 - SMB Brute Force  
- **Tool:** CrackMapExec
- **Target:** Windows 11 VM SMB service (port 445)
- **Method:** Dictionary attack using rockyou.txt wordlist
- **Result:** 40 failed login attempts + account lockout
- **Event IDs:** 4625 (failed login), 4740 (account lockout)
- **Logon Type:** Network

---

## Indicators of Compromise (IOCs)

| IOC | Value | Type |
|-----|-------|------|
| Attacker IP | 192.168.64.3 | IP Address |
| Victim IP | 192.168.64.10 | IP Address |
| Victim Hostname | WIN-B4NBUJUI7M6 | Hostname |
| Attack Port 1 | 22 (SSH) | Port |
| Attack Port 2 | 445 (SMB) | Port |
| Event ID | 4625 | Windows Event |
| Event ID | 4740 | Windows Event |
| Failure Reason | Unknown user name or bad password | Log Field |

---

## MITRE ATT&CK Mapping

| Technique | ID | Description |
|-----------|-----|-------------|
| Brute Force | T1110 | Adversary attempting to gain access by guessing passwords |
| Valid Accounts | T1078 | Using compromised credentials to gain access |
| Network Service Discovery | T1046 | Port scanning to identify open services |

---

## Detection Details

- **SIEM Platform:** Elastic Stack 8.13
- **Detection Rule:** Brute Force Attack - Multiple Failed Logins
- **Rule Type:** Threshold
- **Query:** event.code: 4625
- **Threshold:** >= 2 failed logins per host
- **Severity:** High
- **Risk Score:** 75
- **Alerts Generated:** 2

---

## False Positive Analysis
This alert was determined to be a **True Positive** based on:
- High volume of failed logins in short time period
- All attempts from single source IP
- Sequential password attempts matching dictionary attack pattern
- Account lockout triggered confirming brute force behavior

---

## Response Actions
1. Identified attacker IP (192.168.64.3) via Kibana Discover
2. Confirmed attack pattern via Event ID 4625 log analysis
3. Verified account lockout via Event ID 4740
4. Analyzed logon type (Network) confirming remote attack vector
5. Documented all IOCs for threat intelligence
6. Mapped attack to MITRE ATT&CK framework

---

## Lessons Learned
- SMB (port 445) brute force generated account lockout events faster than SSH
- CrackMapExec was more effective at triggering Windows security events than SSH tools
- Winlogbeat successfully shipped all Windows security events to Elasticsearch in real time
- Custom threshold detection rule successfully identified attack pattern

---

## Recommendations
1. Implement strong account lockout policy (3-5 attempts)
2. Enable Multi-Factor Authentication (MFA) on all accounts
3. Block SSH and SMB access from unauthorized IP ranges at firewall
4. Implement geo-blocking for remote access services
5. Set up automated IP blocking when brute force detected
6. Regular password audits to identify weak credentials
