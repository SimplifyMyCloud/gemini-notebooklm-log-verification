← [Back to README](../../README.md)

# NotebookLM Plus (Google Workspace) — Testing Methodology

**Status: not yet started.** This document is a placeholder establishing scope and context ahead of testing.

## Why this round exists

The customer is currently under the impression that audit/security logging for the Gemini Notebook product family is available **only** in Enterprise mode (the GCP-project-based product tested in `../../enterprise/`), and this belief is a source of frustration. This round tests that belief directly against **NotebookLM Plus**, the tier bundled into Google Workspace Business Standard and above.

## Known context going in

- A dated (March 2026) public forum post from an admin in a PHI environment reported that NotebookLM events could not be found through **any** avenue tried: BigQuery export of Workspace audit logs, the Google Admin console reports, or GCP Logs Explorer. Google Support's own reply, quoted in that thread, confirmed the logs could not be found through those tools. This is third-party evidence, not yet independently confirmed by this assessment — that confirmation is exactly what this round is for.
- Architecturally, NotebookLM Plus is a Google Workspace core service, not a GCP-project resource — it is **not** expected to use `discoveryengine.googleapis.com` or any other GCP-project-scoped API the way Gemini Notebook Enterprise does. Verification here will need to go through the **Google Workspace Admin console** (`admin.google.com`), not `gcloud`.

## Planned test approach

1. Check the Workspace Admin console's **Reporting → Audit and investigation** tool for any NotebookLM-related log source
2. Check the standard **Reporting → Reports** section for the same
3. If a real usage event can be generated and searched for (same marker-based methodology as the Enterprise and BigQuery rounds), attempt to do so
4. Document findings with the same atomic-level command/screenshot evidence standard as the rest of this assessment

## Access confirmed

Workspace admin console access on `simplifymy.cloud` (Business Standard or above) is available for this testing.

---

← [Back to README](../../README.md)
