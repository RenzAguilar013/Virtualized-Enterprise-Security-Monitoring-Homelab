# Repository Structure

The following is the current documentation layout for the homelab, including
A03 supply-chain assessment, A04 transport validation, A06 basket rules and the existing web exercises. Existing exercises and evidence stay in their original folders.
Screenshot collections are collapsed for readability.

```text
Virtualized-Enterprise-Security-Monitoring-Homelab/
    README.md
    assets/
        topology.png
        topology.svg
    docs/
        architecture.md
        repository-structure.md
        active-directory.md
        auditd-configuration.md
        firewall-policies.md
        fortigate-configuration.md
        splunk-configuration.md
        sysmon-telemetry.md
        vmware-network.md
        wazuh-configuration.md
        web-application-security-lab.md
        windows-client.md
    screenshots/
        existing platform and infrastructure evidence
    tests/
        T1021-002-smb-windows-admin-shares/
        T1046-network-service-discovery/
        T1053-005-windows-scheduled-task/
        T1059-001-powershell-execution-validation/
        T1070-004-windows-file-deletion/
        T1087-002-domain-account-discovery/
        T1110-ssh-password-guessing/
        T1543-003-windows-service-creation/
        T1547-001-registry-run-keys/
        T1562-001-impair-defenses/
        owasp-top-10-2025/
            README.md
            shared-config/
                nginx-juice-shop.conf
                splunk-forwarder-inputs.conf
                wazuh-agent-localfile.xml
            A01-A02-exposed-resources/
                existing report, queries, rules and screenshots
            A03-software-supply-chain/
                OWASP-A03-Software-Supply-Chain-Validation.md
                splunk-searches.spl
                wazuh-queries.txt
                wazuh-rules.xml
                screenshots/
                    README.md
            A04-cryptographic-failures/
                OWASP-A04-Cryptographic-Failures-Validation.md
                splunk-searches.spl
                wazuh-queries.txt
                wazuh-rules.xml
                screenshots/
                    README.md
                    02-a04-kali-http-https-transport-check.png
                    12-a04-wazuh-http-login-rule-logtest.png
                    20-a04-recovery-validation.png
            A05-injection/
                OWASP-A05-SQL-Injection-Validation.md
                burp-test-cases.md
                splunk-searches.spl
                wazuh-queries.txt
                screenshots/
                    README.md
                    12 evidence screenshots
            A06-insecure-design/
                OWASP-A06-Insecure-Design-Validation.md
                splunk-searches.spl
                wazuh-queries.txt
                wazuh-rules.xml
                screenshots/
                    README.md
                    03-a06-splunk-basket-telemetry.png
                    06a-a06-scheduler-email-action-diagnostic.png
            A07-authentication-failures/
                existing report, queries, rules and screenshots
```

## A05 uploaded evidence

The A05 screenshot directory contains the following reviewed PNGs:

- `01-a05-web01-search-baseline.png`
- `02-a05-burp-normal-search.png`
- `03-burp-single-quote-response.png`
- `04-burp-boolean-true.png`
- `05-burp-boolean-false.png`
- `06-nginx-search-test-timeline.png`
- `07-splunk-search-test-timeline.png`
- `08-splunk-triggered-alert.png`
- `09-email-notification.png`
- `10-wazuh-sql-injection-alert.png`
- `11-recovery-validation.png`
- `11-wazuh-logtest-before-custom-rule.png`

See the [screenshot manifest](../tests/owasp-top-10-2025/A05-injection/screenshots/README.md) for results and
redaction requirements for screenshots 03, 04, and 06.

## A05 completion

The controlled Boolean response comparison, Nginx/Splunk timeline, scheduled
alert, email delivery, Wazuh single-quote detection and application recovery
are documented. The report records the Boolean Wazuh coverage gap and limits
of the post-test service-health capture. No custom Wazuh rule or application
remediation is claimed.

## A03 evidence and completion

The [A03 report](../tests/owasp-top-10-2025/A03-software-supply-chain/OWASP-A03-Software-Supply-Chain-Validation.md)
documents the assessment, selected advisory review, Splunk and Wazuh detection
outcomes, scheduled alert settings and recovery status.

The folder uses the same structure as A01/A02 and A07: a validation report,
Splunk searches, Wazuh queries, Wazuh rules and a screenshots folder.
The query/rule files are published beside the report.

The [screenshot checklist](../tests/owasp-top-10-2025/A03-software-supply-chain/screenshots/README.md)
distinguishes reviewed captures from operator-confirmed outcomes and remaining
uploads. The A03 source captures, including final recovery images, have since been uploaded.
Original scan-export availability and final recovery outcome review remain tracked in A03.


## A04 structure and evidence

The [A04 activity](../tests/owasp-top-10-2025/A04-cryptographic-failures/OWASP-A04-Cryptographic-Failures-Validation.md)
uses exactly the same five-entry layout as the earlier OWASP folders: validation
report, Splunk searches, Wazuh queries, Wazuh rules and screenshots.

The original three key captures and later operator uploads are available. The
selected packet, live-alert, trigger and received-email evidence has now been
reviewed; the screenshot checklist identifies a concise portfolio selection. Configuration and the complete email message are
embedded in the report, so no extra config or script folders are required.

A04 transport checks, the JSON rule match and operational recovery were reviewed.
HTTPS enforcement remains an open finding. The remaining unfinished categories
are A08, A09 and A10; A03 retains its separately tracked recovery review.


## A06 structure and evidence

The [A06 activity](../tests/owasp-top-10-2025/A06-insecure-design/OWASP-A06-Insecure-Design-Validation.md)
uses the established five-entry layout: validation report, Splunk searches,
Wazuh queries, Wazuh rules and screenshots. Configuration snapshots and the
email subject/message are embedded in the report rather than extra setup folders.

Two supplied images are published: JSON telemetry and a supplemental scheduler
email-action diagnostic. The eight-capture checklist identifies the core
portfolio images still to upload. The operator reported activity completion;
direct image review and reported outcomes remain distinguished.

The negative basket state and restoration were operator-confirmed. Quantity
validation remains open; no checkout or application remediation is claimed.
