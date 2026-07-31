# Gemini Notebook — Audit & Security Logging Assessment
### Enterprise vs. Workspace (NotebookLM Plus)

Part of the same testing series as the Gemini Code Assist Standard and Gemini in BigQuery assessments — same methodology: verify what the documentation claims, then test it directly, API-first, with atomic-level command+output evidence.

**Project tested (Enterprise track):** `simplifymycloud-dev`
**Workspace tested (Workspace track):** `simplifymy.cloud`
**Tester:** Chris (`chris@simplifymy.cloud`)

---

## Why this assessment exists

The customer's core question: **can full audit/security logging be enabled for the Gemini Notebook product family, and is that capability actually gated behind the Enterprise tier** (as the customer currently believes, unhappily)?

This repo is split into two independent tracks to answer that directly.

## Status

| Track | Status | Headline finding |
|---|---|---|
| [`enterprise/`](enterprise/docs/01-methodology.md) | ✅ Complete | **Full prompt/response logging works correctly**, enabled entirely via API (`ObservabilityConfig`), no console dependency. Strongest result of any product tested in this series. |
| [`workspace/`](workspace/docs/01-methodology.md) | 🔲 Not yet started | Will directly test whether NotebookLM Plus (bundled in Workspace Business Standard+) has any logging capability, to confirm or refute the customer's "Enterprise-only" belief. |

## Contents

### Enterprise track (`enterprise/docs/`)
1. [Methodology](enterprise/docs/01-methodology.md) — objective, scope, licensing note (15-license minimum, tested under a 15-day/5,000-license trial), environment setup
2. [Scenario Log](enterprise/docs/02-scenario-log.md) — atomic evidence: enabling full logging via API, a real Bitcoin-whitepaper-grounded usage test, log verification, and the console check
3. [Findings](enterprise/docs/03-findings.md) — summary table, cross-product comparison, bottom line

### Workspace track (`workspace/docs/`)
1. [Methodology](workspace/docs/01-methodology.md) — placeholder, scope and known context ahead of testing

## Key finding so far

Gemini Notebook Enterprise's logging is the cleanest result of any product tested in this series: unlike Code Assist Standard (working API, broken console) or Gemini in BigQuery (no capability at all, API explicitly rejects it), Gemini Notebook Enterprise's logging **just works** via a documented API call, with no console surface to be broken in the first place — full verbatim prompt and response content confirmed captured with real user attribution.

What's still open is the customer's actual frustration: whether that capability *requires* Enterprise, or whether NotebookLM Plus via Workspace has something comparable. That's the next phase.
