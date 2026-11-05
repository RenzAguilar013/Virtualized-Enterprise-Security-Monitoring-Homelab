# Sysmon Telemetry

Sysmon provides detailed Windows endpoint telemetry for the DC01 and Renz systems. Its Operational log is collected by Splunk Universal Forwarder and Wazuh for centralized investigation.

## Monitored events

| Event ID | Activity | Homelab use |
|---:|---|---|
| 1 | Process creation | PowerShell, command line and parent process analysis |
| 3 | Network connection | Process-to-destination correlation when enabled |
| 11 | File creation | Atomic marker and suspicious file-write validation |
| 12-14 | Registry activity | Atomic test and persistence-related changes |
| 22 | DNS query | Process and destination-domain investigation |

## Splunk Universal Forwarder input

```ini
[WinEventLog://Microsoft-Windows-Sysmon/Operational]
disabled = 0
renderXml = true
current_only = 0
```

## T1059.001 telemetry

The PowerShell validation exercise correlated Sysmon Event ID 1 with PowerShell Events 4103 and 4104. Targeted rules also captured Event ID 11 for `art-marker.txt` and Events 12-14 for the temporary `AtomicRedTeam` registry path.

See [PowerShell Execution Validation](../tests/T1059-001-powershell-execution-validation/NIST-MITRE-PowerShell-Execution-Validation.md).
