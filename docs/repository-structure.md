# Repository Structure

Updated 25 November 2026. Existing folders and screenshot uploads are retained.
Tables collapse image collections while listing the exact activity directories.

## Repository root

| Path | Contents |
|---|---|
| `README.md` | Project overview, capabilities, activity links and completion status |
| `assets/topology.png` | Updated rendered network and validation topology |
| `assets/topology.svg` | Editable source retaining the existing card design |
| `docs/` | Architecture, platform guides and this repository layout |
| `screenshots/` | Existing platform and infrastructure evidence |
| `tests/` | MITRE ATT&CK / NIST CSF and OWASP exercises |

## Test directories

| Directory | Contents |
|---|---|
| `tests/T1021-002-smb-windows-admin-shares/` | Existing validation report, queries and evidence |
| `tests/T1046-network-service-discovery/` | Existing validation report, queries and evidence |
| `tests/T1053-005-windows-scheduled-task/` | Existing validation report, queries and evidence |
| `tests/T1059-001-powershell-execution-validation/` | Existing validation report, queries and evidence |
| `tests/T1070-004-windows-file-deletion/` | Existing validation report, queries and evidence |
| `tests/T1087-002-domain-account-discovery/` | Existing validation report, queries and evidence |
| `tests/T1110-ssh-password-guessing/` | Existing validation report, queries and evidence |
| `tests/T1543-003-windows-service-creation/` | Existing validation report, queries and evidence |
| `tests/T1547-001-registry-run-keys/` | Existing validation report, queries and evidence |
| `tests/T1562-001-impair-defenses/` | Existing validation report, queries and evidence |
| `tests/owasp-top-10-2025/` | OWASP progress index, shared configuration and activity folders |

## OWASP activity directories

| Directory | Report | Uploaded PNG files |
|---|---|---:|
| `tests/owasp-top-10-2025/A01-A02-exposed-resources/` | [OWASP-A01-A02-Exposed-Resources-Validation.md](../tests/owasp-top-10-2025/A01-A02-exposed-resources/OWASP-A01-A02-Exposed-Resources-Validation.md) | 41 |
| `tests/owasp-top-10-2025/A03-software-supply-chain/` | [OWASP-A03-Software-Supply-Chain-Validation.md](../tests/owasp-top-10-2025/A03-software-supply-chain/OWASP-A03-Software-Supply-Chain-Validation.md) | 24 |
| `tests/owasp-top-10-2025/A04-cryptographic-failures/` | [OWASP-A04-Cryptographic-Failures-Validation.md](../tests/owasp-top-10-2025/A04-cryptographic-failures/OWASP-A04-Cryptographic-Failures-Validation.md) | 20 |
| `tests/owasp-top-10-2025/A05-injection/` | [OWASP-A05-SQL-Injection-Validation.md](../tests/owasp-top-10-2025/A05-injection/OWASP-A05-SQL-Injection-Validation.md) | 12 |
| `tests/owasp-top-10-2025/A06-insecure-design/` | [OWASP-A06-Insecure-Design-Validation.md](../tests/owasp-top-10-2025/A06-insecure-design/OWASP-A06-Insecure-Design-Validation.md) | 9 |
| `tests/owasp-top-10-2025/A07-authentication-failures/` | [OWASP-A07-Authentication-Failures-Validation.md](../tests/owasp-top-10-2025/A07-authentication-failures/OWASP-A07-Authentication-Failures-Validation.md) | 15 |
| `tests/owasp-top-10-2025/A08-software-data-integrity/` | [OWASP-A08-Software-Data-Integrity-Validation.md](../tests/owasp-top-10-2025/A08-software-data-integrity/OWASP-A08-Software-Data-Integrity-Validation.md) | 0 |

## A08 exact five-entry layout

Folder: `tests/owasp-top-10-2025/A08-software-data-integrity/`

| Entry | Purpose |
|---|---|
| [OWASP-A08-Software-Data-Integrity-Validation.md](../tests/owasp-top-10-2025/A08-software-data-integrity/OWASP-A08-Software-Data-Integrity-Validation.md) | Full validation report, embedded gate/configuration, email message, troubleshooting and recovery |
| [splunk-searches.spl](../tests/owasp-top-10-2025/A08-software-data-integrity/splunk-searches.spl) | Telemetry, scheduled detection, run correlation, recovery and diagnostics |
| [wazuh-queries.txt](../tests/owasp-top-10-2025/A08-software-data-integrity/wazuh-queries.txt) | Threat Hunting filters and correlation guidance |
| [wazuh-rules.xml](../tests/owasp-top-10-2025/A08-software-data-integrity/wazuh-rules.xml) | Corrected parent 119996 and blocked-artifact rule 119997 |
| [screenshots/](../tests/owasp-top-10-2025/A08-software-data-integrity/screenshots/README.md) | Eight-capture naming checklist; actual A08 images pending upload |

The report contains the gate source and host configuration so the activity does
not require separate scripts/config directories. Private signing keys are not
published. The operator reported completion; supplied text results and pending
image evidence are distinguished in the report and screenshot checklist.

## Evidence and progress

A06 now contains its eight core images plus the scheduler diagnostic; earlier
statements about missing uploads describe its previous publication checkpoint.
Detailed image review remains separately tracked. A08 has the five entries
above and no fabricated screenshot placeholders. Upload the actual eight PNGs
using the names in its screenshot checklist. A09 and A10 remain planned.

[OWASP program and status](../tests/owasp-top-10-2025/README.md)
