# NIST–MITRE Linux Impair Defenses Validation

## Exercise summary

This authorized homelab exercise validated detection and recovery for **MITRE ATT&CK T1562.001 — Impair Defenses: Disable or Modify Tools**. On Linux01, the Wazuh Agent was stopped briefly while `auditd` and the Splunk Universal Forwarder remained active. Splunk correlated the stop command, service lifecycle activity and a controlled test marker. A systemd recovery timer restored the Wazuh Agent automatically, after which agent connectivity and service health were verified.

## Scope and authorization

| Item | Value |
|---|---|
| Environment | Private VMware homelab |
| Endpoint | Linux01 / Ubuntu / `10.10.10.50` |
| Wazuh Manager | `10.10.10.30` |
| Splunk Enterprise | `10.10.10.60` |
| Technique | `T1562.001` — Impair Defenses |
| Test type | Controlled, time-bounded service interruption |
| Safety control | Automatic systemd recovery timer configured before interruption |

Only systems owned by the lab operator were included. The exercise did not modify or target third-party systems.

## Objectives

1. Establish endpoint and telemetry baselines.
2. Test whether an interruption of the Wazuh Agent can be observed independently.
3. Correlate auditd, auth and systemd evidence in Splunk.
4. Create a dashboard panel, scheduled alert and email notification.
5. Restore the agent and validate endpoint and SIEM health.
6. Document any detection gap discovered during the test.

## NIST Cybersecurity Framework 2.0 mapping

| Function | Exercise activity |
|---|---|
| Identify | Documented Linux01, its security services, data sources and SIEM connections. |
| Protect | Used a bounded test, retained auditd and Splunk telemetry, and configured automatic recovery. |
| Detect | Correlated the service-stop command, systemd lifecycle events and the controlled marker in Splunk. |
| Respond | Triggered a high-severity alert and email notification and reviewed the interruption timeline. |
| Recover | Restarted the Wazuh Agent and verified connectivity, service enablement and system health. |

## Data sources

- auditd records from `/var/log/audit/audit.log`
- sudo and authentication events from `/var/log/auth.log`
- systemd and journal service lifecycle events
- Splunk Universal Forwarder telemetry from Linux01
- Wazuh Agent connection state and post-recovery events

## Procedure

### 1. Establish the baseline

The Linux01 hostname, IP address, operating system and active monitoring services were recorded. The operator confirmed that `auditd`, `wazuh-agent` and `SplunkForwarder` were running, and verified that Linux logs were searchable in Splunk.

### 2. Configure automatic recovery

Before stopping the agent, a transient systemd timer was used to schedule recovery. This limited the interruption and reduced the risk of leaving Linux01 unmonitored.

```bash
sudo systemd-run \
  --unit=t1562-wazuh-recovery \
  --on-active=90s \
  /usr/bin/systemctl start wazuh-agent
```

### 3. Perform the controlled interruption

The Wazuh Agent was stopped only on Linux01. A unique marker was written so the test could be separated from unrelated administrative activity.

```bash
sudo systemctl stop wazuh-agent
logger -p auth.warning "T1562_001_WAZUH_AGENT_STOPPED"
```

The operator immediately confirmed that auditd and the Splunk Universal Forwarder were still active.

### 4. Investigate in Splunk

Splunk was used to correlate:

- the `systemctl stop wazuh-agent` execution;
- auditd and sudo evidence;
- systemd service-stop and restart activity;
- the `T1562_001_WAZUH_AGENT_STOPPED` marker;
- the automatic recovery action.

The reusable searches are stored in [`splunk-searches.spl`](splunk-searches.spl).

### 5. Review Wazuh visibility

Wazuh was searched for the marker and service-related events. The exact stop event was not consistently available because the Wazuh Agent—the component responsible for forwarding endpoint events—was the service being interrupted. This is an expected sensor-availability limitation and was recorded as the main detection gap.

The reusable DQL searches and recommended columns are stored in [`wazuh-queries.txt`](wazuh-queries.txt).

### 6. Create dashboard and alerting evidence

A Splunk panel summarized the interruption timeline. A high-severity scheduled alert searched for the controlled marker, a `systemctl stop wazuh-agent` command or an audit service-stop record. The alert was added to Triggered Alerts and sent an SMTP email notification.

### 7. Validate recovery

After the recovery timer started the service, the operator verified:

- `wazuh-agent`, `auditd` and `SplunkForwarder` were active;
- the required services remained enabled;
- Linux01 reconnected to the Wazuh Manager;
- Splunk continued receiving Linux telemetry;
- SIEM ports were reachable; and
- `systemctl --failed` returned no unexpected failed units.

## Results

| Control | Result |
|---|---|
| Controlled Wazuh Agent stop | Passed |
| auditd remained active | Passed |
| Splunk Forwarder remained active | Passed |
| Independent Splunk detection | Passed |
| Splunk dashboard and alert | Passed |
| Email notification | Passed |
| Automatic Wazuh recovery | Passed |
| Wazuh self-reporting during stop | Limited by agent interruption |
| Post-test health validation | Passed |

## Detection logic and lessons learned

- A security agent cannot be relied upon as the only source for detecting its own interruption.
- Independent telemetry from auditd, systemd and a separate Splunk forwarder reduced the monitoring blind spot.
- A Wazuh agent may not immediately appear disconnected because connection-state thresholds and keepalive timing apply.
- Recovery must be configured before a controlled service interruption.
- Production detections should distinguish planned maintenance from unauthorized defense impairment and should use allowlists sparingly.

## Evidence

The `screenshots/` directory contains 14 sanitized screenshots covering baseline, interruption, SIEM investigation, dashboard, alert, email and recovery. See [`screenshots/README.md`](screenshots/README.md) for the evidence index.

## References

- [MITRE ATT&CK T1562.001 — Impair Defenses](https://attack.mitre.org/techniques/T1562/001/)
- [Wazuh agent life cycle](https://documentation.wazuh.com/current/user-manual/agent/agent-enrollment/agent-life-cycle.html)
- [Wazuh agent connection](https://documentation.wazuh.com/current/user-manual/agent/agent-management/agent-connection.html)

