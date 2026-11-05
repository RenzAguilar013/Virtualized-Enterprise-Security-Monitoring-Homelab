# auditd Configuration

Linux01 uses auditd to record security-relevant changes and command activity. The audit log is forwarded to Splunk and monitored by the Wazuh agent.

## Coverage

- Identity files such as `/etc/passwd`, `/etc/group` and `/etc/shadow`
- sudo configuration and privileged command execution
- SSH server configuration
- Process execution
- File deletion and permission changes

## Validation

Events were generated in the Ubuntu test system, confirmed in `/var/log/audit/audit.log`, forwarded through Splunk Universal Forwarder and reviewed in Wazuh threat hunting.

## Operational checks

```bash
sudo systemctl status auditd
sudo auditctl -l
sudo ausearch -ts recent
```

The active rules should be reviewed and tuned to prevent unnecessary event volume while retaining visibility into privileged and security-sensitive activity.
