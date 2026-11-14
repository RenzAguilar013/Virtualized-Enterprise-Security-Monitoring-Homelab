# NIST–MITRE Windows File Deletion Validation

## Exercise summary

This authorized homelab exercise validated **MITRE ATT&CK T1070.004 — Indicator Removal: File Deletion** on the Windows 11 endpoint `Renz` (`10.10.10.20`). A harmless marker file was created in a dedicated lab directory, hashed with SHA-256, observed through Sysmon Event `11`, and then deleted in a controlled manner. Windows Security Events `4663` and `4660` provided deletion evidence, which was investigated in Wazuh and correlated in Splunk. A dashboard panel, scheduled high-severity alert and SMTP email notification completed the detection workflow.

## Scope and authorization

| Item | Value |
|---|---|
| Environment | Private VMware homelab |
| Endpoint | Renz / Windows 11 / `10.10.10.20` |
| Wazuh Manager | `10.10.10.30` |
| Splunk Enterprise | `10.10.10.60` |
| Technique | `T1070.004` — File Deletion |
| Test object | `C:\Lab-T1070\T1070_004_marker.txt` |
| Safety control | Dedicated folder, exact path, no wildcards and scoped audit SACL |

Only a harmless marker file on an authorized virtual machine was used. No operating-system, application or user data was deleted.

## Objectives

1. Establish Windows and SIEM telemetry baselines.
2. Preserve the existing Sysmon configuration when FileDelete rules are unavailable.
3. Configure deletion auditing only for a dedicated lab folder.
4. Record file creation, integrity and deletion evidence.
5. Correlate the activity in Wazuh and Splunk.
6. Trigger a dashboard, scheduled alert and email notification.
7. Remove temporary test artifacts and validate endpoint recovery.

## NIST Cybersecurity Framework 2.0 mapping

| Function | Exercise activity |
|---|---|
| Identify | Documented the endpoint, monitored object, accounts, processes and telemetry sources. |
| Protect | Used an isolated marker file, exact paths, a scoped SACL and a SHA-256 baseline. |
| Detect | Correlated Sysmon Event `11` with Security Events `4663`/`4660` in Wazuh and Splunk. |
| Respond | Triggered a high-severity Splunk alert and reviewed the account, process, object and access mask. |
| Recover | Removed the test SACL and folder and verified monitoring-service health. |

## Data sources

- Windows Security log and Advanced Audit Policy
- Sysmon Operational log
- PowerShell Operational log
- Wazuh Agent telemetry and Threat Hunting
- Splunk Universal Forwarder and Splunk Enterprise

## Procedure

### 1. Establish baselines

The operator verified the endpoint identity, address, monitoring services, Windows event channels and recent Sysmon telemetry in Splunk.

### 2. Select the deletion data source

The active Sysmon configuration did not contain `FileDelete` or `FileDeleteDetected` rules, and the original XML file was not retained. Rather than replace the working Sysmon configuration, Windows native Object Access auditing was used. File System auditing was enabled and a successful-deletion SACL was applied only to `C:\Lab-T1070`.

### 3. Create and hash the marker

The harmless file `T1070_004_marker.txt` was created with controlled text. Its metadata and SHA-256 hash were recorded before deletion. Sysmon Event `11` confirmed file creation.

### 4. Perform the controlled deletion

The marker was deleted using an exact literal path:

```powershell
Remove-Item -LiteralPath "C:\Lab-T1070\T1070_004_marker.txt" -Force
```

`Test-Path` returned `False`, confirming the object no longer existed.

### 5. Validate local telemetry

Security Event `4663` associated the marker path with the requesting account, PowerShell process and DELETE access. Related Event `4660` confirmed object-deletion activity but did not necessarily retain the filename.

### 6. Investigate in Wazuh and Splunk

Wazuh Threat Hunting was searched for Events `4663`/`4660`, the marker filename and access mask `0x10000`. Splunk extracted the user, process, object name, access list and access mask and mapped the result to `T1070.004`.

### 7. Dashboard and alert

A Splunk table panel displayed the deletion details. A scheduled alert ran every minute over a five-minute window, triggered on matching Event `4663` results, recorded the event in Triggered Alerts and sent a high-priority email notification.

### 8. Cleanup and recovery

The folder-specific SACL and test directory were removed. The global audit policy was left enabled to avoid overriding another security policy. Sysmon, Wazuh and the Splunk Universal Forwarder were verified as running.

## Results

| Control | Result |
|---|---|
| Controlled marker-file creation | Passed |
| SHA-256 baseline | Passed |
| Sysmon Event `11` creation telemetry | Passed |
| Security Event `4663` deletion access | Passed |
| Related Security Event `4660` review | Passed |
| Splunk detection and dashboard | Passed |
| Wazuh investigation | Completed; visibility depends on alert/archive indexing |
| Scheduled alert and email | Passed |
| Scoped cleanup and service recovery | Passed |

## Key findings

- Sysmon FileDelete telemetry should not be enabled by replacing an unknown working configuration; Windows Object Access auditing provided a safe alternative.
- Event `4663` is the strongest filename-level evidence because it includes the object, account, process and DELETE access.
- Event `4660` confirms deletion but normally requires correlation because it may omit the object name.
- The access mask `0x10000` represents DELETE permission.
- Wazuh results may require `wazuh-archives-*` when an event is collected but not promoted into `wazuh-alerts-*`.

## Evidence

The `screenshots/README.md` file defines the 18-image evidence sequence. The image files should be sanitized and uploaded into the same directory.

## References

- [MITRE ATT&CK T1070.004 — File Deletion](https://attack.mitre.org/techniques/T1070/004/)
- [Microsoft Event 4663](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4663)
- [Microsoft Event 4660](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4660)

