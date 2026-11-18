# OWASP A03 Software Supply Chain Assessment and SIEM Validation

Activity date: **18 November 2026 (Asia/Manila)**.

**Status:** Assessment and SIEM alert workflow completed per operator confirmation.
The final post-test recovery result has not been supplied.

## Objective and scope

Assess the software components of the owner-operated OWASP Juice Shop training
container, review selected vulnerability advisories and validate finding collection,
custom detection and email notification in Wazuh and Splunk.

This activity covers component inventory and dependency assessment within A03.
It does not claim a compromised software supply chain, complete CI/CD assessment
or exploitation of the dependency findings.

## Baseline results

Trivy 0.76.2 generated a CycloneDX 1.7 SBOM containing 592 detected components.

| Metric | Baseline count |
|---|---:|
| Package vulnerability findings | 119 |
| Unique advisory IDs | 93 |
| Critical | 7 |
| High | 23 |
| Medium | 69 |
| Low | 15 |
| Unknown | 5 |

These are scanner-reported package findings. The advisory IDs are not all separate
exploitable application vulnerabilities. Selected findings were reviewed against
installed package metadata, function availability and application configuration.

## Work performed

1. Recorded the deployed image identity and application baseline.
2. Generated the SBOM and scanned it for known dependency vulnerabilities.
3. Reviewed selected package paths, cryptographic defaults and JWT configuration.
4. Distinguished affected-version matches from demonstrated application exposure.
5. Normalized findings as JSON Lines and collected them in both SIEMs.
6. Tested Wazuh severity rules and corrected Splunk event splitting and aggregation.
7. Generated a fresh WEB01 assessment and forwarded a labeled priority sample.
8. Operator confirmed live Wazuh, Splunk triggered-alert and received-email captures.

## Operational findings

The original finding batch was combined into one Splunk event. A controlled
two-record parsing test and a separate recovery source validated event splitting.
The recovered batch contained 119 matching events. Redundant JSON extraction was
removed from the summary query; corrected totals were confirmed by the operator.

The reviewed pre-integration health capture showed running services and forwarder,
a running container with the baseline image and loopback binding, and a normal
search returning HTTP 200 with three products. Final post-test recovery outcomes
remain to be supplied and must be recorded separately.

## Evidence and implementation status

See the [screenshot manifest](screenshots/README.md) and [report inventory](reports/README.md).
The public summary update does not publish the detailed infrastructure configuration,
implementation scripts or source screenshots. These are prepared separately for review.
The [target repository layout](../../../docs/repository-structure.md) identifies the
supporting files and distinguishes them from published content.

## Recommendations and limits

Review compatible supported upgrades for direct and transitive dependencies;
explicitly constrain JWT verification algorithms; use suitable key-derivation
settings where applicable; and rebuild and retest a separate remediation candidate.
No dependency patch, remediation retest or publisher-signature verification is claimed.

## References

- [OWASP A03:2025 Software Supply Chain Failures](https://top10.owasp.org/2025/A03_2025-Software_Supply_Chain_Failures/)
- [Trivy SBOM generation](https://trivy.dev/docs/latest/supply-chain/sbom/)
- [Trivy SBOM scanning](https://trivy.dev/docs/latest/target/sbom/)
