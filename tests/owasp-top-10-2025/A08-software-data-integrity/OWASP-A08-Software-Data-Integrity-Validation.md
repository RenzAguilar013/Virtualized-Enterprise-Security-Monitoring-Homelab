# OWASP A08:2025 Software or Data Integrity Failures Validation

Activity date: **25 November 2026 (Asia/Manila)**. Recorded gate timestamps are
UTC. The initial successful stage at 07:17:36 UTC corresponds to 15:17:36 Manila.

## Objective and outcome

Validate a signature-verification boundary for a small release artifact:
accept the approved signed file, reject a modified copy using the original
signature, preserve the approved staged content, and correlate the real
verification outcome through Wazuh, Splunk and a contextual email.

**Completion:** The operator reported the A08 workflow complete, including the
live Wazuh alert after the rule fix, the scheduled Splunk alert, received email,
and final recovery capture. Supplied text independently establishes valid
initial staging, blocked modified candidates, unchanged active hashes, a
connected agent, and the corrected full-record Wazuh rule match. The eight
screenshots remain to upload; see the [evidence checklist](screenshots/README.md).

## Scope and environment

This exercise used a harmless text release and a separate lab staging directory.
It did not replace Juice Shop files, deploy application code, modify a container
image or demonstrate a compromised production build pipeline.

| Component | Address / path | Function |
|---|---|---|
| Kali | `10.10.10.40`, `~/a08-integrity/` | Artifact publisher; disposable RSA-3072 signing key |
| WEB01 | `10.10.10.70`, `~/a08-integrity/incoming/` | Receives artifact, detached signature and public key |
| Trusted key baseline | `/etc/a08-integrity/` on WEB01 | Root-owned public key and pinned SHA-256 baseline |
| Verification gate | `/usr/local/sbin/a08-verify-stage` on WEB01 | Verifies candidate bytes before atomic staging |
| Staged artifact | `/opt/a08-integrity-demo/staged/release.txt` | Approved text artifact, separate from the live application |
| Audit source | `/var/log/a08-integrity/verification.jsonl` | One JSON verification record per gate run |
| Wazuh | `10.10.10.30`, version `4.14.7` observed | Live blocked-artifact detection, rule `119997` at level 10 |
| Splunk | `10.10.10.60`, index `web`, sourcetype `a08:integrity` | Verification search, scheduled alert and email |

**OWASP mapping:** A08 focuses on trusting software/data without verifying
integrity. This exercise validates the preventive signature check and its
monitoring. SHA-256 values support comparison; the trusted public key and
signature establish the verification boundary. The key's received file hash
was manually compared with the publisher baseline before it was pinned.

## Approved artifact and transfer

Kali's `publisher/release.txt` contained:

```text
A08 homelab release
version=1.0
```

The private key stayed in Kali's `private/` directory, mode 0700, with key mode
0600. Artifact, signature and public key were transferred to WEB01 using SCP;
both received file hashes matched the recorded Kali baselines per the operator.
The public-key hash value was not supplied in the transcript and is not invented here.

Run these publisher commands in the **Kali terminal**, from `~/a08-integrity`.
The key-generation commands describe the original setup; do not rerun them
when testing modified input or comparing against the existing trusted baseline.

```bash
openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:3072 \
  -out private/lab-signing-private.pem
openssl pkey -in private/lab-signing-private.pem -pubout \
  -out publisher/lab-signing-public.pem
openssl dgst -sha256 -sign private/lab-signing-private.pem \
  -out publisher/release.txt.sig publisher/release.txt
openssl dgst -sha256 -verify publisher/lab-signing-public.pem \
  -signature publisher/release.txt.sig publisher/release.txt
sha256sum publisher/release.txt publisher/lab-signing-public.pem
scp publisher/release.txt publisher/release.txt.sig \
  publisher/lab-signing-public.pem \
  web01@10.10.10.70:a08-integrity/incoming/
```

After the baseline comparison, the **WEB01 terminal** setup pinned the key:

```bash
cd ~/a08-integrity
sudo install -d -o root -g root -m 0755 /etc/a08-integrity
sudo install -o root -g root -m 0644 incoming/lab-signing-public.pem \
  /etc/a08-integrity/lab-signing-public.pem
sudo sha256sum /etc/a08-integrity/lab-signing-public.pem \
  | sudo tee /etc/a08-integrity/trusted-key.sha256
sudo sha256sum -c /etc/a08-integrity/trusted-key.sha256
openssl dgst -sha256 -verify /etc/a08-integrity/lab-signing-public.pem \
  -signature incoming/release.txt.sig incoming/release.txt
```

The operator reported both the pinned-key check and signature verification passed.

## Verification gate

The root-run gate locks the staging workflow, validates the pinned public-key
hash, snapshots candidate/signature bytes into a private temporary directory,
verifies the candidate with OpenSSL, and uses `os.replace` to stage only verified
bytes. It records the candidate hash, before/after active hashes, outcome, reason,
OpenSSL exit code and a unique run ID. Nonzero exit status is expected for a
blocked test; a failed test must not be treated as successful staging.

The installed lab implementation is embedded here to retain the established
five-entry activity-folder layout. Its CLI requires root. Candidate and signature
inputs must be regular files, direct symlinks are rejected, and each input is
limited to 1 MiB. It is a training gate with a local root-owned trust baseline;
release freshness, signing-key rotation and full CI/CD enforcement are outside
this validation.

Save the following source as `/usr/local/sbin/a08-verify-stage` on WEB01, owned
by root with mode 0755. The audit directory is root-owned mode 0755 and the JSONL
file is mode 0644 so the existing collectors can read it.

```python
#!/usr/bin/env python3
import datetime
import fcntl
import hashlib
import json
import os
from pathlib import Path
import stat
import subprocess
import sys
import tempfile
import uuid


def digest(path):
    return hashlib.sha256(path.read_bytes()).hexdigest() if path.exists() else "absent"


def snapshot(source, destination):
    fd = os.open(source, os.O_RDONLY | os.O_NOFOLLOW | os.O_NONBLOCK)
    with os.fdopen(fd, "rb") as stream:
        if not stat.S_ISREG(os.fstat(stream.fileno()).st_mode):
            raise ValueError("candidate_or_signature_is_not_a_regular_file")
        data = stream.read(1048577)
    if len(data) > 1048576:
        raise ValueError("lab_input_exceeds_1_MiB")
    destination.write_bytes(data)


def run_gate(candidate, signature, trusted, stage, log):
    stage.mkdir(parents=True, exist_ok=True, mode=0o755)
    log.parent.mkdir(parents=True, exist_ok=True, mode=0o755)
    with (stage / ".gate.lock").open("a") as lock:
        fcntl.flock(lock, fcntl.LOCK_EX)
        active = stage / "release.txt"
        event = {
            "integration": "a08_integrity", "asset": "WEB01",
            "event_type": "artifact_verification", "artifact": "release.txt",
            "timestamp": datetime.datetime.now(datetime.timezone.utc).isoformat(),
            "run_id": uuid.uuid4().hex, "verification_result": "not_checked",
            "deployment_action": "blocked", "candidate_sha256": "unavailable",
            "active_sha256_before": digest(active),
        }
        try:
            key = (trusted / "lab-signing-public.pem").read_bytes()
            expected = (trusted / "trusted-key.sha256").read_text().split()[0]
            if hashlib.sha256(key).hexdigest() != expected:
                raise ValueError("trusted_key_hash_mismatch")
            with tempfile.TemporaryDirectory(prefix=".a08-", dir=stage) as tmp:
                tmp = Path(tmp)
                snapshot(candidate, tmp / "release.txt")
                snapshot(signature, tmp / "release.sig")
                (tmp / "key.pem").write_bytes(key)
                event["candidate_sha256"] = digest(tmp / "release.txt")
                check = subprocess.run([
                    "/usr/bin/openssl", "dgst", "-sha256", "-verify", str(tmp / "key.pem"),
                    "-signature", str(tmp / "release.sig"), str(tmp / "release.txt"),
                ], capture_output=True, text=True, timeout=10)
                event["openssl_exit_code"] = check.returncode
                event["verification_result"] = "passed" if check.returncode == 0 else "failed"
                if check.returncode != 0:
                    raise ValueError("signature_verification_failed")
                (tmp / "release.txt").chmod(0o644)
                os.replace(tmp / "release.txt", active)
                event.update(deployment_action="staged", reason="signature_verified")
        except Exception as error:
            event["reason"] = str(error)[:200]
        event["active_sha256_after"] = digest(active)
        event["staged_content_changed"] = event["active_sha256_before"] != event["active_sha256_after"]
        with log.open("a") as stream:
            stream.write(json.dumps(event) + "\n")
        print(json.dumps(event, indent=2))
        return 0 if event["deployment_action"] == "staged" else 1


if __name__ == "__main__":
    if os.geteuid() != 0 or len(sys.argv) != 3:
        sys.exit("Usage: sudo a08-verify-stage CANDIDATE SIGNATURE")
    sys.exit(run_gate(
        Path(sys.argv[1]), Path(sys.argv[2]), Path("/etc/a08-integrity"),
        Path("/opt/a08-integrity-demo/staged"),
        Path("/var/log/a08-integrity/verification.jsonl"),
    ))
```

## Baseline, modified-input test and observed hashes

**WEB01 terminal — successful approved input:**

```bash
cd ~/a08-integrity
sudo /usr/local/sbin/a08-verify-stage \
  "$PWD/incoming/release.txt" "$PWD/incoming/release.txt.sig"
```

The supplied first result at `2026-11-25T07:17:36.188612+00:00` showed:

- Run `5e841afc71ad445db40f0bac95d27c38`.
- `verification_result=passed`, `deployment_action=staged`, `reason=signature_verified`.
- OpenSSL exit code `0`; active hash changed from `absent` to the approved hash.
- `staged_content_changed=true`, expected on first creation.

**WEB01 terminal — modify a copy, retain the original signature:**

```bash
cd ~/a08-integrity
cp incoming/release.txt incoming/release-tampered.txt
printf '\nmodified=true\n' >> incoming/release-tampered.txt
sudo /usr/local/sbin/a08-verify-stage \
  "$PWD/incoming/release-tampered.txt" "$PWD/incoming/release.txt.sig"
```

| Value | SHA-256 |
|---|---|
| Approved artifact / active before and after rejected runs | `9eb0fc49786c63f9f5f0eb21858eb74bd6d1a34a6ec7563047bc0b861e39b43c` |
| Modified candidate | `04d480ade16776a14f84813304b6d3d65873587cfb884ef5849989a09400653a` |

The supplied rejected runs were `b0aa66ab024247cf92dc8eb14be52b91` at 08:28:40 UTC,
`9b5387a6597e4874b00e051334e6cbe4` at 08:36:59 UTC, and
`a2ab65423ef14c87aa27814aa94f268a` at 08:42:02 UTC. Each showed
`failed`, `blocked`, `signature_verification_failed`, OpenSSL exit code `1`,
identical active before/after hashes, and `staged_content_changed=false`.

These are real gate outputs. They are not fabricated SIEM alerts. A new execution
generates a new run ID for correlation and alert-suppression handling.

## Wazuh collection, rule conflict and fix

WEB01's `/var/ossec/etc/ossec.conf` includes:

```xml
<localfile>
  <log_format>json</log_format>
  <location>/var/log/a08-integrity/verification.jsonl</location>
  <only-future-events>yes</only-future-events>
</localfile>
```

Supplied agent logs showed the file being analyzed from 07:25:56 UTC. The agent
state was connected, with recent acknowledgments and an empty message buffer.
Nevertheless, the pre-fix runs were absent from the manager's real alert file.
An initial `grep` result was the sudo audit of the search command itself, not
an A08 alert. Structured inspection of `data.run_id` confirmed the absence.

The first shortened JSON test matched custom rule `119997`, but omitted the
real event's `timestamp` field. Testing the **complete actual JSON line** exposed
the cause: built-in rule `86600` classified JSON with `timestamp` and `event_type`
as a Suricata message at level 0. The original standalone A08 JSON parent was
not reached on that branch. This was a rule-routing collision, not proof of a
Suricata sensor producing the event.

The fix changed custom parent `119996` from `decoded_as=json` to
`if_sid=86600`, retaining exact checks for integration, asset and event type.
Rule `119997` remains its blocked-outcome child. The corrected
[rule file](wazuh-rules.xml) is installed on the Wazuh server at
`/var/ossec/etc/rules/a08_integrity_rules.xml`.

**Wazuh server terminal — validation and reload:**

```bash
sudo /var/ossec/bin/wazuh-analysisd -t
sudo /var/ossec/bin/wazuh-logtest
```

Paste the complete single-line JSON record from WEB01's audit file into logtest.
The supplied corrected Phase 3 for historical run `a2ab65423ef14c87aa27814aa94f268a`
matched **119997, level 10**, with the A08 blocked-artifact description and groups
`local`, `owasp_a08`, `software_integrity`, `artifact_blocked`.

After leaving logtest, reload the validated rules:

```bash
sudo systemctl restart wazuh-manager
sudo systemctl is-active wazuh-manager
```

Wait for WEB01's agent to reconnect, then execute a fresh modified-input gate
test on WEB01. Because `only-future-events=yes`, restarting or fixing rules
does not turn an already consumed historical event into a new live alert.

In **Wazuh website → Threat Hunting**, select Last 15 minutes and query:

```text
rule.id: "119997"
```

The operator confirmed the live-alert step after the fix and moved to recovery.
The post-fix live run ID and screenshot were not supplied for independent review.
The final rule flags any blocked gate attempt; inspect `reason` to distinguish
signature failure from a missing input or changed trusted-key baseline.
Successful staging stays at parent level 0 and normally has no Wazuh dashboard
alert. `mail=False` in logtest refers to Wazuh's own mail action; the activity's
email is sent by Splunk.

## Splunk collection and parsing

**WEB01 Universal Forwarder**, app
`/opt/splunkforwarder/etc/apps/a08_integrity/local/inputs.conf`:

```ini
[monitor:///var/log/a08-integrity/verification.jsonl]
disabled = false
index = web
sourcetype = a08:integrity
```

**Splunk server**, app `/opt/splunk/etc/apps/a08_integrity/local/props.conf`:

```ini
[a08:integrity]
SHOULD_LINEMERGE = false
LINE_BREAKER = ([\r\n]+)
DATETIME_CONFIG = CURRENT
KV_MODE = json
```

App `metadata/default.meta` exports the sourcetype configuration:

```ini
[props]
export = system
```

The operator supplied the effective parsing settings and confirmed the forwarder
and server running. JSON is extracted by `KV_MODE=json`; the final searches do
not redundantly apply `spath`. `_time` is receipt time under this configuration;
the JSON `timestamp` retains the gate's UTC event time.

In the **Splunk website → Search & Reporting search box**, use the searches in
[splunk-searches.spl](splunk-searches.spl). Do not paste SPL into a Linux shell.

## Scheduled alert and contextual email

| Setting | Configured target |
|---|---|
| Name | `OWASP A08 - Artifact Verification Blocked` |
| Type / schedule | Scheduled, cron `* * * * *` |
| Time window | Earliest `-5m`, latest `now` |
| Condition | Number of Results greater than 0 |
| Trigger mode | For each result |
| Suppression | 5 minutes, fields `asset,artifact,run_id` |
| Actions | Add to Triggered Alerts, High; Send Email with inline results |

```spl
index=web sourcetype="a08:integrity" earliest=-5m latest=now
integration="a08_integrity" event_type="artifact_verification"
deployment_action="blocked"
| dedup asset run_id
| eval staging_unchanged=if(active_sha256_before=active_sha256_after,"yes","no")
| table timestamp asset artifact run_id verification_result deployment_action reason staging_unchanged
```

Per-result suppression prevents the same run being emailed again on overlapping
scheduled searches; a new gate execution has a fresh run ID. Matching scheduler
results or selecting the email action alone does not establish delivery. The
operator explicitly confirmed both the triggered Splunk alert and receipt of
the email during Wazuh troubleshooting. The settings/image remain to upload.

Email subject:

```text
OWASP A08 - Artifact Verification Blocked
```

Email message:

```text
OWASP A08:2025 — Software or Data Integrity Failures

The artifact-verification gate blocked a staging attempt.

Event time: $result.timestamp$
Asset: $result.asset$
Artifact: $result.artifact$
Run ID: $result.run_id$
Verification result: $result.verification_result$
Staging decision: $result.deployment_action$
Reason: $result.reason$
Staged content unchanged: $result.staging_unchanged$

Investigation:
Correlate this run ID with Wazuh rule 119997 and the verification log.
Review the rejection reason, trusted-key baseline and artifact source.
Confirm that the staged artifact remains the approved version.
```

The existing lab mail recipient/settings are reused. No credentials or personal
recipient address are included in this report.

## Recovery and operational validation

Recovery validates that the approved input still passes and the staged artifact
remains correct after rejection. Since the bad input was blocked, no replacement
of compromised application code was required.

**WEB01 terminal:**

```bash
cd ~/a08-integrity
sudo /usr/local/sbin/a08-verify-stage \
  "$PWD/incoming/release.txt" "$PWD/incoming/release.txt.sig"
sha256sum incoming/release.txt /opt/a08-integrity-demo/staged/release.txt
sudo nginx -t
for service in nginx docker wazuh-agent; do
  printf '%s: ' "$service"
  systemctl is-active "$service"
done
sudo /opt/splunkforwarder/bin/splunk status
curl --noproxy '*' -sS --connect-timeout 5 --max-time 10 \
  -o /dev/null -w 'Application HTTP status: %{http_code}\n' \
  'http://10.10.10.70/rest/products/search?q=apple'
```

Expected: `passed`, `staged`, `signature_verified`, unchanged approved hash,
successful Nginx validation, active Nginx/Docker/Wazuh Agent, running Splunk
forwarder and HTTP 200. The first stage changed content from absent; later
successful restaging of the same approved file reports changed=false.

**Splunk website recovery search:**

```spl
index=web sourcetype="a08:integrity" earliest=-15m latest=now
integration="a08_integrity" verification_result="passed" deployment_action="staged"
| sort 0 -_time
| head 1
| table timestamp asset artifact verification_result deployment_action staged_content_changed run_id
```

Match the recovery row to the new successful gate run ID. Screenshot 08 combines
the terminal validation with that row. Recovery completion was operator-reported;
the final output/image has not yet been independently reviewed. The gate and
collection configuration stay available for subsequent lab validation.

## Evidence and lessons learned

| Evidence | Status at publication |
|---|---|
| Publisher signature and transfer baseline checks | Operator-confirmed |
| First approved stage and three rejected test outputs | Supplied text reviewed |
| Before/after approved hash preservation | Supplied text reviewed |
| Agent connection, log collector source and parsing settings | Supplied text reviewed |
| Full-record collision with 86600 and corrected 119997 / level 10 match | Supplied text reviewed |
| Post-fix live Wazuh alert | Operator-confirmed; image pending |
| Scheduled Splunk trigger and received email | Explicitly operator-confirmed; images pending |
| Final known-good recovery and operational-health capture | Operator-confirmed; image pending |

Test detections against the complete emitted record. A minimal synthetic sample
can pass while real events follow a different rule branch. A connected agent,
a successful logtest and a working parallel Splunk path each establish different
parts of the workflow; none alone proves live Wazuh alert generation. Structured
`data.run_id` matching avoids mistaking a sudo search audit for the target event.

The control validation does not establish arbitrary release safety, freshness,
private-key compromise resistance or comprehensive A08 coverage. Existing A04
HTTPS and A06 quantity-validation findings are not remediated by this activity.

## CV-ready achievement

- Implemented an OpenSSL RSA/SHA-256 artifact-verification gate in a Kali/Linux
  homelab, blocking modified releases while preserving the approved staged artifact.
- Integrated verification telemetry with Wazuh and Splunk, resolved a JSON rule
  hierarchy conflict, and validated scheduled alerts, contextual email and recovery.

## References

- [OWASP A08:2025 — Software or Data Integrity Failures](https://top10.owasp.org/2025/A08_2025-Software_or_Data_Integrity_Failures/)
- [OpenSSL dgst signature operations](https://docs.openssl.org/3.3/man1/openssl-dgst/)
- [Wazuh rules syntax: if_sid and field conditions](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/rules.html)
- [Wazuh custom-rule validation and loading](https://documentation.wazuh.com/current/user-manual/ruleset/rules/custom.html)
