← [Back: 02 — Scenario Log](02-scenario-log.md) | [README](../../README.md)

# Gemini Notebook Enterprise — Findings

## TL;DR

**Full logging — including complete prompt and response content — works correctly for Gemini Notebook Enterprise, enabled entirely via API, with no console dependency at all.** This is the strongest, cleanest logging result of the three GCP AI products tested in this series so far.

## Findings Summary

| # | Question | Answer | Evidence |
|---|---|---|---|
| 1 | Can full logging (prompts + responses + metadata) be enabled? | **Yes.** Confirmed via `ObservabilityConfig` API, both `observabilityEnabled` and `sensitiveLoggingEnabled` set `true` in one call | [Scenario A](02-scenario-log.md#scenario-a-enable-full-observability-logging-via-api) |
| 2 | Does real usage actually get captured, with real content? | **Yes.** Full verbatim prompt and full verbatim response captured, correctly attributed to the real user | [Scenario B](02-scenario-log.md#scenario-b-real-usage-test-bitcoin-whitepaper-notebook), [Scenario C](02-scenario-log.md#scenario-c-log-verification) |
| 3 | Is there a console path to enable/verify this? | **No console control surface exists at all** — not broken (unlike Code Assist), not rejected (unlike BigQuery) — simply absent from the UI. API is correctly the only path | [Scenario D](02-scenario-log.md#scenario-d-console-check-no-observability-control-surface-exists) |
| 4 | Is this the same logging mechanism as standard Cloud Audit Logs? | **Likely not** — `resource.type` is `consumed_api`, not `audited_resource` as seen for Code Assist/BigQuery's true Cloud Audit Log entries. Documented for technical accuracy; doesn't affect the customer's use case given their log-routing architecture | [Scenario C](02-scenario-log.md#scenario-c-log-verification) |
| 5 | Any data-quality concerns for downstream log consumption? | **Yes, minor:** the `serviceTextReply` field contains duplicated/overlapping streamed-chunk text rather than a single deduplicated final response. Worth handling in any downstream parser | [Scenario C](02-scenario-log.md#scenario-c-log-verification) |

## Comparison across all three products tested

| Product | Full logging available? | Enable path | Console reliability |
|---|---|---|---|
| Gemini Code Assist Standard | Yes | API (`gcloud gemini logging-settings`) | **Broken** — confirmed console bug, doesn't reflect true state in either direction |
| Gemini in BigQuery | **No** — no mechanism exists at all | N/A — API explicitly rejects it | N/A — no control exists to be broken |
| Gemini Notebook Enterprise | **Yes** | API (`ObservabilityConfig` REST call) | N/A — no control surface exists, by design; nothing to be broken |

## Bottom line for the customer

Gemini Notebook Enterprise directly answers the customer's core question: **yes, full prompt and response logging can be enabled**, and it works correctly and completely when tested with real technical content. This should resolve any concern in that direction for the Enterprise tier specifically.

**Still open:** the customer's separate, specific frustration was the *belief that this logging capability is Enterprise-only*. This Enterprise-tier confirmation doesn't resolve that belief either way — it only confirms Enterprise itself works. The next phase of this assessment (`../../workspace/`) tests **NotebookLM Plus via Google Workspace** directly, to confirm or refute whether the same logging capability (or some logging capability) also exists on the non-Enterprise tier.

