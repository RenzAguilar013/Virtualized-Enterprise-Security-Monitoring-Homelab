# NIST–MITRE Windows Scheduled Task Detection Validation

## Overview

This exercise validates the detection and investigation of Windows scheduled-task creation in an isolated homelab. A harmless one-time task named `T1053.005-LabTask` was created on the Windows 11 endpoint `Renz` (`10.10.10.20`) and monitored with native Windows auditing, Task Scheduler Operational logging, Sysmon, Wazuh and Splunk.

The activity is mapped to **MITRE ATT&CK T1053.005 — Scheduled Task/Job: Scheduled Task**. The exercise demonstrates telemetry preparation, controlled execution, SIEM correlation, dashboarding, alerting, email notification, cleanup and recovery verification.

## Authorization and scope

- Environment: privately owned VMware homelab
- Target: `Renz` — Windows 11 — `10.10.10.20`
- SIEM platforms: Wazuh `10.10.10.30` and Splunk Enterprise `10.10.10.60`
- Test object: `T1053.005-LabTask`
- Test action: `C:\Windows\System32\cmd.exe /c exit 0`
- Safety: the task was harmless, scheduled for a non-operational future time and removed after validation

## Framework mapping

| Framework | Mapping | Application |
|---|---|---|
| MITRE ATT&CK | `T1053.005` | Windows scheduled-task creation and execution mechanism |
| NIST CSF 2.0 — Identify | `ID.AM`, `ID.RA` | Identified the endpoint, telemetry sources and persistence risk |
| NIST CSF 2.0 — Protect | `PR.PS`, `PR.AA` | Enabled auditing, logging and controlled administrative access |
| NIST CSF 2.0 — Detect | `DE.CM`, `DE.AE` | Monitored Windows, Sysmon, Wazuh and Splunk events |
| NIST CSF 2.0 — Respond | `RS.AN`, `RS.MI` | Investigated the task and removed the test object |
| NIST CSF 2.0 — Recover | `RC.RP`, `RC.CO` | Verified task removal, service health and SIEM connectivity |

## Telemetry and expected events

| Source | Event | Purpose |
|---|---:|---|
| Windows Security | `4698` | A scheduled task was created |
| Windows Security | `4699` | A scheduled task was deleted |
| Task Scheduler Operational | `106` | A task was registered |
| Task Scheduler Operational | `141` | A task was deleted |
| Sysmon Operational | `1` | Process creation for `schtasks.exe` |
| Wazuh | Rule-based alert | Endpoint detection and threat-hunting evidence |
| Splunk | Correlated events | Investigation, dashboard, alert and email action |

## Procedure

### 1. Prepare telemetry

Other Object Access Events auditing and the Task Scheduler Operational channel were enabled. Security, Task Scheduler, Sysmon, Wazuh Agent and Splunk Universal Forwarder health were verified before testing.

### 2. Establish a baseline

Existing scheduled tasks were exported before the test to distinguish the controlled object from normal operating-system tasks.

### 3. Create the controlled task

The following authorized command created the harmless task:

```powershell
schtasks.exe /Create `
/TN "T1053.005-LabTask" `
/TR "C:\Windows\System32\cmd.exe /c exit 0" `
/SC ONCE `
/SD 12/31/2027 `
/ST 23:59 `
/F
```

### 4. Validate local telemetry

Local Windows logs were reviewed for Security Event `4698`, Task Scheduler Event `106` and Sysmon Event `1`. The task name, image, parent process and command line were recorded.

### 5. Correlate in Wazuh and Splunk

Events were searched by host, task name and event ID. Splunk normalized the activity into a timeline containing the task registration and `schtasks.exe` process creation. Wazuh supplied endpoint alert context.

### 6. Create dashboard and alert

A Splunk dashboard panel named **T1053.005 — Windows Scheduled Task Activity** was created. A high-severity scheduled alert searched a five-minute window every minute for Events `4698` and `106`, then added the result to Triggered Alerts and sent an SMTP email.

### 7. Remove the task and verify recovery

```powershell
schtasks.exe /Delete /TN "T1053.005-LabTask" /F
```

Deletion telemetry was reviewed through Events `4699` and `141`. The task was confirmed absent, and Sysmon, Wazuh Agent, Splunk Universal Forwarder and SIEM connectivity were verified.

## Detection logic

Reusable Splunk searches are stored in [splunk-searches.spl](splunk-searches.spl). Wazuh threat-hunting queries are stored in [wazuh-queries.txt](wazuh-queries.txt).

Detection focuses on:

- Task creation or registration events
- `schtasks.exe` process creation
- The task name and command line
- User, host and timestamp correlation
- Task deletion and recovery activity

## Results

- Windows scheduled-task auditing was successfully enabled.
- The controlled task produced Security, Task Scheduler and Sysmon telemetry.
- Wazuh and Splunk received and displayed the relevant activity.
- A Splunk dashboard panel and high-severity alert were created.
- The alert appeared under Triggered Alerts and delivered an SMTP email after throttle suppression was removed for the test.
- The task was deleted and endpoint monitoring and network connectivity remained operational.

## Evidence

The [screenshots](screenshots/) directory contains 20 ordered screenshots covering prerequisites, creation, local telemetry, Wazuh and Splunk correlation, dashboarding, alerting, email delivery, cleanup and recovery.

## Lessons learned

- Alert throttling can suppress a repeat validation event even when the alert search is functioning.
- Local Windows events should be confirmed before troubleshooting forwarding or SIEM searches.
- Task Scheduler Operational logging adds useful context beyond Security Event `4698`.
- Correlating native Windows events with Sysmon improves process and command-line visibility.
- Cleanup and recovery evidence are necessary to show that a controlled persistence test left no lab artifact.

## References

- [MITRE ATT&CK T1053.005 — Scheduled Task](https://attack.mitre.org/techniques/T1053/005/)
- [Microsoft Event 4698 — A scheduled task was created](https://learn.microsoft.com/windows/security/threat-protection/auditing/event-4698)

