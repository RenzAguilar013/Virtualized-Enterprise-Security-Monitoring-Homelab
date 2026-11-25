# A06 Portfolio Evidence

Use the compact eight-capture story below. Repeated editor, backup, configuration
test and restart screenshots are not additional portfolio requirements.

## Published and reviewed in this update

| File | Evidence |
|---|---|
| [03-a06-splunk-basket-telemetry.png](03-a06-splunk-basket-telemetry.png) | Six indexed JSON records; actual index/source/sourcetype; quantity 1 and HTTP 200/401 fields |
| [06a-a06-scheduler-email-action-diagnostic.png](06a-a06-scheduler-email-action-diagnostic.png) | Supplemental troubleshooting evidence: successful scheduled runs, one matching result and email action selection; not an inbox receipt |

The supplied ingestion image uses a broader diagnostic search than the cleaned
Screenshot 03 query in `splunk-searches.spl`. Both identify the same A06 source.

## Core portfolio filenames

| Filename | Purpose / availability |
|---|---|
| `01-a06-normal-basket-baseline.png` | Own-account normal basket; capture not supplied |
| `02-a06-negative-quantity-validation.png` | Negative update and actual basket state; behavior operator-confirmed, capture not supplied |
| `03-a06-splunk-basket-telemetry.png` | Published ingestion evidence |
| `04-a06-splunk-alert-settings.png` | Saved schedule, condition, suppression and actions; capture not supplied |
| `05-a06-wazuh-negative-quantity-alert.png` | Live rule 119995 with quantity/source/status; capture not supplied |
| `06-a06-splunk-triggered-alert.png` | Triggered-alert listing and time; capture not supplied; 06a is supplemental scheduler evidence |
| `07-a06-email-notification.png` | Received message and results; notification reported as appearing, capture not supplied |
| `08-a06-recovery-validation.png` | Restored basket and fresh quantity-1 telemetry; operator reported completion, capture not supplied |

The operator confirmed the negative basket state and restoration and declared
the activity complete. These reported outcomes are distinguished from the two
images independently reviewed here. No replacement screenshots are fabricated.

## Publication handling

Exclude Authorization headers, session cookies and real credentials. Hide
personal email addresses in inbox/log captures. Preserve readable quantities,
status codes, rule IDs, event times and the relevant source/asset fields.

**The basket was restored per operator confirmation. The server-side validation
finding remains open; no application remediation is claimed.**
