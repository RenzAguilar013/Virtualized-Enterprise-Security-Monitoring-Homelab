# OWASP 2025 A03 Software Supply Chain Validation

Activity date: **18 November 2026 (Asia/Manila)**.

## Objective

Inventory the deployed Juice Shop container, assess known dependency vulnerabilities,
review selected advisory prerequisites and validate Wazuh/Splunk collection,
scheduled detection and email notification.

## Scope and authorization

Testing was limited to the owner-controlled Juice Shop training container on WEB01.
This activity assesses container components and dependency findings within
**A03:2025 Software Supply Chain Failures**. It does not demonstrate a compromised
package, registry or build system, or assess the entire CI/CD supply chain.

## Environment

| Component | Address | Purpose |
|---|---|---|
| WEB01 | `10.10.10.70` | Nginx, Docker/Juice Shop, Trivy, Wazuh Agent and Splunk Universal Forwarder |
| Juice Shop | `127.0.0.1:3000` on WEB01 | Owner-controlled application behind Nginx |
| Wazuh Manager | `10.10.10.30` | JSON decoding, custom severity rules and live alerts |
| Splunk Enterprise | `10.10.10.60` | Finding investigation, scheduled alert and email notification |
| Trivy | `0.76.2` | CycloneDX SBOM generation and vulnerability assessment |

## Security mapping

- **OWASP Top 10 A03:2025 — Software Supply Chain Failures**
- **NIST CSF 2.0:** Identify through component inventory and risk assessment; Detect through centralized finding collection and severity-based alerts.
- No ATT&CK technique execution or dependency exploit is claimed for this assessment.

## Test method

1. Recorded the running image identity, container binding and application baseline.
2. Generated a CycloneDX 1.7 SBOM containing 592 detected components.
3. Scanned for known vulnerabilities and retained package-level findings.
4. Checked selected package versions, function availability and backend usage.
5. Exported findings as JSON Lines with asset, scan ID, package, advisory, severity and version fields.
6. Collected the findings in Wazuh and Splunk.
7. Validated Wazuh Critical/High rules with logtest and reviewed live collection.
8. Corrected Splunk event splitting and redundant JSON extraction.
9. Generated a fresh WEB01 scan and forwarded one real priority finding as a labeled test sample with a new scan ID.
10. Operator confirmed live Wazuh, Splunk triggered-alert and received-email screenshot completion.

## Findings

### Baseline dependency assessment

| Metric | Count |
|---|---:|
| Detected SBOM components | 592 |
| Package vulnerability findings | 119 |
| Unique advisory IDs | 93 |
| Critical | 7 |
| High | 23 |
| Medium | 69 |
| Low | 15 |
| Unknown | 5 |

These are scanner-reported package findings. One advisory can match multiple
package instances; the counts do not establish 119 exploitable application flaws.

### Selected advisory review

| Package / advisory | Observed evidence | Assessment |
|---|---|---|
| crypto-js 3.3.0 / CVE-2023-46233 | PBKDF2 exists with one default iteration and SHA-1 | Weak defaults confirmed; no PBKDF2 reference found in the scoped backend search. Application exposure remains unconfirmed. |
| Nested lodash 2.4.2 / CVE-2019-10744 | Installed beneath sanitize-html; `defaultsDeep` returned undefined | Scanner match requires applicability review; exploitation was not demonstrated. |
| express-jwt 0.1.3 / CVE-2020-15084 | Authorization passes a public key without an explicit algorithms option | Verification-hardening concern observed. The advisory's JWKS-provider prerequisite was not established. |

The scoped backend search examined 212 files and found four matching lines.
Absence of a source-text match does not establish complete unreachability.
The separate RS256 signing option does not establish a verification restriction.

## Detection outcome

### Splunk

The final searches use index `web`, sourcetype `trivy:findings` and the exact
recovery source `/var/log/trivy/splunk-recovery/findings.jsonl`. This excludes
the preserved original merged batch and the separate canary file.

The original 119 JSON records arrived as one event. A two-record canary later
produced two events with two distinct probe IDs. The recovered batch matched
119 events. Removing redundant `spath` from searches using `KV_MODE=json`
resolved the fourfold summary overcount, as confirmed by the operator.
The precise cause of the original merge was not conclusively established.

| Alert setting | Configured workflow |
|---|---|
| Schedule | Every minute: `* * * * *` |
| Lookback | Last five minutes |
| Condition | Critical or High findings; number of results greater than zero |
| Trigger mode | Per result |
| Suppression | 24 hours by `asset` and `scan_id` |
| Actions | Triggered Alerts and the existing email action |

Each controlled WEB01 test uses a new `alert-test-...` ID and forwards one
real Critical/High finding with `validation_sample=true`. It validates the
alert pipeline; it is not a replay of all 119 findings or a dependency exploit.
The operator confirmed triggered-alert and received-email screenshot completion.

### Wazuh

Wazuh collects `/var/log/trivy/findings.json` with the JSON decoder.

| Rule | Level | Purpose |
|---|---:|---|
| `119980` | 0 | Parent match for Trivy vulnerability-finding records |
| `119981` | 12 | Critical scanner finding |
| `119982` | 10 | High scanner finding |

Reviewed logtest captures show the JSON decoder and expected Critical/High rule
matches. Live Wazuh alerts were confirmed by the operator. The baseline would
produce 7 Critical and 23 High alerts when collected once; no exact live count
was supplied. Filter by scan ID to separate later test samples.
These custom scanner-finding rules are separate from native Wazuh Vulnerability Detection.

## Recovery validation

The reviewed pre-integration health capture showed successful `nginx -t`,
active Nginx/Docker/Wazuh-agent services, running Splunk forwarder processes,
the running baseline container with loopback binding, and normal application
search returning HTTP 200 with three products.

**The final post-test recovery results have not been supplied.** Screenshots
21a/21b remain pending; the earlier health capture is not proof of the final
post-alert state. No dependency upgrade, application patch or remediation
retest is claimed.

## Recommended remediation

- Review supported upgrades for direct and transitive dependencies and rebuild a separate remediation candidate.
- Explicitly constrain JWT verification algorithms and review key-derivation settings where used.
- Pin reviewed image versions or digests and assess trusted-source and signature controls.
- Repeat vulnerability assessment, application checks and monitoring validation after changes.

## Evidence availability

The screenshot checklist distinguishes reviewed captures, operator-confirmed
outcomes and images still to upload. Original SBOM/scan exports and the final
alert/email captures have not been uploaded in this repository update.

## Related files

- [Splunk searches](splunk-searches.spl)
- [Wazuh queries](wazuh-queries.txt)
- [Wazuh rules](wazuh-rules.xml)
- [Evidence checklist](screenshots/README.md)
- [Shared WEB01 configuration](../shared-config/)

## References

- [OWASP A03:2025 Software Supply Chain Failures](https://top10.owasp.org/2025/A03_2025-Software_Supply_Chain_Failures/)
- [Trivy SBOM generation](https://trivy.dev/docs/latest/supply-chain/sbom/)
- [Trivy SBOM scanning](https://trivy.dev/docs/latest/target/sbom/)
