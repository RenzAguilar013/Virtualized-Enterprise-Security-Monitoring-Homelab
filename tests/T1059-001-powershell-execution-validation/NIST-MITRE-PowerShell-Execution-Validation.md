# NIST and MITRE PowerShell Execution Validation

## Test summary

| Field | Value |
|---|---|
| Test ID | `T1059.001-PWSH-001` |
| Test date | November 5, 2026 |
| Status | Completed |
| MITRE ATT&CK | `T1059.001 — Command and Scripting Interpreter: PowerShell` |
| Test framework | Atomic Red Team |
| Target endpoint | `Renz` — Windows 11 — `10.10.10.20` |
| SIEM | Splunk Enterprise — `10.10.10.60` |
| Telemetry | Sysmon and PowerShell Operational logging |
| Authorization | Privately owned, isolated homelab |

## Overview

This exercise validated the collection, detection, investigation and alerting of suspicious PowerShell activity in an isolated enterprise security homelab.

A controlled Atomic Red Team test was executed on the Windows 11 endpoint. Sysmon and PowerShell Operational logging recorded the activity, the Splunk Universal Forwarder transmitted the events, and Splunk Enterprise was used for investigation, correlation, dashboard visualization, alerting and email notification.

## Objectives

The objectives were to:

1. Validate Sysmon and PowerShell telemetry collection.
2. Establish normal telemetry baselines in Splunk.
3. Execute an authorized Atomic Red Team PowerShell test.
4. Detect process, script, registry and file activity.
5. Correlate related events into an investigation timeline.
6. Create a Splunk dashboard panel.
7. Configure a scheduled Splunk alert.
8. Verify SMTP email notification.
9. Remove all temporary test artifacts.

## Scope and authorization

Testing was restricted to the following privately owned virtual machines:

| System | Address | Purpose |
|---|---:|---|
| Renz | `10.10.10.20` | Windows 11 test endpoint |
| Splunk Enterprise | `10.10.10.60` | Centralized event collection and detection |

No production systems, external organizations or unauthorized devices were targeted.

## Tools used

- Atomic Red Team
- Invoke-AtomicRedTeam PowerShell module
- Windows PowerShell
- PowerShell Operational logging
- Sysmon
- Splunk Universal Forwarder
- Splunk Enterprise
- VMware Workstation

## Framework mapping

### MITRE ATT&CK

| Technique | Name | Validation |
|---|---|---|
| `T1059.001` | Command and Scripting Interpreter: PowerShell | Controlled fileless PowerShell execution |
| `T1112` | Modify Registry | Temporary Atomic Red Team registry value |
| `T1106` | Native API | PowerShell and native Windows process execution |

The primary technique validated by this exercise was `T1059.001`.

### NIST Cybersecurity Framework 2.0

| Function | Category | Activity |
|---|---|---|
| Identify | Risk Assessment | Defined the authorized test scope and expected behavior |
| Protect | Platform Security | Enabled PowerShell logging and Sysmon monitoring |
| Detect | Continuous Monitoring | Collected Sysmon and PowerShell telemetry |
| Detect | Adverse Event Analysis | Correlated process, script, registry and file events |
| Respond | Incident Analysis | Investigated the activity through Splunk |
| Respond | Incident Mitigation | Validated alerting and reviewed containment options |
| Recover | Recovery Planning | Removed artifacts and confirmed endpoint recovery |

## Test prerequisites

Before execution, the following prerequisites were verified:

- Sysmon service was running.
- PowerShell Operational logging was enabled.
- PowerShell Script Block Logging was enabled.
- Splunk Universal Forwarder was running.
- Sysmon and PowerShell inputs were enabled in `inputs.conf`.
- The forwarder was connected to `10.10.10.60:9997`.
- Baseline Sysmon and PowerShell events were visible in Splunk.
- The Invoke-AtomicRedTeam module was available.
- A VMware snapshot was created before testing.

## Atomic Red Team test

| Field | Value |
|---|---|
| Technique | `T1059.001` |
| Test number | `1` |
| Test name | PowerShell Fileless Script Execution |
| Test GUID | `fa050f5e-bc75-4230-af73-b6fd7852cd73` |
| Executor | PowerShell |
| Elevation required | No |

The controlled test created temporary detection artifacts:

```text
HKCU\Software\Classes\AtomicRedTeam
C:\Windows\Temp\art-marker.txt
```

The test stored encoded content in a temporary registry value and used PowerShell decoding and execution behavior to create the marker file.

## Execution command

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass -Force

Import-Module Invoke-AtomicRedTeam -Force

Invoke-AtomicTest T1059.001 `
  -TestNames "PowerShell Fileless Script Execution" `
  -PathToAtomicsFolder "C:\AtomicRedTeam\atomics"
```

## Telemetry validated

| Source | Event ID | Description |
|---|---:|---|
| Sysmon | `1` | Process creation |
| Sysmon | `11` | File creation |
| Sysmon | `12` | Registry object creation or deletion |
| Sysmon | `13` | Registry value modification |
| Sysmon | `14` | Registry object rename |
| PowerShell Operational | `4103` | Module and command activity |
| PowerShell Operational | `4104` | Script Block Logging |

## Validation workflow

### Phase 1: Prerequisites and baseline

1. Verified PowerShell logging.
2. Verified the Sysmon service.
3. Verified the Splunk Universal Forwarder.
4. Enabled Sysmon and PowerShell Forwarder inputs.
5. Confirmed that the Atomic Red Team module was available.
6. Generated normal PowerShell activity.
7. Confirmed Sysmon Event ID `1` locally.
8. Confirmed Sysmon process telemetry in Splunk.
9. Confirmed PowerShell Operational telemetry in Splunk.

### Phase 2: Controlled execution

1. Reviewed the Atomic test details.
2. Confirmed that the test did not target credentials or external systems.
3. Executed the controlled T1059.001 test.
4. Verified creation of the registry and marker-file artifacts.
5. Confirmed that no unexpected system changes occurred.

### Phase 3: Investigation and correlation

Splunk was used to identify:

- PowerShell Events `4103` and `4104`.
- Sysmon Event ID `1` process creation.
- PowerShell and `reg.exe` command lines.
- Registry Events `12–14`.
- File creation Event ID `11`.
- Activity related to `AtomicRedTeam`.
- `FromBase64String` and `Invoke-Expression` behavior.
- The chronological relationship between the events.

The complete searches are available in [`splunk-searches.spl`](splunk-searches.spl).

### Phase 4: Alerting and notification

A scheduled Splunk alert was configured with the following settings:

| Setting | Value |
|---|---|
| Alert name | MITRE T1059.001 Suspicious PowerShell Execution |
| Type | Scheduled |
| Frequency | Every minute |
| Search window | Previous five minutes |
| Trigger condition | Number of results greater than zero |
| Severity | High |
| Throttling | Ten minutes by host |
| Action | SMTP email notification |

The alert successfully triggered after the Atomic test was repeated. The email notification included the hostname, event IDs, telemetry sources, detection reason, process information and investigation link.

## Detection indicators

The detection searched for the following behaviors:

```text
FromBase64String
Invoke-Expression
IEX
-EncodedCommand
-enc
AtomicRedTeam
art-marker.txt
```

These indicators were correlated with PowerShell Events `4103/4104` and Sysmon Event ID `1`.

## Results

| Validation | Result |
|---|---|
| PowerShell events created locally | Passed |
| Sysmon events created locally | Passed |
| Splunk Universal Forwarder connected | Passed |
| PowerShell events received by Splunk | Passed |
| Sysmon process events received by Splunk | Passed |
| Registry events received by Splunk | Passed |
| FileCreate Event ID 11 received by Splunk | Passed |
| Investigation timeline created | Passed |
| Dashboard panel created | Passed |
| Scheduled alert triggered | Passed |
| Email notification received | Passed |
| Atomic artifacts removed | Passed |

## Telemetry gap and improvement

Sysmon FileCreate Event ID `11` was initially not recorded because the active configuration did not include a matching FileCreate rule.

A targeted rule was added:

```xml
<FileCreate onmatch="include">
  <TargetFilename condition="end with">\art-marker.txt</TargetFilename>
</FileCreate>
```

This provided the required visibility without enabling excessive file-creation logging.

## Troubleshooting performed

The following issues were identified and corrected during testing:

- The Splunk deployment-server connection was incorrectly using port `9997` instead of management port `8089`.
- The Splunk Forwarder initially lacked permission to read the Sysmon Operational channel.
- The Forwarder Sysmon input was missing from the effective configuration.
- A test command contained `Set-Cont` instead of `Set-Content`.
- Sysmon Event ID `11` required a targeted FileCreate rule.
- Broad searches were used before applying narrow field extractions.

These issues demonstrated the importance of validating each telemetry stage independently:

```text
Event generation → Local Windows log → Splunk Forwarder → Splunk index → Detection
```

## Cleanup

The Atomic Red Team cleanup command was executed:

```powershell
Invoke-AtomicTest T1059.001 `
  -TestNames "PowerShell Fileless Script Execution" `
  -Cleanup `
  -PathToAtomicsFolder "C:\AtomicRedTeam\atomics"
```

Cleanup was verified with:

```powershell
Test-Path "C:\Windows\Temp\art-marker.txt"
Test-Path "HKCU:\Software\Classes\AtomicRedTeam"
```

Expected and observed result:

```text
False
False
```

## Evidence

| Screenshot | Evidence |
|---|---|
| [01](screenshots/01-powershell-logging-enabled.png) | PowerShell logging configuration |
| [02](screenshots/02-sysmon-service-running.png) | Sysmon service status |
| [03](screenshots/03-splunk-forwarder-running.png) | Forwarder service and connection |
| [04](screenshots/04-splunk-inputs-enabled.png) | Effective PowerShell and Sysmon inputs |
| [05](screenshots/05-atomic-module-and-tests-verified.png) | Atomic module verification |
| [06](screenshots/06-local-powershell-baseline.png) | Local PowerShell baseline |
| [07](screenshots/07-local-sysmon-baseline.png) | Local Sysmon baseline |
| [08](screenshots/08-splunk-sysmon-baseline.png) | Splunk Sysmon baseline |
| [09](screenshots/09-splunk-powershell-event.png) | Splunk PowerShell test event |
| [10](screenshots/10-atomic-t1059-001-test-details.png) | Atomic test review |
| [11](screenshots/11-atomic-t1059-001-execution.png) | Controlled execution |
| [12](screenshots/12-splunk-powershell-atomic-event.png) | PowerShell Atomic event |
| [13](screenshots/13-splunk-sysmon-process-creation.png) | Sysmon process creation |
| [14](screenshots/14-splunk-registry-file-artifacts.png) | Registry and file artifacts |
| [15](screenshots/15-splunk-t1059-001-timeline.png) | Correlated timeline |
| [16](screenshots/16-splunk-t1059-001-dashboard.png) | Dashboard panel |
| [17](screenshots/17-splunk-t1059-001-alert-config.png) | Alert configuration |
| [18](screenshots/18-splunk-t1059-001-alert-triggered.png) | Triggered alert |
| [19](screenshots/19-t1059-001-email-alert.png) | Email notification |
| [20](screenshots/20-atomic-cleanup-validation.png) | Cleanup validation |

## Lessons learned

- Local event generation should be confirmed before troubleshooting the SIEM.
- Effective Forwarder configuration should be checked using `btool`.
- Sysmon rules should be targeted to balance visibility and event volume.
- PowerShell and Sysmon telemetry provide stronger detection when correlated.
- Scheduled alerts require an appropriate search window and throttling.
- Cleanup and recovery evidence are necessary parts of repeatable security testing.
- Detection tests should be reviewed before execution.

## Conclusion

The exercise successfully demonstrated an end-to-end detection-engineering workflow for MITRE ATT&CK technique `T1059.001`.

The activity was generated through Atomic Red Team, recorded by Sysmon and PowerShell logging, forwarded to Splunk, investigated through SPL, visualized on a dashboard, detected by a scheduled alert and delivered through email notification.

The temporary artifacts were removed and the Windows endpoint was returned to its original operational state.

## Disclaimer

All activity was conducted against privately owned virtual machines in an isolated homelab. The exercise was performed exclusively for defensive security education and portfolio development.
