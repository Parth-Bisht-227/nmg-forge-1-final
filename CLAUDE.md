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
- Dashboard (localhost:7700) was not loading because run.py was not starting mcp/server.py
  as a subprocess first. The MCP server must boot before the detect loop runs.
- `Response Time` column uses floats — always use `_float()` not `_int()`.
- `Word Count` can be empty string in some rows — `_int()` handles None/empty safely.
