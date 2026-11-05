# NIST and MITRE PowerShell Execution Validation

## Test summary

| Field | Value |
|---|---|
| Test ID | `T1059.001-PWSH-001` |
| Date | November 5, 2026 |
| Status | Completed |
| MITRE ATT&CK | `T1059.001 - Command and Scripting Interpreter: PowerShell` |
| Framework | Atomic Red Team |
| Target | Renz - Windows 11 - `10.10.10.20` |
| SIEM | Splunk Enterprise - `10.10.10.60` |
| Authorization | Privately owned isolated homelab |

## Objective

Validate an end-to-end detection-engineering workflow for controlled fileless PowerShell activity: endpoint telemetry, forwarding, centralized investigation, correlation, dashboarding, alerting, email notification and cleanup.

## NIST CSF 2.0 mapping

| Function | Validation activity |
|---|---|
| Identify | Defined the authorized scope, assets and expected behavior |
| Protect | Enabled PowerShell logging, Sysmon and SIEM forwarding |
| Detect | Collected and correlated process, script, registry and file telemetry |
| Respond | Investigated the activity and triggered a high-severity alert |
| Recover | Removed test artifacts and confirmed endpoint recovery |

## Test details

| Field | Value |
|---|---|
| Atomic test | PowerShell Fileless Script Execution |
| Test number | 1 |
| Test GUID | `fa050f5e-bc75-4230-af73-b6fd7852cd73` |
| Executor | PowerShell |
| Elevation required | No |

The test created temporary artifacts at:

```text
HKCU\Software\Classes\AtomicRedTeam
C:\Windows\Temp\art-marker.txt
```

## Telemetry validated

| Source | Event IDs | Purpose |
|---|---|---|
| Sysmon | `1` | Process and command-line creation |
| Sysmon | `11` | File creation |
| Sysmon | `12-14` | Registry activity |
| PowerShell Operational | `4103` | Module and command activity |
| PowerShell Operational | `4104` | Script block logging |

## Method

1. Verified Sysmon, PowerShell logging and Splunk Universal Forwarder health.
2. Confirmed baseline Sysmon and PowerShell events in Splunk.
3. Reviewed the Atomic test and created a VMware snapshot.
4. Executed the authorized T1059.001 test.
5. Investigated PowerShell, process, registry and file events.
6. Built a correlated Splunk timeline and dashboard panel.
7. Created a scheduled high-severity alert with email notification.
8. Ran Atomic cleanup and verified removal of both temporary artifacts.

## Detection logic

The detection correlates PowerShell Events 4103/4104 and Sysmon Event ID 1 with indicators such as:

```text
FromBase64String
Invoke-Expression
IEX
-EncodedCommand
-enc
AtomicRedTeam
art-marker.txt
```

Reusable searches are stored in [`splunk-searches.spl`](splunk-searches.spl).

## Alert configuration

| Setting | Value |
|---|---|
| Name | MITRE T1059.001 Suspicious PowerShell Execution |
| Schedule | Every minute |
| Search window | Previous five minutes |
| Trigger | Result count greater than zero |
| Severity | High |
| Throttle | Ten minutes by host |
| Action | SMTP email notification |

## Results

| Validation | Result |
|---|---|
| Local Sysmon and PowerShell events | Passed |
| Splunk forwarding and indexing | Passed |
| Process, registry and file telemetry | Passed |
| Correlated investigation timeline | Passed |
| Dashboard panel | Passed |
| Scheduled alert and email | Passed |
| Cleanup validation | Passed |

## Improvements made

- Corrected the deployment-server management port from `9997` to `8089` while retaining `9997` for event forwarding.
- Corrected Forwarder permissions for the Sysmon Operational channel.
- Added the missing Sysmon Forwarder input.
- Corrected a `Set-Cont` test-command typo to `Set-Content`.
- Added a targeted Sysmon FileCreate rule for `art-marker.txt`.

## Cleanup

```powershell
Invoke-AtomicTest T1059.001 `
  -TestNames "PowerShell Fileless Script Execution" `
  -Cleanup `
  -PathToAtomicsFolder "C:\AtomicRedTeam\atomics"

Test-Path "C:\Windows\Temp\art-marker.txt"
Test-Path "HKCU:\Software\Classes\AtomicRedTeam"
```

Both validation commands returned `False` after cleanup.

## Evidence

The complete evidence set is indexed in [`screenshots/README.md`](screenshots/README.md). It covers the endpoint baseline, Atomic test review and execution, local and centralized telemetry, dashboard, alert, email notification and cleanup. Evidence is sanitized before publication. Existing related Atomic Red Team and Sigma evidence is also available in the repository's [`screenshots/testing`](../../screenshots/testing/) directory.

## Conclusion

The exercise demonstrated repeatable MITRE ATT&CK-aligned validation across event generation, endpoint logging, forwarding, Splunk investigation, dashboarding, alerting, notification and recovery. All activity was limited to authorized lab systems.
