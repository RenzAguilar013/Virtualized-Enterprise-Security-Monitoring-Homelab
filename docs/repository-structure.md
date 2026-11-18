# Repository Structure

The following is the current documentation layout for the homelab, including
A03 supply-chain assessment and the existing web exercises. Existing exercises and evidence stay in their original folders.
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
                wazuh-rules.xml
                wazuh-queries.txt
                config/
                    README.md
                    splunk-inputs.conf
                    splunk-props.conf
                    splunk-default.meta
                    wazuh-localfile.xml
                scripts/
                    README.md
                    trivy_common.py
                    export-trivy-findings.py
                    run-alert-test.py
                reports/
                    README.md
                    original exports to upload from WEB01
                screenshots/
                    README.md
                    published source captures
                    final captures to upload
                    troubleshooting/
            A05-injection/
                OWASP-A05-SQL-Injection-Validation.md
                burp-test-cases.md
                splunk-searches.spl
                wazuh-queries.txt
                screenshots/
                    README.md
                    12 evidence screenshots
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
records assessment and alert workflow completion per operator confirmation.
The public A03 update contains the summary and manifests. Thirteen source captures,
implementation configuration and scripts are prepared separately for review. Final summary, live dashboard,
triggered-alert/email images and original report exports still need uploading.
Final post-test recovery results have not been supplied.

Use the [A03 screenshot manifest](../tests/owasp-top-10-2025/A03-software-supply-chain/screenshots/README.md)
for exact filenames 01–21, including 16a/16b, optional 18b and final recovery
21a/21b. The reports inventory lists expected exports without fabricating them.

## A03 public update scope

The A03 file tree above is the target layout. The summary and directory README
files are published; detailed configuration, scripts, raw exports and screenshots
are not included in this public summary update. Existing exercises remain in place.
