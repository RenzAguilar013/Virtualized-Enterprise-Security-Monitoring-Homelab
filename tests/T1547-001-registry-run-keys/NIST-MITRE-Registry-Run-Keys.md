# NIST–MITRE Registry Run Keys Detection Validation

## Overview

This exercise validates the detection and investigation of Windows Registry Run-key persistence in an isolated homelab. A harmless value named `T1547.001-LabRunKey` was created under the current user's Run key on the Windows 11 endpoint `Renz` (`10.10.10.20`). Native registry tools, Sysmon, Wazuh and Splunk were used to collect, correlate, alert on and investigate the activity.

The exercise maps to **MITRE ATT&CK T1547.001 — Boot or Logon Autostart Execution: Registry Run Keys / Startup Folder** and covers preparation, controlled execution, SIEM detection, dashboarding, email notification, cleanup and recovery.

## Authorization and scope

- Environment: privately owned VMware homelab
- Target: `Renz` — Windows 11 — `10.10.10.20`
- Registry path: `HKCU\Software\Microsoft\Windows\CurrentVersion\Run`
- Test value: `T1547.001-LabRunKey`
- Test data: `C:\Windows\System32\cmd.exe /c exit 0`
- SIEM platforms: Wazuh `10.10.10.30` and Splunk Enterprise `10.10.10.60`
- Safety: the value was harmless, was not executed through a logon cycle and was removed after validation

## Framework mapping

| Framework | Mapping | Application |
|---|---|---|
| MITRE ATT&CK | `T1547.001` | Registry Run-key persistence mechanism |
| NIST CSF 2.0 — Identify | `ID.AM`, `ID.RA` | Identified the endpoint, registry location and persistence risk |
| NIST CSF 2.0 — Protect | `PR.PS`, `PR.AA` | Maintained endpoint monitoring and controlled administrative access |
| NIST CSF 2.0 — Detect | `DE.CM`, `DE.AE` | Monitored registry changes through Sysmon, Wazuh and Splunk |
| NIST CSF 2.0 — Respond | `RS.AN`, `RS.MI` | Investigated and removed the unauthorized-style persistence value |
| NIST CSF 2.0 — Recover | `RC.RP`, `RC.CO` | Verified value removal, service health and SIEM connectivity |

## Telemetry and expected events

| Source | Event | Purpose |
|---|---:|---|
| Sysmon Operational | `12` | Registry object creation/deletion, including deletion evidence |
| Sysmon Operational | `13` | Registry value set |
| Sysmon Operational | `14` | Registry key/value rename, when applicable |
| Wazuh | Rule-based event | Endpoint detection and threat-hunting context |
| Splunk | Correlated event | Investigation, dashboard, alert and email action |

## Procedure

### 1. Verify monitoring

Sysmon, Wazuh Agent and Splunk Universal Forwarder were confirmed running on `Renz`.

### 2. Establish a registry baseline

The existing contents of the following key were displayed, saved as text and exported before testing:

```text
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
```

### 3. Validate Sysmon registry telemetry

A temporary benign key named `T1547BaselineTest` was created, modified and removed. Sysmon Events `12` and `13` confirmed that registry activity was being collected before the persistence test.

### 4. Create the controlled Run value

```powershell
$RunKey = "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run"

New-ItemProperty `
-Path $RunKey `
-Name "T1547.001-LabRunKey" `
-PropertyType String `
-Value "$env:SystemRoot\System32\cmd.exe /c exit 0" `
-Force
```

No sign-out or restart was performed, so the value did not need to execute.

### 5. Validate local telemetry

Sysmon Event `13` was reviewed for the user, process image, target registry object and value data.

### 6. Correlate in Wazuh and Splunk

Events were searched by host, test value and Event ID. The investigation correlated:

- Host and user
- Sysmon Event ID
- Process image
- Registry target object
- Registry value data
- Event timestamp

### 7. Create dashboard and alert

A Splunk dashboard panel named **T1547.001 — Registry Run Key Activity** was created. A high-severity scheduled alert searched a five-minute window every minute and added matching results to Triggered Alerts and an SMTP email action.

### 8. Remove the value and verify recovery

Only the controlled value was removed:

```powershell
Remove-ItemProperty `
-Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" `
-Name "T1547.001-LabRunKey" `
-Force
```

The entire Windows Run key was not deleted. Value absence, Sysmon deletion telemetry, monitoring-service status, Splunk forwarding and Wazuh connectivity were verified afterward.

## Detection logic

Reusable Splunk searches are stored in [splunk-searches.spl](splunk-searches.spl). Wazuh threat-hunting queries are stored in [wazuh-queries.txt](wazuh-queries.txt).

Detection focuses on:

- Sysmon Event `13` targeting Windows Run locations
- Newly created or modified Run values
- Process and user responsible for the change
- Value data that launches an executable, script or command interpreter
- Sysmon Event `12` during containment or cleanup

## Results

- A clean Run-key baseline was recorded.
- Sysmon registry collection was verified before testing.
- The controlled Run value generated Sysmon Event `13`.
- Wazuh and Splunk displayed the associated registry telemetry.
- A Splunk dashboard panel and high-severity alert were created.
- The alert appeared in Triggered Alerts and delivered an SMTP email.
- The test value was removed without deleting legitimate Run values.
- Endpoint monitoring and SIEM connectivity remained operational.

## Evidence

The [screenshots](screenshots/) directory contains 15 ordered screenshots covering endpoint preparation, registry baseline, controlled persistence, local and SIEM telemetry, dashboarding, alerting, email delivery, cleanup and recovery.

## Lessons learned

- A registry baseline helps distinguish legitimate startup applications from test or suspicious values.
- Sysmon Event `13` provides useful process, user, registry path and value-data context.
- Alerts should focus on sensitive Run locations while retaining enough fields for investigation.
- Throttling prevents repeated notifications but should be considered during controlled validation.
- Cleanup must remove only the test value rather than the complete Windows Run key.
- Recovery evidence confirms that testing left no persistence artifact.

## References

- [MITRE ATT&CK T1547.001 — Registry Run Keys / Startup Folder](https://attack.mitre.org/techniques/T1547/001/)
- [Microsoft Sysmon event documentation](https://learn.microsoft.com/sysinternals/downloads/sysmon)

