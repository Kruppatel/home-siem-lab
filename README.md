# Home SOC Lab - Threat Detection & Incident Response

## Project Overview
Built a home Security Operations Center (SOC) lab using Elastic Stack (ELK) on Apple Silicon Mac with UTM virtualization. Simulated real-world attacker/victim scenarios and detected threats using custom SIEM detection rules.

## Architecture
Kali Linux (Attacker) → SSH/SMB Brute Force → Windows 11 VM (Victim)
↓
Winlogbeat (Log Shipper)
↓
Elasticsearch (Log Storage)
↓
Kibana (SIEM Dashboard)
↓
SOC Analyst (Investigation)

## Tools Used
- **Elastic Stack 8.13** - SIEM Platform
- **Elasticsearch** - Log storage and search
- **Kibana** - Dashboard and alerting
- **Filebeat** - Mac log collection
- **Winlogbeat** - Windows Event Log collection
- **UTM** - Virtualization on Apple Silicon
- **Kali Linux** - Attack simulation
- **Windows 11 ARM** - Target machine
- **CrackMapExec** - SMB brute force tool
- **Python/Paramiko** - Custom brute force script

## Attack Scenarios
### 1. SSH Brute Force Attack
- Tool: Custom Python script (Paramiko)
- Target: Windows 11 VM (192.168.64.10)
- Result: 1053 failed login events (Event ID 4625)

### 2. SMB Brute Force Attack
- Tool: CrackMapExec
- Target: Windows 11 VM (192.168.64.10)
- Result: 40 failed login events (Event ID 4625)
- Account lockout detected (Event ID 4740)
- Triggered High severity alert in Kibana

## Detection Rules
### Brute Force Detection Rule
- Type: Threshold
- Query: event.code: 4625
- Threshold: >= 2 events per host
- Severity: High
- Risk Score: 75

## Key Findings
- Detected 1,053 failed SSH login attempts (Event ID 4625)
- Detected 40 failed SMB login attempts (Event ID 4625)
- 4 account lockout events triggered (Event ID 4740)
- Source IP identified: 192.168.64.3 (Kali Linux)
- Target: WIN-B4NBUJUI7M6 (Windows VM)
- SSH attack: Interactive logon type
- SMB attack: Network logon type
- High severity alert fired in Kibana
