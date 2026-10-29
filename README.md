# 🏠 Virtualized Enterprise Security & Monitoring Homelab

A virtualized enterprise security homelab built with **VMware Workstation, FortiGate, Active Directory, Wazuh SIEM, Splunk Enterprise, Kali Linux, Nessus, Atomic Red Team, Sigma, and Windows/Linux security telemetry**.

The lab simulates practical **SOC monitoring, network security, vulnerability management, centralized log collection, SIEM correlation, detection engineering, incident investigation, alerting, and MITRE ATT&CK-aligned security validation**.

---

## 🏗️ Lab Architecture

### Core Infrastructure

- VMware Workstation
- FortiGate Firewall
- Windows Server 2022
- Active Directory Domain Services
- Internal DNS
- Windows 11 Pro
- Ubuntu 22.04 LTS
- VMware NAT and host-only networks

### Security Monitoring and Detection

- Wazuh Manager, Indexer, and Dashboard
- Wazuh Windows and Linux agents
- Splunk Enterprise
- Splunk Universal Forwarder
- Windows Event Logs
- Sysmon telemetry
- PowerShell Operational logging
- FortiGate syslog
- Splunk dashboards, scheduled alerts, and SMTP email notifications

### Assessment and Detection Validation

- Kali Linux
- Nmap
- Wireshark
- Nessus Vulnerability Scanner
- Atomic Red Team
- Sigma CLI
- MITRE ATT&CK
- Sigma-to-Splunk rule conversion

---

## 🌐 Network Topology

![Virtualized Enterprise Security and Monitoring Homelab Topology](assets/topology.png)

### Network Summary

| Component | Address | Purpose |
|---|---:|---|
| VMware NAT gateway | `192.168.100.2` | Provides outbound internet connectivity |
| FortiGate WAN | `192.168.100.10` | Connects the firewall to VMware NAT |
| FortiGate LAN | `10.10.10.1` | LAN gateway, firewall, NAT, and DHCP |
| Internal LAN | `10.10.10.0/24` | Isolated enterprise homelab network |
| DC01 | `10.10.10.10` | Active Directory, DNS, Windows logs, and Sysmon |
| Windows 11 Renz | `10.10.10.20` | Monitored endpoint and Atomic Red Team test system |
| Wazuh Server | `10.10.10.30` | Wazuh Manager, Indexer, Dashboard, and correlation |
| Kali Linux | `10.10.10.40` | Security testing, scanning, packet analysis, and Sigma CLI |
| Linux01 | `10.10.10.50` | Linux telemetry and FortiGate syslog receiver |
| Splunk Enterprise | `10.10.10.60` | Centralized searching, dashboards, alerts, and email actions |

---

## 📡 Telemetry and Log Collection

The lab collects and centralizes telemetry from multiple operating systems and security devices:

- DC01 forwards Windows Security, System, Application, PowerShell, and Sysmon events.
- The Windows 11 endpoint forwards Windows Event Logs, Sysmon telemetry, and PowerShell activity.
- Linux01 forwards Linux authentication, SSH, sudo, and system logs.
- FortiGate sends firewall and traffic logs to the Linux syslog receiver.
- Splunk Universal Forwarders send endpoint and syslog data to Splunk Enterprise.
- Wazuh agents send Windows and Linux security events to the Wazuh Manager.
- Splunk and Wazuh provide dashboards, investigation views, correlation, and security alerts.

---

## 🔍 Detection and Alerting

The lab includes detection and investigation workflows for:

- Windows and Linux failed logins
- Successful logins following repeated failures
- Account lockouts
- Windows account and group changes
- PowerShell activity
- Sysmon process creation
- Linux SSH and sudo activity
- FortiGate denied traffic
- Vulnerability findings
- Top source and destination IP addresses
- Wazuh and Splunk agent health
- Brute-force activity with SMTP email notification

---

## 🧪 MITRE ATT&CK Detection Validation

Atomic Red Team is installed on the Windows 11 endpoint to generate controlled and repeatable security events.

The first validation test used:

```text
T1059.001 — Command and Scripting Interpreter: PowerShell
```

The test workflow included:

1. Executing controlled PowerShell and registry activity with Atomic Red Team.
2. Collecting process-creation and PowerShell telemetry with Sysmon and Windows logging.
3. Forwarding the events to Splunk Enterprise.
4. Investigating the activity by hostname, process, command line, and Event ID.
5. Creating a Sigma detection rule mapped to `T1059.001`.
6. Converting the Sigma YAML rule into Splunk SPL using Sigma CLI.
7. Testing the converted detection against the collected endpoint telemetry.
8. Documenting the rule, converted query, matching events, and cleanup results.

This process demonstrates a repeatable detection-engineering workflow:

```text
Atomic Red Team
      ↓
Windows and Sysmon telemetry
      ↓
Splunk and Wazuh
      ↓
Sigma detection logic
      ↓
Splunk investigation and alerting
```

---

## 🛡️ Current Security Capabilities

- Network segmentation and firewall policy enforcement
- Active Directory identity and authentication monitoring
- Windows and Linux endpoint telemetry
- Centralized SIEM log collection
- Brute-force and account-lockout detection
- Splunk email alerting
- Wazuh event correlation
- FortiGate traffic-log monitoring
- Vulnerability assessment with Nessus
- Network discovery with Nmap
- Packet analysis with Wireshark
- MITRE ATT&CK-aligned testing with Atomic Red Team
- Portable detection-as-code using Sigma
- Sigma-to-Splunk rule conversion and validation
