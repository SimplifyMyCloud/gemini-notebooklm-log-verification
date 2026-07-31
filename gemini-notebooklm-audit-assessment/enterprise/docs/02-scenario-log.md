← [Back: 01 — Methodology](01-methodology.md) | [README](../../README.md) | Next: [03 — Findings](03-findings.md) →

# Gemini Notebook Enterprise — Scenario Log

## Scenario A — Enable full observability logging via API

Consistent with the theme of this whole assessment series: enable via API, not console. This is the documented `ObservabilityConfig` mechanism for this product.

```bash
gcloud auth print-access-token
```
Confirmed a fresh token was available before proceeding.

```bash
TOKEN=$(gcloud auth print-access-token)

curl -X PATCH \
  -H "Authorization: Bearer ${TOKEN}" \
  -H "Content-Type: application/json" \
  -H "X-Goog-User-Project: simplifymycloud-dev" \
  "https://us-discoveryengine.googleapis.com/v1alpha/projects/simplifymycloud-dev?updateMask=customerProvidedConfig.notebooklmConfig.observabilityConfig" \
  -d '{
    "customerProvidedConfig": {
      "notebooklmConfig": {
        "observabilityConfig": {
          "observabilityEnabled": true,
          "sensitiveLoggingEnabled": true
        }
      }
    }
  }'
```

Response:
```json
{
  "name": "projects/288261943767",
  "createTime": "2026-06-02T14:13:36.092294785Z",
  "provisionCompletionTime": "2026-06-02T14:13:36.092294785Z",
  "customerProvidedConfig": {
    "notebooklmConfig": {
      "observabilityConfig": {
        "observabilityEnabled": true,
        "sensitiveLoggingEnabled": true
      }
    }
  }
}
```

Both `observabilityEnabled` and `sensitiveLoggingEnabled` confirmed `true` in the same API response — the **maximum logging configuration available** for this product, matching the customer's stated "enable everything available" requirement.

## Scenario B — Real usage test: Bitcoin whitepaper notebook

To get a realistic, demo-worthy test rather than a synthetic string, a real notebook was built with actual technical source material:

- **Source:** Bitcoin: A Peer-to-Peer Electronic Cash System (Satoshi Nakamoto's original whitepaper)

**Prompt sent** (via the Gemini Notebook Enterprise user interface), tagged with a distinctive marker:

```
nblogtest01 - explain in one paragraph how Satoshi's paper solves the double-spending problem without a trusted third party
```

Gemini generated a real, source-grounded response citing the uploaded whitepaper. Immediately after:

```bash
date -u
```
```
Fri Jul 31 18:39:02 UTC 2026
```

## Scenario C — Log verification

**Check 1 — broad service-level audit log:**

```bash
gcloud logging read \
  'protoPayload.serviceName="discoveryengine.googleapis.com"' \
  --project=simplifymycloud-dev \
  --freshness=15m \
  --format=json > /tmp/nb-check1.json

wc -l /tmp/nb-check1.json
```
```
4398 /tmp/nb-check1.json
```
A large volume of general Discovery Engine service activity — not yet filtered to the specific usage-logging stream.

**Check 2 — dedicated log name pattern for this service:**

```bash
gcloud logging read \
  'logName=~"projects/simplifymycloud-dev/logs/discoveryengine.googleapis.com.*"' \
  --project=simplifymycloud-dev \
  --freshness=15m \
  --format=json > /tmp/nb-check2.json

wc -l /tmp/nb-check2.json
grep -i "nblogtest01" /tmp/nb-check2.json
```
```
251 /tmp/nb-check2.json
        "userQuery": "nblogtest01 - explain in one paragraph how Satoshi's paper solves the double-spending problem without a trusted third party"
```

**Match found.** Full prompt text captured verbatim.

**Check 3 — list the actual log streams that exist for this service:**

```bash
gcloud logging logs list --project=simplifymycloud-dev --filter="discoveryengine"
```
```
NAME
projects/simplifymycloud-dev/logs/discoveryengine.googleapis.com%2Fgemini_enterprise_user_activity
projects/simplifymycloud-dev/logs/discoveryengine.googleapis.com%2Fnotebooklm_enterprise_user_activity
```

Two distinct log streams exist. Our marker landed in the **NotebookLM-specific** stream (`notebooklm_enterprise_user_activity`), not the generic `gemini_enterprise_user_activity` stream — worth knowing if building log-routing filters, since a filter scoped only to the generic stream would miss NotebookLM activity entirely.

**Full log entry, pulled and inspected field by field:**

```bash
python3 -c "
import json
with open('/tmp/nb-check2.json') as f:
    data = json.load(f)
for entry in data:
    payload = str(entry.get('jsonPayload', {}))
    if 'nblogtest01' in payload:
        print(json.dumps(entry, indent=2))
"
```

```json
{
  "insertId": "auat7ce5p56y",
  "jsonPayload": {
    "logMetadata": {
      "methodName": "GenerateFreeFormStreamed",
      "name": "projects/288261943767/locations/us/notebooks/e437af8f-291e-40b8-90fb-17e412facbfc",
      "serviceLabel": "NOTEBOOKLM_ENTERPRISE",
      "serviceName": "google.cloud.notebooklm.v1main.NotebookService",
      "timestamp": "2026-07-31T18:30:00.149015368Z"
    },
    "request": {
      "name": "projects/288261943767/locations/us/notebooks/e437af8f-291e-40b8-90fb-17e412facbfc",
      "userQuery": "nblogtest01 - explain in one paragraph how Satoshi's paper solves the double-spending problem without a trusted third party"
    },
    "response": {},
    "serviceTextReply": "Satoshi Nakamoto's paper solves the double-spending problem... [full response text, verbatim, with source citations [1]-[5] — see note below on a duplication artifact in this field]",
    "userIamPrincipal": "chris@simplifymy.cloud"
  },
  "logName": "projects/simplifymycloud-dev/logs/discoveryengine.googleapis.com%2Fnotebooklm_enterprise_user_activity",
  "receiveTimestamp": "2026-07-31T18:30:00.394001167Z",
  "resource": {
    "labels": {
      "credential_id": "",
      "location": "us",
      "method": "GenerateFreeFormStreamed",
      "project_id": "simplifymycloud-dev",
      "service": "google.cloud.notebooklm.v1main.NotebookService",
      "version": "v1"
    },
    "type": "consumed_api"
  },
  "severity": "INFO",
  "timestamp": "2026-07-31T18:30:00.149015368Z"
}
```

**Confirmed fields:**
- `userQuery` — full prompt, verbatim
- `serviceTextReply` — full response, verbatim, correctly grounded in the uploaded source with citation markers
- `userIamPrincipal: chris@simplifymy.cloud` — real user attribution
- `logMetadata.name` — ties the entry to the specific notebook resource (`notebooks/e437af8f-...`)
- `logMetadata.serviceLabel: NOTEBOOKLM_ENTERPRISE`, `methodName: GenerateFreeFormStreamed` — clean, unambiguous method identification

**Data-quality note (not a security finding):** the `serviceTextReply` field in the raw entry contains the same paragraph **repeated three to four times with overlapping/growing boundaries** — consistent with the logging pipeline capturing each cumulative streaming chunk of `GenerateFreeFormStreamed` rather than deduplicating down to the final complete response. This doesn't affect the audit value of the entry (the actual content is fully present and correct) but is worth flagging to whoever builds downstream parsers against this log stream — naive string concatenation or duplicate-detection logic may be needed.

**Architecture note:** `resource.type` here is `"consumed_api"`, not `"audited_resource"` as seen in the true Cloud Audit Log entries from the Code Assist and BigQuery rounds. This suggests NotebookLM Enterprise's usage logging is a related but distinct logging mechanism from standard Cloud Audit Logs, rather than the same `cloudaudit.googleapis.com` pipeline. This was not pursued further this round per the customer's explicit scoping (see [Findings](03-findings.md)) — they route raw logs to GCS/Dataflow/LogScale and BigQuery regardless of the underlying log type, so the practical impact is minimal for their use case, but it's worth documenting for technical accuracy.

## Scenario D — Console check: no observability control surface exists

Checked the Gemini Notebook Enterprise console/admin interface for `simplifymycloud-dev`:

```
https://console.cloud.google.com/gemini/notebooklm-enterprise?project=simplifymycloud-dev
```

**Result:** the interface contains no observability, logging, or audit-related settings of any kind — the only user-configurable option present is a dark mode toggle.

**This is a distinct pattern from both other products tested in this series:**
- Code Assist Standard: console control **exists but is broken** (confirmed root-caused bug, see the Code Assist assessment)
- Gemini in BigQuery: console control **doesn't exist**, and the API explicitly **rejects** any attempt to configure it
- Gemini Notebook Enterprise: console control **doesn't exist**, but the API **works correctly** — there's simply nothing to misconfigure via UI, by design rather than by failure

## Scenarios explicitly not run this round (documented, not overlooked)

Per direct customer instruction mid-assessment, the following were deliberately scoped out:

- **Metadata-only logging mode** (`sensitiveLoggingEnabled: false`, `observabilityEnabled: true`) — not relevant; customer's requirement is binary (enable maximum logging or don't), not "how does partial logging behave"
- **Fully-disabled mode** — same reasoning
- **IAM viewer-role access control** on who can see these logs via Logs Explorer — not relevant; the customer routes all raw logs to GCS buckets (consumed by Dataflow into LogScale) and to BigQuery for a data lake, independent of any Cloud Logging console viewer permissions

---

← [Back: 01 — Methodology](01-methodology.md) | [README](../../README.md) | Next: [03 — Findings](03-findings.md) →
