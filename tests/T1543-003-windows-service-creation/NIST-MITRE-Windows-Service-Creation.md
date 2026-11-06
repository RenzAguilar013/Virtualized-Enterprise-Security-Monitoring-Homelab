# NIST and MITRE Windows Service Creation Detection

## Test summary

| Field | Value |
|---|---|
| Test ID | `T1543.003-SVC-001` |
| Test date | November 5, 2026 |
| Status | Completed |
| MITRE ATT&CK | `T1543.003 - Create or Modify System Process: Windows Service` |
| Target | Renz - Windows 11 - `10.10.10.20` |
| Test service | `T1543LabService` |
| SIEM platforms | Wazuh and Splunk Enterprise |
| Authorization | Privately owned isolated homelab |

## Overview

This exercise validated an end-to-end detection and response workflow for Windows service creation. A harmless manual-start service was created on the Windows 11 endpoint, detected through native Windows and Sysmon telemetry, investigated in Wazuh and Splunk, converted into a dashboard and scheduled alert, delivered through SMTP email, and removed during recovery.

## Objectives

1. Establish endpoint and SIEM telemetry baselines.
2. Enable Windows service-installation auditing.
3. Create a controlled temporary service without starting it.
4. Validate System Event `7045` and Security Event `4697`.
5. Review related Sysmon process and registry telemetry.
6. Correlate the activity in Wazuh and Splunk.
7. Create a Splunk dashboard panel and high-severity alert.
8. Verify the triggered-alert and SMTP notification workflows.
9. Remove the service and confirm endpoint recovery.

## Scope and safety controls

- Testing was limited to the privately owned Windows 11 VM named `Renz`.
- The service used a unique lab-only name: `T1543LabService`.
- Startup type was set to manual.
- The service was not started.
- Existing Windows services were not modified.
- A pre-test service inventory was exported before execution.

## NIST CSF 2.0 mapping

| Function | Activity |
|---|---|
| Identify | Defined the target, service name, telemetry sources and expected behavior |
| Protect | Enabled auditing and confirmed Sysmon, Wazuh and Splunk Forwarder health |
| Detect | Collected and correlated Events `4697`, `7045` and relevant Sysmon events |
| Respond | Investigated the activity and triggered a high-severity Splunk alert and email |
| Recover | Deleted the service and verified removal of the service and registry key |

## Telemetry validated

| Source | Event ID | Purpose |
|---|---:|---|
| Windows System | `7045` | New service installed |
| Windows Security | `4697` | Service installation audit event |
| Sysmon | `1` | `sc.exe` process activity |
| Sysmon | `12-14` | Available service-registry activity |
| Wazuh | Multiple | Endpoint event decoding, rules and severity |
| Splunk | Multiple | Central search, correlation, dashboard and alerting |

## Controlled execution

The service was created from an elevated PowerShell session:

```powershell
New-Service `
  -Name "T1543LabService" `
  -BinaryPathName "C:\Windows\System32\cmd.exe /c exit 0" `
  -DisplayName "T1543.003 Lab Service" `
  -StartupType Manual
```

The service configuration was verified with:

```powershell
sc.exe qc T1543LabService
```

## Investigation workflow

1. Verified endpoint identity, monitoring services and event-log availability.
2. Enabled the `Security System Extension` audit subcategory.
3. Confirmed Security, System and Sysmon inputs in the Splunk Universal Forwarder.
4. Exported a pre-test Windows service inventory.
5. Created and inspected `T1543LabService`.
6. Confirmed local Events `7045` and `4697`.
7. Reviewed matching Sysmon activity.
8. Correlated service events in Splunk by host, event ID, service name and path.
9. Reviewed the Wazuh event, rule level and description.
10. Added the detection to a Splunk dashboard.
11. Created a scheduled alert with Triggered Alerts and SMTP email actions.
12. Removed the service and verified recovery.

## Alert configuration

| Setting | Value |
|---|---|
| Alert name | MITRE T1543.003 - Windows Service Creation |
| Type | Scheduled |
| Frequency | Every minute |
| Search window | Previous five minutes |
| Trigger condition | Result count greater than zero |
| Severity | High |
| Throttle | Ten minutes by host and service name |
| Actions | Add to Triggered Alerts and send email |

Reusable Splunk searches are stored in [`splunk-searches.spl`](splunk-searches.spl). Wazuh DQL queries are stored in [`wazuh-queries.txt`](wazuh-queries.txt).

## Results

| Validation | Result |
|---|---|
| Endpoint and monitoring-service baseline | Passed |
| Service-installation auditing | Passed |
| Controlled service creation | Passed |
| System Event `7045` | Passed |
| Security Event `4697` | Passed |
| Sysmon visibility | Passed |
| Splunk correlation search | Passed |
| Wazuh visibility | Passed |
| Splunk dashboard panel | Passed |
| High-severity triggered alert | Passed |
| SMTP email notification | Passed |
| Service cleanup and recovery | Passed |

## Cleanup and recovery

```powershell
sc.exe delete T1543LabService

Get-Service T1543LabService -ErrorAction SilentlyContinue
Test-Path "HKLM:\SYSTEM\CurrentControlSet\Services\T1543LabService"
```

The service no longer appeared in the Windows service inventory, and the corresponding registry path was removed. Historical Wazuh and Splunk events were retained as evidence.

## Evidence

The [`screenshots`](screenshots/) directory contains 20 sanitized screenshots covering the baseline, controlled execution, local Windows events, Sysmon, Wazuh, Splunk, dashboard, alert, email notification and recovery. The complete evidence index is available in [`screenshots/README.md`](screenshots/README.md).

## Lessons learned

- Native Windows System and Security events provide authoritative service-installation evidence.
- Sysmon adds process and registry context when the active configuration includes the relevant rules.
- Short scheduled-alert windows require a fresh test event for reliable validation.
- Adding both Triggered Alerts and email actions preserves local SIEM history and external notification evidence.
- A unique lab service name simplifies correlation across endpoint, Wazuh and Splunk data.
- Cleanup and recovery evidence are required to demonstrate a controlled, repeatable test.

## Conclusion

The exercise successfully demonstrated controlled validation of MITRE ATT&CK technique `T1543.003`. Windows, Sysmon, Wazuh and Splunk telemetry supported investigation, dashboarding, high-severity alerting, email notification and verified recovery without modifying an existing production-style service.

## Disclaimer

All activity was conducted against privately owned virtual machines in an isolated homelab for defensive security education and portfolio development.
