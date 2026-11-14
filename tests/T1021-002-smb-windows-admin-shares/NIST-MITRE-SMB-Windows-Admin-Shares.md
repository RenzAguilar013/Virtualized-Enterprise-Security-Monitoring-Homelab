# NIST–MITRE SMB/Windows Admin Shares Validation

## Exercise summary

This authorized homelab exercise validated **MITRE ATT&CK T1021.002 — Remote Services: SMB/Windows Admin Shares** between the domain-joined Windows 11 endpoint `Renz` (`10.10.10.20`) and domain controller `DC01` (`10.10.10.10`). A controlled, read-only connection to `\\DC01\IPC$` was created with the currently authenticated domain account. Windows Security Events `4624`, `5140` and `5145` and Sysmon Event `3` provided source, destination, account, share and network evidence. The activity was investigated in Wazuh and correlated in Splunk, followed by a dashboard panel, high-severity scheduled alert, SMTP notification and SMB-session cleanup.

## Scope and authorization

| Item | Value |
|---|---|
| Environment | Private VMware homelab |
| Source endpoint | Renz / Windows 11 / `10.10.10.20` |
| Destination endpoint | DC01 / Windows Server 2022 / `10.10.10.10` |
| Wazuh Manager | `10.10.10.30` |
| Splunk Enterprise | `10.10.10.60` |
| Technique | `T1021.002` — SMB/Windows Admin Shares |
| Test share | `\\DC01\IPC$` |
| Test type | Read-only authenticated SMB connection and share enumeration |

No password guessing, remote execution, service creation or file modification was performed. Credentials were never embedded in commands or evidence.

## Objectives

1. Verify source/destination identity, domain membership and TCP `445` connectivity.
2. Establish SMB-session, monitoring-service and SIEM baselines.
3. Enable the required File Share, Detailed File Share and Logon auditing on DC01.
4. Create a controlled IPC$ connection using the current domain account.
5. Correlate source IP, destination IP, account, logon type and share-access evidence.
6. Create a Splunk dashboard, scheduled alert and email notification.
7. Remove the SMB session and validate domain and SIEM health.

## NIST Cybersecurity Framework 2.0 mapping

| Function | Exercise activity |
|---|---|
| Identify | Documented both endpoints, the domain account, IPC$ share and required telemetry. |
| Protect | Used an authenticated read-only connection, secure credential handling and an isolated lab. |
| Detect | Correlated Events `4624`, `5140`, `5145` and Sysmon Event `3` in Wazuh and Splunk. |
| Respond | Triggered a high-severity alert and reviewed the account, source, destination and share. |
| Recover | Removed the IPC$ mapping and verified domain trust, monitoring services and SIEM connectivity. |

## Data sources

- DC01 Windows Security log
- Renz Sysmon Operational log
- Advanced Audit Policy: File Share, Detailed File Share and Logon
- Wazuh Agent and Threat Hunting
- Splunk Universal Forwarders and Splunk Enterprise
- Windows SMB client state and TCP `445` connectivity

## Procedure

### 1. Establish the baseline

The operator confirmed the identity and domain membership of Renz, resolved DC01, verified TCP port `445`, recorded existing SMB mappings and checked monitoring services on both endpoints. Recent events from both hosts were confirmed in Splunk.

### 2. Enable DC01 auditing

Successful and failed File Share and Detailed File Share auditing were enabled, and successful Logon auditing was confirmed. These settings support Events `5140`, `5145` and `4624`.

### 3. Create the controlled IPC$ connection

The following read-only connection used the currently authenticated domain account:

```powershell
net use \\DC01\IPC$
net view \\DC01
```

No password was typed directly into a command or stored in the repository.

### 4. Validate destination telemetry

On DC01:

- Event `4624` with Logon Type `3` recorded network authentication from `10.10.10.20`.
- Event `5140` recorded access to the `IPC$` network share.
- Event `5145` provided detailed share-access checks where available.

### 5. Validate source telemetry

On Renz, Sysmon Event `3` was reviewed for an outbound TCP connection to `10.10.10.10:445`. Windows may attribute SMB traffic to the `System` process because the operating system handles the SMB client connection.

### 6. Wazuh and Splunk investigation

Wazuh and Splunk correlated the source host/IP, destination host/IP, account, Logon Type `3`, share name and requested access. The activity was mapped to `T1021.002`.

### 7. Dashboard and alerting

A Splunk table panel displayed the correlated SMB timeline. A scheduled alert ran every minute over a five-minute window, triggered on matching results, wrote to Triggered Alerts and sent an SMTP notification.

### 8. Cleanup and recovery

The exact IPC$ mapping was removed with `net use \\DC01\IPC$ /delete /y`. Domain trust, Sysmon/Wazuh/Splunk Forwarder status and Wazuh/Splunk connectivity were validated. File Share auditing was retained because it provides ongoing defensive visibility.

## Results

| Control | Result |
|---|---|
| DC01 DNS and TCP `445` connectivity | Passed |
| Controlled IPC$ connection | Passed |
| Security Event `4624` Logon Type `3` | Passed |
| Security Event `5140` share access | Passed |
| Security Event `5145` detailed access review | Passed |
| Sysmon Event `3` source-network review | Passed |
| Wazuh and Splunk correlation | Passed |
| Dashboard, scheduled alert and email | Passed |
| SMB cleanup and recovery validation | Passed |

## Key findings

- Events `4624`, `5140` and `5145` provide complementary authentication and share-access context on the destination.
- Sysmon Event `3` adds source-side network evidence when network-event collection is enabled.
- SMB administrative shares are legitimate Windows mechanisms, so detections should include user, source host, destination, share and expected administrative workflows.
- Direct passwords must not be placed in commands, screenshots or repositories.
- Internal traffic on the same LAN may not traverse FortiGate; endpoint telemetry is therefore essential.

## Evidence

The `screenshots/` directory contains 18 sanitized screenshots covering baselines, auditing, the IPC$ connection, local events, Wazuh/Splunk investigation, dashboard, alert, email and recovery. See [`screenshots/README.md`](screenshots/README.md).

## References

- [MITRE ATT&CK T1021.002 — SMB/Windows Admin Shares](https://attack.mitre.org/techniques/T1021/002/)
- [Microsoft Event 4624](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4624)
- [Microsoft Event 5140](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-5140)
- [Microsoft Event 5145](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-5145)

