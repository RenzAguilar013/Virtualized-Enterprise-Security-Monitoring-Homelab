# NIST CSF 2.0 and MITRE ATT&CK SSH Brute Force Detection Exercise

## 1. Executive Summary

This exercise simulated a controlled SSH password-guessing attack from Kali Linux against an Ubuntu endpoint inside an authorized virtualized security homelab.

The objective was to validate whether Linux authentication logs, Wazuh, Splunk Enterprise and email alerting could detect and report repeated SSH authentication failures.

The simulated activity was mapped to MITRE ATT&CK technique **T1110 - Brute Force** and sub-technique **T1110.001 - Password Guessing**. The exercise was documented using the six NIST Cybersecurity Framework 2.0 functions: Govern, Identify, Protect, Detect, Respond and Recover.

The test successfully generated SSH authentication failures from `10.10.10.40` against `10.10.10.50`. The activity was recorded by Linux, forwarded to the SIEM platforms, investigated in Splunk and Wazuh, and used to trigger a Splunk email notification.

---

## 2. Exercise Information

| Field | Value |
|---|---|
| Exercise name | SSH Brute Force Detection |
| Exercise date | [ENTER DATE] |
| Tester | Renz Nicole M. Aguilar |
| Environment | Virtualized Enterprise Security and Monitoring Homelab |
| Attacker system | Kali Linux |
| Attacker IP | `10.10.10.40` |
| Target system | Linux01 Ubuntu Server |
| Target IP | `10.10.10.50` |
| Target service | SSH |
| Destination port | TCP `22` |
| Test account | `LabTesting` |
| Detection platforms | Wazuh and Splunk Enterprise |
| Alert method | Splunk scheduled alert and SMTP email |
| Test status | [SUCCESSFUL / PARTIALLY SUCCESSFUL] |

---

## 3. Objectives

The objectives of this exercise were to:

1. Generate controlled failed SSH authentication attempts.
2. Verify that Linux recorded the authentication failures.
3. Confirm that the Splunk Universal Forwarder sent the events to Splunk.
4. Confirm that the Wazuh agent reported the activity to the Wazuh manager.
5. Identify the source IP, destination IP, destination port and target username.
6. Trigger a Splunk brute-force alert after five or more failures.
7. Receive an email notification containing investigation details.
8. Simulate containment by temporarily blocking the source IP.
9. Restore normal SSH connectivity after containment.
10. Document the complete exercise using NIST CSF 2.0 and MITRE ATT&CK.

---

## 4. Scope and Authorization

This exercise was performed only inside a privately owned and authorized virtualized homelab.

The activity was limited to the following systems:

- Kali Linux at `10.10.10.40`
- Linux01 at `10.10.10.50`
- The fictional `LabTesting` account
- SSH service on TCP port `22`
- Wazuh Manager at `10.10.10.30`
- Splunk Enterprise at `10.10.10.60`

The password list contained only fictional incorrect passwords. The test was limited to five password attempts and used one concurrent Hydra task to reduce unnecessary traffic.

No external, production or unauthorized systems were targeted.

---

## 5. Lab Systems

| System | IP address | Function |
|---|---:|---|
| FortiGate | `10.10.10.1` | Firewall, routing, NAT and DHCP |
| DC01 | `10.10.10.10` | Active Directory and DNS |
| Renz Windows 11 | `10.10.10.20` | Windows endpoint telemetry |
| Wazuh | `10.10.10.30` | SIEM manager, indexer and dashboard |
| Kali Linux | `10.10.10.40` | Authorized security-testing system |
| Linux01 | `10.10.10.50` | Ubuntu target and telemetry endpoint |
| Splunk Enterprise | `10.10.10.60` | Centralized search, dashboards and alerting |

---

## 6. MITRE ATT&CK Mapping

| Field | Mapping |
|---|---|
| Tactic | Credential Access |
| Technique | T1110 - Brute Force |
| Sub-technique | T1110.001 - Password Guessing |
| Simulated behavior | Repeated SSH password attempts against one account |
| Source system | Kali Linux |
| Target system | Linux01 |
| Detection source | Linux SSH authentication logs |
| Detection platforms | Wazuh and Splunk |

The simulated activity represented an adversary repeatedly submitting possible passwords to gain access to an SSH account.

Reference: [MITRE ATT&CK T1110 Brute Force](https://attack.mitre.org/techniques/T1110/)

---

## 7. NIST CSF 2.0 Mapping

### Govern

The exercise was authorized, documented and restricted to systems inside the homelab.

Testing limitations included:

- One authorized source system
- One authorized destination system
- One fictional account
- Five incorrect passwords
- One Hydra task
- No production systems
- No destructive activity

### Identify

The following assets and risks were identified:

| Asset | Risk |
|---|---|
| Linux01 | Unauthorized remote access |
| SSH TCP/22 | Password-guessing exposure |
| `LabTesting` account | Account compromise or lockout |
| Authentication logs | Missing or incomplete telemetry |
| Splunk and Wazuh | Detection or alerting failure |

### Protect

The following protective controls were verified before testing:

- SSH was running only on the intended target.
- A fictional account was used.
- FortiGate provided network segmentation.
- Linux authentication logging was enabled.
- The Splunk Universal Forwarder was running.
- The Wazuh agent was running.
- Splunk email settings were configured.
- The password list did not contain the correct password.

### Detect

Detection was performed using:

- `/var/log/auth.log`
- Linux SSH authentication events
- Wazuh agent telemetry
- Wazuh authentication-failure rules
- Splunk searches
- Splunk scheduled alerts
- SMTP email notifications

### Respond

The response process included:

1. Confirming the alert was generated by authorized testing.
2. Identifying the source IP.
3. Identifying the destination IP and port.
4. Identifying the targeted account.
5. Counting the failed attempts.
6. Reviewing the attack timeline.
7. Temporarily blocking the source IP.
8. Preserving screenshots and search results as evidence.

### Recover

Recovery activities included:

1. Stopping the Hydra test.
2. Removing the temporary firewall rule.
3. Checking the status of the SSH service.
4. Checking whether the test account was locked.
5. Restoring the test account when necessary.
6. Successfully reconnecting to Linux01 using SSH.
7. Confirming that Wazuh and Splunk continued receiving events.

Reference: [NIST Cybersecurity Framework 2.0](https://www.nist.gov/cyberframework)

---

## 8. Test Preparation

### 8.1 Verify SSH on Linux01

```bash
sudo systemctl status ssh
sudo ss -lntp | grep ':22'
```

### 8.2 Verify the test account

```bash
id LabTesting
```

### 8.3 Verify the Splunk Universal Forwarder

```bash
sudo /opt/splunkforwarder/bin/splunk status
```

### 8.4 Verify the Wazuh agent

```bash
sudo systemctl status wazuh-agent
```

### 8.5 Verify connectivity from Kali

```bash
ping -c 4 10.10.10.50
nmap -p 22 10.10.10.50
```

### 8.6 Create the incorrect-password list

```bash
printf '%s\n' \
'WrongPassword1!' \
'Password123!' \
'Welcome123!' \
'Admin123!' \
'Summer2026!' > passwords.txt
```

The password list contained only fictional incorrect passwords.

---

## 9. Baseline Validation

Before running Hydra, the following Splunk search was used to establish an authentication baseline:

```spl
index=* host="Linux01"
("Failed password" OR "Accepted password" OR "authentication failure")
| table _time host source sourcetype _raw
| sort -_time
```

The baseline was checked using a time range of **Last 15 minutes**.

![Splunk baseline before testing](./screenshots/01-before-test.png)

---

## 10. Test Execution

The following command was executed from Kali Linux:

```bash
hydra -l LabTesting -P passwords.txt -t 1 -W 3 -V ssh://10.10.10.50
```

Command explanation:

| Option | Purpose |
|---|---|
| `-l LabTesting` | Specifies the fictional test account |
| `-P passwords.txt` | Specifies the incorrect-password list |
| `-t 1` | Uses one concurrent task |
| `-W 3` | Waits between connection retries |
| `-V` | Displays each attempted credential |
| `ssh://10.10.10.50` | Specifies the authorized SSH target |

![Hydra password-guessing test](./screenshots/02-hydra-password-guessing.png)

---

## 11. Linux Authentication Evidence

Linux01 recorded the SSH authentication failures in `/var/log/auth.log`.

The following command was used to find events originating from Kali:

```bash
sudo grep "10.10.10.40" /var/log/auth.log | tail -20
```

The events contained:

- Target account: `LabTesting`
- Source IP: `10.10.10.40`
- Destination host: Linux01
- Authentication result: Failed password
- Service: SSH

![Linux SSH authentication log](./screenshots/03-linux-auth-log.png)

---

## 12. Splunk Investigation

### 12.1 Detailed failed-login search

```spl
index=* host="Linux01"
("Failed password" OR "authentication failure")
| rex field=_raw "Failed password for (?:invalid user )?(?<User>\S+) from (?<SourceIP>\d{1,3}(?:\.\d{1,3}){3})"
| search SourceIP="10.10.10.40"
| eval EventType="SSH Failed Login"
| eval DestinationIP="10.10.10.50"
| eval DestinationPort="22"
| table _time EventType User SourceIP DestinationIP DestinationPort host
| sort -_time
```

### 12.2 Failed-attempt summary

```spl
index=* host="Linux01" "Failed password"
| rex field=_raw "Failed password for (?:invalid user )?(?<User>\S+) from (?<SourceIP>\d{1,3}(?:\.\d{1,3}){3})"
| search SourceIP="10.10.10.40"
| eval DestinationIP="10.10.10.50"
| stats count AS FailedAttempts
        earliest(_time) AS FirstSeen
        latest(_time) AS LastSeen
        BY SourceIP DestinationIP User host
| convert ctime(FirstSeen) ctime(LastSeen)
| table SourceIP DestinationIP User host FailedAttempts FirstSeen LastSeen
```

### 12.3 Splunk findings

| Field | Result |
|---|---|
| Source IP | `10.10.10.40` |
| Destination IP | `10.10.10.50` |
| Destination port | TCP `22` |
| Target user | `LabTesting` |
| Failed attempts | [ENTER COUNT] |
| First observed | [ENTER TIME] |
| Last observed | [ENTER TIME] |
| Splunk host | `Linux01` |

![Splunk failed-login investigation](./screenshots/04-splunk-failed-logins.png)

---

## 13. Wazuh Investigation

The following Wazuh query was used:

```text
agent.name: "UBUNTU-10.10.10.50" AND data.srcip: "10.10.10.40"
```

An authentication-focused query was also used:

```text
agent.name: "UBUNTU-10.10.10.50" AND rule.groups: "authentication_failed"
```

The following fields were reviewed:

- `timestamp`
- `agent.name`
- `rule.id`
- `rule.level`
- `rule.description`
- `data.srcip`
- `data.dstuser`
- `full_log`

### Wazuh findings

| Field | Result |
|---|---|
| Agent | `UBUNTU-10.10.10.50` |
| Source IP | `10.10.10.40` |
| Target user | `LabTesting
| Rule ID | [ENTER RULE ID] |
| Rule level | [ENTER RULE LEVEL] |
| Rule description | [ENTER DESCRIPTION] |

![Wazuh SSH detection](./screenshots/05-wazuh-ssh-detection.png)

---

## 14. Splunk Alert Configuration

The following SPL generated an alert when five or more failures occurred within five minutes:

```spl
index=* host="Linux01" "Failed password"
| rex field=_raw "Failed password for (?:invalid user )?(?<User>\S+) from (?<SourceIP>\d{1,3}(?:\.\d{1,3}){3})"
| eval DestinationIP="10.10.10.50"
| eval DestinationPort="22"
| bin _time span=5m
| stats count AS FailedAttempts BY _time SourceIP DestinationIP DestinationPort User host
| where FailedAttempts >= 5
| eval Severity="HIGH"
| eval Detection="Possible SSH password-guessing attack"
| table _time Severity Detection SourceIP DestinationIP DestinationPort User host FailedAttempts
```

### Alert settings

| Setting | Configuration |
|---|---|
| Alert name | Linux SSH Brute Force Detected |
| Alert type | Scheduled |
| Cron schedule | `*/1 * * * *` |
| Search window | Last five minutes |
| Trigger condition | Results greater than zero |
| Trigger mode | Once |
| Throttle period | Ten minutes |
| Throttle fields | SourceIP, DestinationIP and User |
| Severity | High |
| Actions | Add to Triggered Alerts and Send Email |

![Splunk triggered alert](./screenshots/06-splunk-triggered-alert.png)

---

## 15. Email Notification

Splunk sent an SMTP notification containing:

- Alert severity
- Detection description
- Source IP
- Destination IP
- Destination port
- Target username
- Affected host
- Failed-attempt count
- Detection time
- MITRE ATT&CK mapping
- Splunk investigation link

The successful email notification confirmed that the complete detection and notification workflow was operational.

![Splunk brute-force email notification](./screenshots/07-brute-force-email-alert.png)

---

## 16. Simulated Containment

The Kali source IP was temporarily blocked from connecting to SSH on Linux01:

```bash
sudo iptables -I INPUT -s 10.10.10.40 -p tcp --dport 22 -j DROP
```

The rule was verified using:

```bash
sudo iptables -L INPUT -n --line-numbers
```

An SSH connection was attempted from Kali to verify that the containment rule was effective:

```bash
ssh LabTesting@10.10.10.50
```

The connection failed or timed out while the containment rule was active.

![Temporary source-IP containment](./screenshots/08-source-ip-contained.png)

---

## 17. Recovery Validation

The temporary containment rule was removed:

```bash
sudo iptables -D INPUT -s 10.10.10.40 -p tcp --dport 22 -j DROP
```

The SSH service was verified:

```bash
sudo systemctl status ssh --no-pager
```

If necessary, the fictional account lockout was reset:

```bash
sudo faillock --user LabTesting --reset
```

A successful SSH connection was established from Kali:

```bash
ssh LabTesting@10.10.10.50
```

After connecting, the following commands were used to verify the user and host:

```bash
hostname
whoami
ip -br address
```

![Successful SSH recovery validation](./screenshots/09-ssh-service-recovered.png)

---

## 18. Results

| Validation requirement | Result |
|---|---|
| Hydra generated controlled failures | [PASS / FAIL] |
| Linux recorded the failed logins | [PASS / FAIL] |
| Splunk received the authentication logs | [PASS / FAIL] |
| Splunk extracted the source IP | [PASS / FAIL] |
| Splunk displayed the destination IP | [PASS / FAIL] |
| Wazuh detected the SSH failures | [PASS / FAIL] |
| Splunk alert triggered | [PASS / FAIL] |
| Alert appeared under Triggered Alerts | [PASS / FAIL] |
| Email notification was received | [PASS / FAIL] |
| Source-IP containment was successful | [PASS / FAIL] |
| SSH connectivity was restored | [PASS / FAIL] |

### Overall result

```text
[ENTER OVERALL RESULT]
```

Example:

```text
The exercise was successful. Five controlled SSH password failures were generated from Kali, recorded by Linux01, forwarded to Wazuh and Splunk, and used to trigger a Splunk email notification. The source IP was temporarily contained, and normal SSH connectivity was successfully restored.
```

---

## 19. Problems Encountered

During the exercise, the following problem was encountered:

```text
The Splunk email notification was received, but the alert initially did not appear on the Triggered Alerts page.
```

### Cause

The alert had a **Send Email** action, but the separate **Add to triggered alerts** action was not enabled.

### Resolution

The alert was edited and the following action was added:

```text
Add to triggered alerts
```

The severity was set to **High**, and the alert was tested again. Subsequent detections appeared on the Triggered Alerts page.

---

## 20. Lessons Learned

This exercise demonstrated that:

1. Email delivery confirms that an alert action executed, but Splunk requires a separate tracking action to display it under Triggered Alerts.
2. Source and destination IP fields are important for understanding the direction of an authentication attack.
3. Alert thresholds reduce noise from isolated password mistakes.
4. Throttling prevents repeated notifications for the same activity.
5. Wazuh and Splunk provide complementary detection and investigation capabilities.
6. Containment must be followed by recovery validation.
7. Screenshots and saved queries provide repeatable evidence of the detection workflow.
8. Detection testing should use fictional accounts and tightly controlled password lists.

---

## 21. Recommended Improvements

Future improvements include:

- Configure an SSH account-lockout policy.
- Detect successful authentication following repeated failures.
- Add failed and successful SSH activity to the Splunk dashboard.
- Create a Wazuh custom rule for the exact lab threshold.
- Use FortiGate for centralized source-IP containment.
- Add GeoIP enrichment for external source addresses.
- Create a lookup containing authorized lab systems.
- Add risk scoring based on account, host and attempt count.
- Monitor attacks against multiple usernames.
- Compare password guessing with password spraying.
- Measure mean time to detect and mean time to respond.
- Export the alert and investigation results as repeatable evidence.

---

## 22. Conclusion

The SSH brute-force detection exercise successfully validated an end-to-end security-monitoring workflow inside the virtualized enterprise homelab.

The exercise began with controlled password-guessing activity from Kali Linux and continued through Linux authentication logging, Wazuh detection, Splunk investigation, alert generation, email notification, simulated containment and recovery validation.

Mapping the activity to MITRE ATT&CK provided a technical description of the simulated adversary behavior. Mapping the exercise to NIST CSF 2.0 demonstrated how the security controls, detection processes, response actions and recovery procedures support a structured cybersecurity risk-management program.

---

## 23. Evidence Files

```text
screenshots/
├── 01-before-test.png
├── 02-hydra-password-guessing.png
├── 03-linux-auth-log.png
├── 04-splunk-failed-logins.png
├── 05-wazuh-ssh-detection.png
├── 06-splunk-triggered-alert.png
├── 07-brute-force-email-alert.png
├── 08-source-ip-contained.png
└── 09-ssh-service-recovered.png
```

---

## 24. References

- [NIST Cybersecurity Framework 2.0](https://www.nist.gov/cyberframework)
- [NIST SP 800-61 Revision 3](https://csrc.nist.gov/pubs/sp/800/61/r3/final)
- [MITRE ATT&CK T1110 Brute Force](https://attack.mitre.org/techniques/T1110/)
- [MITRE ATT&CK T1110.001 Password Guessing](https://attack.mitre.org/techniques/T1110/001/)
