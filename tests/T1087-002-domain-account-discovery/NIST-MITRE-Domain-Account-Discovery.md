# NIST–MITRE Domain Account Discovery Validation

## Exercise summary

This authorized homelab exercise validated **MITRE ATT&CK T1087.002 — Account Discovery: Domain Account** from the domain-joined Windows 11 endpoint `Renz` (`10.10.10.20`) against `DC01` (`10.10.10.10`). Two read-only discovery methods were tested: native `net user /domain` enumeration and PowerShell discovery using `Get-ADUser` or ADSI. Sysmon Event `1` and PowerShell Event `4104` captured the process and script activity, which was investigated in Wazuh and correlated in Splunk. A combined dashboard panel, scheduled high-severity alert and SMTP notification completed the detection workflow.

## Scope and authorization

| Item | Value |
|---|---|
| Environment | Private VMware homelab |
| Source endpoint | Renz / Windows 11 / `10.10.10.20` |
| Domain controller | DC01 / Windows Server 2022 / `10.10.10.10` |
| Wazuh Manager | `10.10.10.30` |
| Splunk Enterprise | `10.10.10.60` |
| Technique | `T1087.002` — Domain Account Discovery |
| Test type | Read-only domain queries |

The activity was performed only against an authorized private Active Directory domain. No accounts, groups, passwords or permissions were modified.

## Objectives

1. Verify domain membership and connectivity to DC01.
2. Establish Windows, Sysmon, PowerShell and SIEM baselines.
3. Execute native and PowerShell-based domain-account discovery.
4. Correlate the process, parent process, user, command line and script block.
5. Investigate the activity in Wazuh and Splunk.
6. Create a dashboard, scheduled alert and email notification.
7. Verify domain trust and endpoint/SIEM health after testing.

## NIST Cybersecurity Framework 2.0 mapping

| Function | Exercise activity |
|---|---|
| Identify | Documented the source endpoint, domain controller, accounts, tools and telemetry sources. |
| Protect | Used read-only queries, an isolated lab domain and sanitized evidence. |
| Detect | Correlated Sysmon Event `1` and PowerShell Event `4104` with discovery commands and scripts. |
| Respond | Triggered a high-severity Splunk alert and reviewed the process, user and command evidence. |
| Recover | Verified domain trust, monitoring services and Wazuh/Splunk network connectivity. |

## Data sources

- Sysmon Operational log — Event `1`
- PowerShell Operational log — Event `4104`
- Windows Security log
- Wazuh Agent and Threat Hunting
- Splunk Universal Forwarder and Splunk Enterprise
- Active Directory DNS, LDAP and SMB connectivity

## Procedure

### 1. Establish the baseline

The operator confirmed that `Renz` was joined to the lab domain, resolved `DC01`, and connected to LDAP port `389` and SMB port `445`. Sysmon, the Wazuh Agent and the Splunk Universal Forwarder were verified as running. Security, Sysmon and PowerShell event channels were enabled and visible in Splunk.

### 2. Verify discovery tooling

The availability of the Active Directory PowerShell module and `Get-ADUser` was checked. Native `net.exe` was verified as the universal discovery method. Where the AD module was unavailable, the built-in ADSI searcher provided a read-only alternative.

### 3. Native domain-account discovery

The following command enumerated domain account names without changing the directory:

```powershell
net user /domain
```

Sysmon Event `1` recorded `net.exe` or `net1.exe`, the `user /domain` command line, the invoking user and the PowerShell parent process.

### 4. PowerShell/ADSI discovery

The second test used one of the following read-only methods:

```powershell
Get-ADUser -Filter * | Select-Object -First 20 SamAccountName,Enabled,DistinguishedName
```

or:

```powershell
$Searcher = [adsisearcher]"(&(objectCategory=person)(objectClass=user))"
$Searcher.PropertiesToLoad.Add("samaccountname") | Out-Null
$Searcher.FindAll() | Select-Object -First 20
```

PowerShell Event `4104` captured the script block and directory-search terms.

### 5. Wazuh and Splunk investigation

Wazuh Threat Hunting was used to review Sysmon Event `1`, `net.exe`/`net1.exe`, `user /domain`, and PowerShell Event `4104`. Splunk extracted the event ID, image, command line, parent image, user and script block and mapped the evidence to `T1087.002`.

### 6. Dashboard and alerting

A combined Splunk table displayed both native and PowerShell/ADSI discovery. A scheduled alert ran every minute over a five-minute window, triggered on matching results, added an entry to Triggered Alerts and sent a high-priority SMTP notification.

### 7. Recovery validation

Because the exercise was read-only, no Active Directory cleanup was required. The domain secure channel returned `True`, Wazuh port `1514` and Splunk port `9997` were reachable, and Sysmon, Wazuh and the Splunk Forwarder remained active.

## Results

| Control | Result |
|---|---|
| Domain membership and DC01 connectivity | Passed |
| Native `net user /domain` discovery | Passed |
| Sysmon Event `1` process telemetry | Passed |
| PowerShell/ADSI discovery | Passed |
| PowerShell Event `4104` telemetry | Passed |
| Wazuh investigation | Passed |
| Splunk detection and dashboard | Passed |
| Scheduled alert and email | Passed |
| Domain trust and SIEM recovery validation | Passed |

## Key findings

- Domain account discovery can use common administrative utilities and therefore requires contextual detection rather than blocking every execution.
- Sysmon Event `1` provides strong process, parent-process, user and command-line evidence for `net user /domain`.
- PowerShell Event `4104` exposes higher-level directory queries such as `Get-ADUser` and ADSI filters.
- Combining multiple data sources reduces reliance on a single command or tool name.
- Production alerts should account for approved administration, help-desk workflows and known management systems.

## Evidence

The `screenshots/` directory contains 18 sanitized screenshots covering baselines, both discovery methods, local telemetry, Wazuh/Splunk investigations, dashboard, alert, email and recovery. See [`screenshots/README.md`](screenshots/README.md).

## References

- [MITRE ATT&CK T1087.002 — Domain Account](https://attack.mitre.org/techniques/T1087/002/)
- [Microsoft Sysmon documentation](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)
- [Microsoft PowerShell script block logging](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_logging_windows)

