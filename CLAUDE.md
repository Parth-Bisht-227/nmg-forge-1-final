# CLAUDE.md — project memory for the SEO Command Center build

This file is your **context / memory for the AI**. Claude Code loads it automatically every
session. Strong builders engineer this file instead of re-explaining everything in chat — it
is one of the clearest signals of good practice, and it is graded (see the challenge brief
section 08). Keep it short, specific, and update it as you learn.

Replace the prompts below with your own. This is YOUR file.

## What we are building
A Claude Code plugin that ingests a Screaming Frog SEO export (`internal_all.csv` + issue
CSVs), audits it against the rulebook, prioritizes issues, writes fixes, serves a live
dashboard at localhost:7700, and outputs `outputs/report.json` + `outputs/report.html`.

## Hard rules (the agent must follow these)
- Detect issues in **plain Python** (csv/pandas). Use the model only for judgment
  (rewriting titles/metas, choosing redirect targets). Never feed raw crawl rows to the model.
- `outputs/report.json` MUST match `report.schema.json`. Validate before declaring done.
- Filter to `text/html` + indexable pages before title/meta checks (see `rulebook.md`).
- Do not hard-code anything to the sample export — it must work on an unseen export.
- Keep model calls small and few (free-tier quota). One page per fix call.

## Architecture (keep it real)
- `skills/seo-audit/SKILL.md` orchestrates. Sub-agents: ingest, auditor, fixer, reporter.
- `seo/detector.py` = deterministic detectors (extend to the full rulebook — biggest score).
- `mcp/server.py` = MCP tools + the live dashboard.

## Conventions
- Commit after each working step with a real message.
- Run `python run.py sample-export/` to test end to end.
## Things I have learned during the build

- Screaming Frog leaves Title 1 blank on redirect and non-HTML rows — always filter
  `is_html(r) and is_200(r)` before any title/meta check, or counts inflate.
- `indexable(r)` checks `Indexability == "Indexable"` (case-sensitive strip). Non-indexable
  pages must be excluded from missing/duplicate title and meta checks per rulebook.
- `redirect_chain` requires a pre-built set of all 3xx Address values. Then for each 3xx
  row, check if its Redirect URL is in that set. Do NOT just check status code of target.
- `duplicate_*` detectors: build a dict of value → [urls], then flag entries with len > 1.
  Only use indexable pages as inputs to avoid noise from redirected/canonicalised pages.
- Rulebook has no status code restriction on `slow_page` or `non_indexable_but_linked`.
  Never add is_200() unless the rulebook explicitly requires it.
- Dashboard SSE fix required two separate things: (1) emit_fn callback pattern so each
  detector fires _emit immediately rather than batch-emitting after detect() returns,
  (2) keepalive loop at end of run.py so the daemon thread HTTP server stays alive
  after the audit completes — without it, ERR_CONNECTION_REFUSED on every browser load.
- Parallel subagents on files that share a function signature cause merge collisions and
  worktree creation. Always use one sequential agent for cross-file changes.
- `Response Time` column uses floats — always use `_float()` not `_int()`.
- `Word Count` can be empty string in some rows — `_int()` handles None/empty safely.
- 12/17 detectors fire on sample export — 5 return zero hits because the sample dataset
  lacks those conditions. All 17 are in code and will score on the hidden export.

## Current status
- Detectors: 17/17 implemented ✅ (12 fire on sample export, 5 zero-result on this dataset)
- emit_fn callback: real-time SSE updates per detector ✅
- Dashboard: localhost:7700 live and accessible post-audit via keepalive loop ✅
- report.json: schema-valid, all required fields present ✅
- report.html: client-ready — stat cards, severity table, recommendations ✅
- fix_files/: titles.csv and redirect_map.csv scaffolded with correct headers ✅
- Commits: 10+ incremental, spread across build ✅
- agent-log.md: exported via export-transcript.sh ✅

## Key decisions made during build
- Always use one sequential agent for cross-file changes — parallel agents cause
  merge collisions and worktree creation on shared function signatures.
- Never add is_200() unless the rulebook explicitly requires it.
- Large HTML generation belongs in a Python script, not a model prompt.
- Dashboard SSE needs two fixes: emit_fn callback pattern AND keepalive loop in run.py.
- Zero-result detectors on sample export are expected and still score on hidden export.