cat > ~/home-siem-lab/PROJECT-SUMMARY.md << 'EOF'
# Project Summary - Home SOC Lab
## Threat Detection & Incident Response using Elastic SIEM

---

## What is this project?

This project is a fully functional home Security Operations Center (SOC) lab built from scratch on an Apple Silicon Mac. The goal was to simulate a real corporate environment where an attacker attempts to break into a Windows machine, and a SOC analyst (me) detects, investigates, and documents the attack using professional tools.

Everything in this lab mirrors what happens in real SOC environments every single day.

---

## Why I Built This

Breaking into cybersecurity without hands-on experience is difficult. Certifications teach theory but employers want proof that you can actually do the work. This lab was built to:
- Get real hands-on experience with enterprise SIEM tools
- Understand how attackers think and operate
- Practice the full SOC analyst workflow from detection to documentation
- Build a portfolio project that demonstrates practical skills

---

## The Lab Environment

Think of this as a mini corporate network running inside a Mac:
[Kali Linux VM]          [Mac Host]           [Windows 11 VM]
192.168.64.3    <-->  192.168.64.1   <-->   192.168.64.10
(Attacker)           (SIEM/Elastic)          (Victim)


**Why these tools?**
- **UTM** — Best free virtualization tool for Apple Silicon Macs
- **Kali Linux** — Industry standard for penetration testing and attack simulation
- **Windows 11** — Most common corporate target in real attacks
- **Elastic Stack** — Used by thousands of companies as their SIEM platform

---

## The Full Story

### Chapter 1 — Building the Lab
The first challenge was setting up two virtual machines on an Apple Silicon Mac. This required using UTM instead of VirtualBox because Apple Silicon uses ARM architecture. Configuring the network so both VMs could communicate with each other and with the Elastic Stack running on the Mac host took careful planning.

**Key decisions:**
- Used Shared Network mode in UTM for VM connectivity
- Set static IP on Windows VM to ensure consistent addressing
- Installed Elastic Stack directly on Mac host for better performance

### Chapter 2 — Setting Up the SIEM
Elastic Stack was installed and configured on the Mac host. This involved:
- Installing Elasticsearch (the log database)
- Installing Kibana (the dashboard)
- Configuring SSL/TLS security
- Setting up encryption keys for alert persistence

**Key insight:** Elasticsearch 8.x has security enabled by default which required careful configuration of certificates and authentication.

### Chapter 3 — Connecting Log Sources
Two Beat agents were deployed to collect logs:
- **Filebeat** on Mac — collects system logs
- **Winlogbeat** on Windows VM — collects Windows Event Logs

**Why Winlogbeat is critical:** Windows security events like failed logins (4625) and account lockouts (4740) are only available through Windows Event Log. Winlogbeat ships these to Elasticsearch in real time, giving the SOC analyst visibility into everything happening on the Windows machine.

### Chapter 4 — The Attack

**Attack 1: SSH Brute Force**
A custom Python script was written using the Paramiko SSH library. The script automated login attempts using the rockyou.txt password wordlist — the same wordlist real attackers use. 500 passwords were tried against the Windows SSH service.

*Why custom Python?* Writing the script myself demonstrates programming ability and shows I understand how brute force attacks work at a technical level, not just conceptually.

**Attack 2: SMB Brute Force**
CrackMapExec was used to attack Windows SMB (port 445). SMB is one of the most commonly attacked services in real corporate environments — it was the vector used in the famous WannaCry ransomware attack. This attack ultimately triggered an account lockout, generating Event ID 4740.

*Why SMB?* Port scanning with nmap revealed port 445 was open. A real attacker would identify this as a high-value target since SMB vulnerabilities have historically led to major breaches.

### Chapter 5 — Detection
All attack logs flowed into Elasticsearch via Winlogbeat. In Kibana Discover we could see:
- 1,132 failed login events (Event ID 4625)
- Source IP 192.168.64.3 (Kali) attacking 192.168.64.10 (Windows)
- Failed reason: "Unknown user name or bad password"
- Logon type: Network (confirming remote attack)
- 4 account lockout events (Event ID 4740)

### Chapter 6 — Building the Detection Rule
A custom threshold detection rule was created in Kibana Security:
- Monitors for Event ID 4625 across all hosts
- Triggers when 2 or more events occur on same host
- Severity: High | Risk Score: 75
- Runs every 5 minutes automatically

**Result:** 2 High severity alerts fired successfully proving the SIEM detected the attack.

### Chapter 7 — Investigation
The alerts were investigated like a real SOC analyst:
1. Opened alert in Kibana
2. Identified attacker IP from source.ip field
3. Confirmed attack pattern from log volume and timing
4. Identified attack vector (Network logon = remote SMB attack)
5. Mapped to MITRE ATT&CK T1110 (Brute Force)
6. Determined True Positive vs False Positive

### Chapter 8 — Documentation
A formal incident report was written following industry standards including executive summary, timeline, IOCs, MITRE ATT&CK mapping, false positive analysis, response actions, and recommendations.

---

## Challenges & How I Solved Them

| Challenge | Solution |
|-----------|----------|
| VirtualBox doesn't support Apple Silicon | Switched to UTM which natively supports ARM |
| Kali Linux display not working in UTM | Added Serial device to VM settings |
| Windows not getting IPv4 address | Set static IP manually via netsh command |
| Elasticsearch Java error | Symlinked Homebrew Java to Elasticsearch JDK path |
| Winlogbeat couldn't reach Kibana | Changed Kibana server.host to 0.0.0.0 |
| SSH brute force not generating logs | Switched to SMB attack with CrackMapExec |
| Alerts not firing | Added encryption key to kibana.yml |

---

## Key Results

| Metric | Result |
|--------|--------|
| Failed login events detected | 1,132 |
| Account lockout events | 4 |
| High severity alerts fired | 2 |
| Detection rule risk score | 75 |
| MITRE ATT&CK techniques mapped | 3 |
| Time to detect after attack | < 5 minutes |

---

## What I Learned

**Technical skills:**
- Configuring and managing Elastic Stack SIEM
- Writing custom detection rules using KQL
- Log analysis and threat hunting in Kibana
- Windows Event Log forensics (Event IDs 4625, 4740, 4624)
- Network security fundamentals (SMB, SSH, port scanning)
- Python scripting for security automation
- Linux and Windows administration

**Analyst skills:**
- Full SOC workflow from alert to incident report
- True positive vs false positive analysis
- IOC identification and documentation
- MITRE ATT&CK framework application
- Professional incident report writing

---

## Real World Relevance

Everything done in this lab happens in real SOC environments:
- Companies use Elastic SIEM (or similar) to monitor thousands of machines
- Brute force attacks are one of the most common attack types SOC teams see daily
- Event ID 4625 and 4740 are among the first events SOC analysts learn to monitor
- Custom detection rules are written by SOC engineers to catch specific threats
- Incident reports are written for every significant security event

This lab proves I can perform real SOC analyst work, not just pass certification exams.
EOF
