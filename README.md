# Virtualized Enterprise Security and Monitoring Homelab

A multi-host enterprise security homelab built with VMware Workstation, FortiGate, Active Directory, Wazuh, Splunk Enterprise, Kali Linux, Nessus, Sysmon, auditd, Atomic Red Team, Sigma, Docker, Nginx, OWASP Juice Shop and Burp Suite Community Edition.

The environment simulates SOC monitoring, centralized telemetry, firewall administration, identity security, vulnerability assessment, detection engineering, alerting, incident investigation, containment, recovery and repeatable NIST CSF 2.0 and MITRE ATT&CK validation.

## Architecture

![Enterprise security homelab topology](assets/topology.png)

| System | Address | Role |
|---|---:|---|
| FortiGate VM | `10.10.10.1` | LAN gateway, DHCP, firewall, NAT and syslog source |
| DC01 | `10.10.10.10` | Windows Server, AD DS, DNS, Sysmon, SMB/File Share auditing, Wazuh agent and Splunk Universal Forwarder |
| Renz | `10.10.10.20` | Windows 11 domain endpoint, Sysmon process/network telemetry, PowerShell, Task Scheduler, Active Directory discovery, SMB validation and SIEM agents |
| Wazuh | `10.10.10.30` | Manager, indexer, dashboard, correlation and endpoint alerting |
| Kali Linux | `10.10.10.40` | Nmap, Wireshark, Nessus, Hydra and validation tooling |
| Linux01 | `10.10.10.50` | Ubuntu telemetry host, auditd, FortiGate syslog receiver, Wazuh agent, Splunk Universal Forwarder and controlled agent-health validation |
| Splunk Enterprise | `10.10.10.60` | Central search, dashboards, scheduled alerts and SMTP email actions |
| WEB01 | `10.10.10.70` | Ubuntu web-security target with Nginx, Docker, OWASP Juice Shop, Wazuh Agent and Splunk Universal Forwarder |

### Network design

- VMware NAT network: `192.168.100.0/24`
- VMware NAT gateway: `192.168.100.2`
- FortiGate WAN: `192.168.100.10/24`
- Security lab LAN: `10.10.10.0/24`
- FortiGate LAN gateway: `10.10.10.1`
- Internal DNS: `10.10.10.10`
- DHCP and endpoint reservations managed by FortiGate
- Juice Shop is contained on WEB01 loopback port `3000`; Nginx is the only LAN-facing entry point on port `80`
- No WAN port-forward exposes the intentionally vulnerable web target

## Implemented capabilities

### Network and identity

- FortiGate firewalling, NAT, DHCP, routing and traffic logging
- Windows Server Active Directory Domain Services and internal DNS
- Windows 11 domain client and centralized account administration
- VMware WAN and host-only security-lab segmentation

### Telemetry and SIEM

- Windows Security, System, Application, PowerShell Operational and Task Scheduler Operational event collection
- Sysmon process, network, registry and file telemetry
- Linux authentication, system and auditd logging
- FortiGate syslog collection through Linux01
- Splunk Universal Forwarders and Wazuh agents on Windows and Linux endpoints
- Splunk dashboards, Wazuh threat hunting, scheduled alerts and SMTP notifications
- Dedicated WEB01 Nginx access/error telemetry forwarded to Wazuh and the Splunk `web` index

### Detection and investigation

- Windows and Linux authentication monitoring
- SSH password-guessing and brute-force detection
- Windows account lockout, account lifecycle and privileged-group monitoring
- PowerShell and Sysmon process investigation
- Windows service-installation monitoring
- Windows scheduled-task creation and deletion monitoring
- Windows Registry Run-key persistence monitoring
- Windows file creation and deletion monitoring through Sysmon Event `11` and Security Events `4663`/`4660`
- Active Directory account-discovery monitoring through Sysmon process creation and PowerShell script-block logging
- SMB network-logon and administrative-share monitoring through Events `4624`, `5140`, `5145` and Sysmon Event `3`
- Linux sudo, SSH, execution and file-deletion activity
- Linux security-agent interruption and recovery monitoring
- FortiGate denied and local-in traffic analysis
- Nessus vulnerability scanning and remediation evidence
- OWASP Juice Shop request inspection through Burp Suite Proxy, HTTP history and Repeater
- Exposed-directory and document-access monitoring for `/ftp/` resources
- Web authentication-failure monitoring, five-event threshold correlation and post-failure successful-login analysis
- Source IP, destination IP, user, host, port, URI, HTTP status and timeline correlation

## Detection engineering and validation

- Executed controlled Atomic Red Team tests mapped to MITRE ATT&CK
- Authored Sigma detections and converted them to Splunk SPL
- Validated Sysmon Event IDs `1`, `11` and `12-14`
- Validated PowerShell Operational Event IDs `4103` and `4104`
- Validated Windows service installation through Security Event `4697`, System Event `7045` and related Sysmon telemetry
- Validated scheduled-task activity through Security Events `4698`/`4699`, Task Scheduler Events `106`/`141` and Sysmon process creation
- Validated Registry Run-key persistence through Sysmon Events `12` and `13`, Wazuh threat hunting and Splunk correlation
- Validated Linux defense impairment by stopping the Wazuh Agent while auditd and Splunk remained active, then verifying automated recovery
- Validated controlled Windows file deletion using a SHA-256 baseline, scoped Object Access auditing, Security Events `4663`/`4660`, Wazuh review and Splunk correlation
- Validated domain-account discovery using `net user /domain`, `Get-ADUser`/ADSI, Sysmon Event `1`, PowerShell Event `4104`, Wazuh and Splunk
- Validated controlled SMB/IPC$ access from Renz to DC01 using network-logon, share-access and Sysmon network-connection telemetry
- Deployed OWASP Juice Shop behind Nginx on WEB01 with Docker loopback binding and dual Wazuh/Splunk telemetry
- Validated OWASP 2025 A01/A02 exposed-resource access through Burp Suite and correlated HTTP `200` responses in Nginx, Wazuh and Splunk
- Authored Wazuh web rules and a Splunk scheduled alert for `/ftp` directory or document access
- Validated OWASP 2025 A07 by replaying a bounded five-request HTTP `401` sequence and confirming the absence of rate limiting or temporary lockout
- Correlated repeated Juice Shop authentication failures with Wazuh rules `119972`/`119973` and a high-severity Splunk alert
- Built Splunk investigations, dashboards, scheduled alerts and email notifications
- Documented cleanup, containment, recovery and lessons learned

## NIST CSF 2.0 and MITRE ATT&CK exercises

| Exercise | ATT&CK mapping | Evidence |
|---|---|---|
| [SSH Brute Force Detection](tests/T1110-ssh-password-guessing/NIST-MITRE-SSH-Brute-Force-Detection.md) | `T1110`, `T1110.001` | Hydra, Linux authentication logs, Wazuh, Splunk, email, containment and recovery |
| [Network Service Discovery](tests/T1046-network-service-discovery/NIST-MITRE-Nmap-Network-Service-Discovery.md) | `T1046` | Nmap, PCAP, Linux, Wazuh, Splunk, FortiGate, email, containment and recovery |
| [PowerShell Execution Validation](tests/T1059-001-powershell-execution-validation/NIST-MITRE-PowerShell-Execution-Validation.md) | `T1059.001` | Atomic Red Team, Sysmon, PowerShell logging, Splunk correlation, dashboard, alert, email and cleanup |
| [Windows Service Creation](tests/T1543-003-windows-service-creation/NIST-MITRE-Windows-Service-Creation.md) | `T1543.003` | Controlled service installation, Events 4697/7045, Sysmon, Wazuh, Splunk, dashboard, email and recovery |
| [Windows Scheduled Task](tests/T1053-005-windows-scheduled-task/NIST-MITRE-Windows-Scheduled-Task.md) | `T1053.005` | Controlled task creation, Events 4698/106, Sysmon, Wazuh, Splunk dashboard, high-severity alert, email and recovery |
| [Registry Run Keys](tests/T1547-001-registry-run-keys/NIST-MITRE-Registry-Run-Keys.md) | `T1547.001` | Controlled HKCU Run-value creation, Sysmon Events 12/13, Wazuh, Splunk dashboard, high-severity alert, email and recovery |
| [Linux Impair Defenses](tests/T1562-001-impair-defenses/NIST-MITRE-Linux-Impair-Defenses.md) | `T1562.001` | Controlled Wazuh Agent interruption, systemd and auditd evidence, independent Splunk detection, alert, email and recovery |
| [Windows File Deletion](tests/T1070-004-windows-file-deletion/NIST-MITRE-Windows-File-Deletion.md) | `T1070.004` | Controlled marker-file deletion, SHA-256 baseline, Events 4663/4660, Wazuh investigation, Splunk dashboard, alert, email and cleanup |
| [Domain Account Discovery](tests/T1087-002-domain-account-discovery/NIST-MITRE-Domain-Account-Discovery.md) | `T1087.002` | Native and PowerShell/ADSI account discovery, Sysmon Event 1, PowerShell Event 4104, Wazuh, Splunk dashboard, alert, email and recovery |
| [SMB/Windows Admin Shares](tests/T1021-002-smb-windows-admin-shares/NIST-MITRE-SMB-Windows-Admin-Shares.md) | `T1021.002` | Controlled IPC$ connection, Events 4624/5140/5145, Sysmon Event 3, source/destination correlation, Wazuh, Splunk dashboard, alert, email and cleanup |

## OWASP web application security exercises

| Exercise | OWASP mapping | Evidence |
|---|---|---|
| [Juice Shop exposed-resource detection](tests/owasp-top-10-2025/A01-A02-exposed-resources/OWASP-A01-A02-Exposed-Resources-Validation.md) | **A01:2025 Broken Access Control**, **A02:2025 Security Misconfiguration** | WEB01, Docker, Nginx, Burp Suite, exposed `/ftp/` resources, Wazuh custom rules, Splunk investigation, triggered alert and email notification |
| [Authentication failure and rate-limit validation](tests/owasp-top-10-2025/A07-authentication-failures/OWASP-A07-Authentication-Failures-Validation.md) | **A07:2025 Authentication Failures** | Controlled HTTP `401` sequence, immediate post-failure `200`, Nginx, Wazuh rules `119972`/`119973`, Splunk threshold alert and SMTP notification |

## Completed response workflows

### SSH password guessing

1. Generated a bounded password-guessing test from Kali using a fictional account.
2. Verified failed SSH events in Linux authentication logs.
3. Correlated the source IP, destination IP, user and attempt count in Wazuh and Splunk.
4. Triggered a Splunk alert and SMTP email notification.
5. Contained the source and verified service recovery.

### Network service discovery

1. Performed a controlled Nmap SYN and version-detection scan.
2. Captured and analyzed packets with tcpdump and Wireshark.
3. Investigated related Linux, Wazuh, Splunk and FortiGate telemetry.
4. Triggered a T1046 Splunk alert and email notification.
5. Tested containment and restored connectivity.

### PowerShell execution validation

1. Established Sysmon and PowerShell telemetry baselines on the Windows 11 endpoint.
2. Reviewed and executed an authorized Atomic Red Team T1059.001 test.
3. Correlated process creation, script block, registry and file-creation telemetry in Splunk.
4. Created a dashboard panel and high-severity scheduled alert.
5. Verified the email workflow and removed the Atomic test artifacts.

### Windows service creation

1. Established Windows service and SIEM telemetry baselines on the Windows 11 endpoint.
2. Enabled service-installation auditing and created a harmless manual-start test service.
3. Validated Security Event `4697`, System Event `7045` and related Sysmon activity.
4. Correlated the service name, binary path, user, host and event timeline in Wazuh and Splunk.
5. Added a Splunk dashboard panel, high-severity scheduled alert and SMTP notification.
6. Removed the test service and verified service and registry-key recovery.

### Windows scheduled task

1. Enabled Other Object Access Events auditing and the Task Scheduler Operational log.
2. Established a scheduled-task baseline and created a harmless one-time test task.
3. Validated Security Event `4698`, Task Scheduler Event `106` and Sysmon `schtasks.exe` process telemetry.
4. Correlated the task name, event ID, process, command line, host and timestamp in Wazuh and Splunk.
5. Added a Splunk dashboard panel, high-severity scheduled alert and SMTP email notification.
6. Deleted the task, reviewed Events `4699`/`141`, and verified agent, forwarder and SIEM connectivity.

### Registry Run-key persistence

1. Established a baseline of `HKCU\Software\Microsoft\Windows\CurrentVersion\Run`.
2. Verified Sysmon registry telemetry with a temporary benign key.
3. Created a harmless Run value named `T1547.001-LabRunKey` pointing to `cmd.exe /c exit 0`.
4. Correlated Sysmon Event `13`, the process, user, registry target and value data in Wazuh and Splunk.
5. Added a Splunk dashboard panel, high-severity scheduled alert and SMTP email notification.
6. Removed only the test value, reviewed Sysmon deletion telemetry and verified endpoint and SIEM recovery.

### Linux impair defenses

1. Established service, Wazuh connection and Linux log-collection baselines on Linux01.
2. Scheduled an automatic systemd recovery timer before stopping the Wazuh Agent.
3. Confirmed that auditd and the Splunk Universal Forwarder remained active during the controlled interruption.
4. Correlated the stop command, systemd service lifecycle, audit records and controlled marker in Splunk.
5. Documented that Wazuh could not reliably report its own stop command while impaired, demonstrating the value of independent telemetry.
6. Added a Splunk dashboard panel, high-severity alert and SMTP notification.
7. Verified Wazuh reconnection, SIEM connectivity, enabled services and the absence of failed systemd units.

### Windows file deletion

1. Established Windows endpoint, monitoring-service, event-channel and Splunk telemetry baselines.
2. Confirmed the active Sysmon configuration did not include FileDelete rules and preserved the existing configuration.
3. Enabled Windows File System auditing and applied a deletion SACL only to `C:\Lab-T1070`.
4. Created a harmless marker file, recorded its SHA-256 hash and confirmed Sysmon Event `11` for file creation.
5. Deleted only the marker file and validated Security Event `4663` with DELETE access and related Event `4660` activity.
6. Investigated the event in Wazuh and Splunk, then added a dashboard panel, high-severity scheduled alert and SMTP notification.
7. Removed the temporary SACL and test folder and verified Sysmon, Wazuh and the Splunk Forwarder remained healthy.

### Domain account discovery

1. Verified Windows domain membership, DNS resolution, LDAP/SMB connectivity to DC01 and endpoint monitoring services.
2. Established Security, Sysmon and PowerShell event-channel and Splunk telemetry baselines.
3. Executed the read-only `net user /domain` command and validated Sysmon Event `1` with the process, parent process, user and command line.
4. Performed a second read-only discovery test with `Get-ADUser` or ADSI and validated PowerShell Event `4104`.
5. Correlated both discovery methods in Wazuh and Splunk and mapped the evidence to `T1087.002`.
6. Added a combined Splunk dashboard panel, high-severity scheduled alert and SMTP notification.
7. Verified the domain secure channel, Wazuh/Splunk connectivity and endpoint monitoring-service health.

### SMB/Windows Admin Shares

1. Verified Renz and DC01 identity, domain membership, TCP `445` connectivity, monitoring services and Splunk ingestion.
2. Enabled File Share, Detailed File Share and successful Logon auditing on DC01.
3. Created a controlled, read-only `\\DC01\IPC$` connection from Renz using the currently authenticated domain account.
4. Validated DC01 Security Event `4624` Logon Type `3`, Event `5140` share access and Event `5145` detailed share access.
5. Reviewed Renz Sysmon Event `3` for the outbound connection to `10.10.10.10:445` and correlated source/destination evidence in Wazuh and Splunk.
6. Added a Splunk dashboard panel, high-severity scheduled alert and SMTP notification.
7. Removed the IPC$ test session and verified domain trust, monitoring services and SIEM connectivity.

### OWASP Juice Shop exposed-file access

1. Built WEB01 as a contained Ubuntu web target and installed Docker, Nginx, the Wazuh Agent and Splunk Universal Forwarder.
2. Ran OWASP Juice Shop on `127.0.0.1:3000` and exposed it to the lab only through Nginx on `10.10.10.70:80`.
3. Configured Burp Suite Community Edition on Kali and inspected authorized HTTP traffic with Proxy history and Repeater.
4. Confirmed unauthenticated HTTP `200` access to the `/ftp/` directory and `/ftp/acquisitions.md`.
5. Correlated the source IP, URI, status code and user agent in Nginx, Wazuh and the Splunk `web` index.
6. Added Wazuh rule `119971` and a high-severity Splunk alert, then verified triggered-alert and SMTP email workflows.
7. Documented OWASP 2025 A01/A02 mappings, containment controls, remediation guidance and evidence-handling requirements.

### OWASP Juice Shop authentication failures

1. Established a successful-login baseline with a fictional lab-only Juice Shop account.
2. Captured and replayed `POST /rest/user/login` requests through Burp Suite Repeater.
3. Sent five bounded invalid-password requests and recorded HTTP `401` responses without an HTTP `429`, progressive delay or temporary lockout.
4. Confirmed an immediate valid login returned HTTP `200` after the failure sequence.
5. Correlated the source IP, method, URI, status, failure count and timeline in Nginx and Splunk.
6. Added Wazuh rule `119972` for individual failures and rule `119973` for five failures from one source within five minutes.
7. Triggered a high-severity Splunk alert, restored DNS/SMTP egress, validated email notification and confirmed application and monitoring-service recovery.

## Documentation

- [Architecture](docs/architecture.md)
- [VMware network](docs/vmware-network.md)
- [FortiGate configuration](docs/fortigate-configuration.md)
- [Firewall policies](docs/firewall-policies.md)
- [Active Directory](docs/active-directory.md)
- [Windows client](docs/windows-client.md)
- [Wazuh configuration](docs/wazuh-configuration.md)
- [Splunk configuration](docs/splunk-configuration.md)
- [Sysmon telemetry](docs/sysmon-telemetry.md)
- [auditd configuration](docs/auditd-configuration.md)
- [Web application security lab](docs/web-application-security-lab.md)

## Repository structure

```text
.
├── README.md
├── assets/
│   ├── topology.png
│   └── topology.svg
├── docs/
│   ├── architecture.md
│   ├── active-directory.md
│   ├── auditd-configuration.md
│   ├── fortigate-configuration.md
│   ├── firewall-policies.md
│   ├── splunk-configuration.md
│   ├── sysmon-telemetry.md
│   ├── vmware-network.md
│   ├── wazuh-configuration.md
│   ├── web-application-security-lab.md
│   └── windows-client.md
├── screenshots/
│   └── platform and infrastructure evidence
└── tests/
    ├── T1046-network-service-discovery/
    ├── T1053-005-windows-scheduled-task/
    ├── T1059-001-powershell-execution-validation/
    ├── T1070-004-windows-file-deletion/
    ├── T1087-002-domain-account-discovery/
    ├── T1110-ssh-password-guessing/
    ├── T1543-003-windows-service-creation/
    ├── T1547-001-registry-run-keys/
    ├── T1562-001-impair-defenses/
    ├── T1021-002-smb-windows-admin-shares/
    └── owasp-top-10-2025/
        ├── README.md
        ├── shared-config/
        │   ├── nginx-juice-shop.conf
        │   ├── splunk-forwarder-inputs.conf
        │   └── wazuh-agent-localfile.xml
        ├── A01-A02-exposed-resources/
        │   ├── OWASP-A01-A02-Exposed-Resources-Validation.md
        │   ├── splunk-searches.spl
        │   ├── wazuh-queries.txt
        │   ├── wazuh-rules.xml
        │   └── screenshots/
        │       ├── README.md
        │       └── 41 evidence screenshots
        └── A07-authentication-failures/
            ├── OWASP-A07-Authentication-Failures-Validation.md
            ├── splunk-searches.spl
            ├── wazuh-queries.txt
            ├── wazuh-rules.xml
            └── screenshots/
                └── README.md
```

## Evidence standards

Only sanitized evidence is published. Passwords, application passwords, API tokens, private keys, cookies and other secrets are excluded. Exercise evidence is organized inside each test folder, with reusable queries stored as text files.

## Skills demonstrated

- SOC monitoring and SIEM investigation
- Windows, Linux and Active Directory administration
- Firewall configuration and traffic analysis
- Log onboarding, field extraction and correlation
- Splunk SPL, dashboards, scheduled alerts and email actions
- Wazuh threat hunting and agent monitoring
- Detection engineering with Sigma and Atomic Red Team
- Windows service creation and persistence detection
- Windows scheduled-task persistence detection
- Windows Registry Run-key persistence detection
- Linux security-agent impairment, independent detection and recovery validation
- Windows Object Access auditing and controlled file-deletion detection
- Active Directory domain-account discovery detection with native and PowerShell telemetry
- SMB network-logon and Windows administrative-share detection across source and destination endpoints
- NIST CSF 2.0 and MITRE ATT&CK mapping
- Vulnerability assessment and packet analysis
- Web application testing with Burp Suite Proxy, HTTP history and Repeater
- Dockerized OWASP Juice Shop deployment behind an Nginx reverse proxy
- Nginx access-log onboarding, HTTP field extraction and exposed-resource detection
- Authentication-control testing, rate-limit and lockout validation
- Wazuh custom web rules and Splunk web-security alert engineering
- Incident containment, recovery and technical documentation

## Disclaimer

All security testing documented in this repository was conducted only against privately owned and authorized virtual machines in an isolated homelab. The material is intended for defensive security education and portfolio demonstration.



