← [Back to README](../../README.md) | Next: [02 — Scenario Log](02-scenario-log.md) →

# Gemini Notebook Enterprise — Testing Methodology

## Testing Objective

Confirm, with command-level evidence, whether full audit/security logging (prompts, responses, metadata) can be enabled for Gemini Notebook Enterprise (formerly NotebookLM Enterprise), and by what mechanism.

**Customer framing for this round:** the customer's requirement is binary — *can full logging be enabled at all?* If yes, they will enable everything available. This round does **not** test partial/metadata-only logging modes, and does **not** test fine-grained access control on who can view the logs, since the customer routes all raw logs to GCS buckets (consumed by Dataflow → LogScale) and BigQuery for a data lake regardless of console-level viewer permissions. Those two angles were deliberately scoped out — see [Findings](03-findings.md) for the explicit scoping notes.

**Customer context that motivated this round:** the customer was under the impression that logging for the Gemini Notebook product family is available **only** in Enterprise mode, and this made them unhappy. This assessment tests Enterprise directly (this document) and will separately test **NotebookLM Plus via Google Workspace** (see `../../workspace/`) to confirm or refute that belief.

## Scope

**In scope:** Gemini Notebook Enterprise (formerly NotebookLM Enterprise), a GCP-project-based product on the Discovery Engine backend (`discoveryengine.googleapis.com`).

**Explicitly not tested this round:** metadata-only logging mode, fully-disabled mode, IAM viewer-role access control on the logs, and the Admin/Owner/User notebook-sharing role distinctions — none of these matter to the customer's stated requirement.

**Project tested:** `simplifymycloud-dev`
**Test date:** 2026-07-31
**Tester:** Chris (`chris@simplifymy.cloud`, `roles/owner`)

## Licensing Note

Gemini Notebook Enterprise has a **15-license minimum** purchase requirement — there is no way to license a single seat for a quick test. Testing here was done under a **15-day / 5,000-license free trial**, activated directly from the console:

```
https://console.cloud.google.com/gemini/notebooklm-enterprise?project=simplifymycloud-dev
```

Worth flagging to the customer: the 15-license minimum is a real commercial constraint if they intend to pilot this with a small group before a wider rollout.

## Environment Setup (API-only, no Terraform, no console-driven config)

Consistent with the Code Assist and BigQuery rounds, all configuration in this assessment was done via `gcloud`/REST API — the console was checked only to confirm whether a working alternative path exists (it doesn't — see [Scenario D](02-scenario-log.md#scenario-d-console-check-no-observability-control-surface-exists)).

```bash
PROJECT_ID="simplifymycloud-dev"

# 1. Enable the Discovery Engine API (Gemini Notebook Enterprise runs on this,
#    not cloudaicompanion — a different backend than Code Assist or BigQuery)
gcloud services enable discoveryengine.googleapis.com --project="${PROJECT_ID}"

# Verify
gcloud services list --enabled --project="${PROJECT_ID}" \
  --filter="config.name:discoveryengine.googleapis.com" \
  --format="table(config.name)"
```

**IAM note:** Google's own documentation is internally inconsistent about the exact role needed to enable observability/audit logging for this product — one page cites `roles/discoveryengine.admin`, another cites `roles/discoveryengine.agentspaceAdmin` for the same "NotebookLM Enterprise Admin" capability described in the console UI as "Cloud NotebookLM Admin." This was **not resolved further**, because the test principal already held `roles/owner` on the project, which covers both regardless of which is correct. If replicating this on a project where the tester is *not* Owner, resolve this ambiguity first — don't assume either role name without confirming.

---

← [Back to README](../../README.md) | Next: [02 — Scenario Log](02-scenario-log.md) →
