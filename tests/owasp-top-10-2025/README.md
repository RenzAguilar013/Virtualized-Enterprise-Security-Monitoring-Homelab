# OWASP Top 10:2025 Validation Program

This directory tracks authorized web application security exercises performed against the owner-controlled OWASP Juice Shop instance on WEB01 (`10.10.10.70`). Web-request exercises combine Burp Suite and Nginx evidence with Wazuh and Splunk telemetry. A03 adds Trivy inventory, dependency triage and scanner-finding collection in both SIEMs.

## Progress

| Category | Activity | Status |
|---|---|---|
| A01:2025 Broken Access Control | Unauthenticated access to exposed `/ftp/` resources | Completed |
| A02:2025 Security Misconfiguration | Directory listing and exposed document access | Completed |
| A03:2025 Software Supply Chain Failures | Container/SBOM assessment, advisory triage and dual-SIEM scanner-finding alerts | Assessment and alert workflow completed (operator-confirmed); final recovery result/evidence pending |
| A04:2025 Cryptographic Failures | Transport, token, and sensitive-data review | Planned |
| A05:2025 Injection | Product-search SQL injection and SIEM detection review | Completed; evidence reviewed, limitations documented |
| A06:2025 Insecure Design | Business-logic and abuse-case analysis | Planned |
| A07:2025 Authentication Failures | Login and rate-limit validation | Completed |
| A08:2025 Software or Data Integrity Failures | Image and deployment integrity validation | Planned |
| A09:2025 Security Logging and Alerting Failures | Detection coverage and logging-gap analysis | Planned |
| A10:2025 Mishandling of Exceptional Conditions | Controlled malformed-request and error-handling review | Planned |

## Completed activity

- [A01/A02 Exposed Resources](A01-A02-exposed-resources/OWASP-A01-A02-Exposed-Resources-Validation.md)
- [A07 Authentication Failures](A07-authentication-failures/OWASP-A07-Authentication-Failures-Validation.md)

- [A05 Product Search SQL Injection](A05-injection/OWASP-A05-SQL-Injection-Validation.md)
  includes the Boolean response comparison, Nginx/Splunk records, triggered alert,
  received email, Wazuh rule 31164, recovery evidence, and documented limitations.
  Screenshots 03, 04, and 06 require token redaction.

## A03 assessment and SIEM workflow

- [A03 Container and Dependency Assessment](A03-software-supply-chain/OWASP-A03-Software-Supply-Chain-Validation.md)
  records the 592-component SBOM, 119 package findings / 93 advisory IDs,
  applicability checks, custom Wazuh rules, Splunk ingestion troubleshooting,
  and operator-confirmed live alert/email captures. Final recovery outcomes and
  the remaining original evidence uploads are tracked in its manifest.

## Shared configuration

The `shared-config/` directory contains the WEB01 Nginx, Splunk Universal Forwarder, and Wazuh log-collection configuration used across the web-security exercises.

## Evidence standard

Every activity should document scope, authorization, baseline, request and response evidence, server telemetry, Wazuh detection, Splunk investigation, alerting, remediation, cleanup, and retest results. Do not publish credentials, tokens, cookies, private email addresses, or unnecessary confidential-looking content.
