# Repository Structure

The following is the final documentation layout for the existing homelab plus
Exercise 3. Existing exercises and evidence stay in their original folders.
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
            A05-injection/
                OWASP-A05-SQL-Injection-Validation.md
                burp-test-cases.md
                splunk-searches.spl
                wazuh-queries.txt
                screenshots/
                    README.md
            A07-authentication-failures/
                existing report, queries, rules and screenshots
```

## A05 evidence filenames

Add the following sanitized original images inside
`tests/owasp-top-10-2025/A05-injection/screenshots/` as evidence is available:

- `01-baseline.png`
- `02-burp-normal-search.png`
- `03-burp-single-quote-response.png`
- `04-burp-boolean-true.png`
- `05-burp-boolean-false.png`
- `06-nginx-search-test-timeline.png`
- `07-splunk-search-test-timeline.png`
- `08-splunk-triggered-alert.png`
- `09-email-notification.png`
- `10-wazuh-sql-injection-alert.png`
- `11-recovery-validation.png`

These PNGs are intended additions, not files supplied in this update. The
[screenshot manifest](../tests/owasp-top-10-2025/A05-injection/screenshots/README.md)
explains each capture. No `wazuh-rules.xml` is added because a custom A05 rule
was not implemented. The observed existing Wazuh alert belongs in the report.

## Completion criteria

Record exact Boolean response outcomes, Wazuh rule ID and level, Splunk alert
and email outcomes, and recovery results before marking the full A05 workflow
complete. A screenshot filename or a supplied procedure does not establish that
the corresponding validation passed.
