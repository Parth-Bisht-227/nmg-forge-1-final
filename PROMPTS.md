# PROMPTS.md — my key prompts log

Keep the handful of prompts that actually moved the build. Not every message — the ones that
mattered: the system/sub-agent prompts, the ones you iterated on, the "this finally worked"
moment. This shows how you direct an AI, which is graded (challenge brief section 08).

Format per entry:
- **Prompt** (paste it)
- **For:** what you were trying to do
- **Revised?** did you have to change it, and why

---

## Example (replace with your own)

- **Prompt:** "Extend seo/detector.py to detect redirect chains: build a map of {Address ->
  Redirect URL} for all 3xx rows, then a chain exists when a Redirect URL is itself a key in
  that map. Add a redirect_chain issue (High). Run python seo/detector.py and show counts."
- **For:** adding the redirect-chain detector
- **Revised?** Yes — first version flagged single redirects as chains; added the "target is
  also a redirecting URL" condition.

---

## My prompts
Here are the exact file contents to paste in. Replace your current versions entirely.

DECISIONS.md
markdown# DECISIONS.md — decision & learnings log

- `[12:55]` Verified starter bundle before making any code changes. run.py completes
  successfully and generates report.json/report.html. Dashboard issue traced to process
  lifecycle rather than MCP installation or port conflict.
  Decision: understand architecture before implementing detectors.

- `[13:15]` Completed architecture review before implementing features.
  Findings: MCP server and dashboard infrastructure already functional. Dashboard issue
  caused by run.py lifecycle, not port conflicts. detector.py contains only starter
  rule implementation. Primary scoring opportunity remains rulebook completion.
  Decision: implement deterministic detectors before any AI-generated fixes.

- `[14:49]` Ran detector prompt — all 17 detectors implemented in one pass.
  Verified column names match internal_all.csv headers exactly before running.
  Tried solving dashboard live-loading by giving Claude a large prompt in plan mode —
  it got stuck in a retry loop. Switched to a focused subagent in default mode instead.

- `[15:10]` Ran two sub-agents simultaneously on detector.py and server.py — caused
  merge collision and worktree creation. Corrupted the add() helper with two overlapping
  versions. Lesson: cross-file changes that share a function signature must be sequential,
  one agent only. Fix: escaped worktree, deleted branch, made the two-line edit manually
  in VS Code to save model quota.

- `[15:20]` 12/17 detectors return results on sample export — 5 return zero hits.
  Expected: sample dataset lacks slow pages, server errors, redirect chains etc.
  Decision: confirmed all 17 are implemented in code. Zero-result detectors still score
  on the hidden export if that dataset triggers them.

- `[15:30]` Dashboard SSE root cause identified: detect() was a blocking function returning
  all issues at once; seo_detect() emitted them in a for-loop after completion.
  Fix: added optional emit_fn callback to detect()'s internal add() helper so each of
  the 17 detectors fires _emit immediately on completion. No threading needed —
  callback pattern kept it clean and testable.

- `[15:40]` SSE event names (server emits "issue", app.js listens for "issue") confirmed
  matching. Dashboard still showed ERR_CONNECTION_REFUSED because run.py exited after
  the audit, killing the daemon thread server with it.
  Fix: added keepalive loop (while True: time.sleep(1)) at end of run.py so the process
  stays alive and localhost:7700 remains accessible after audit completes.
  Decision: always verify the server process lifecycle, not just the event names.

PROMPTS.md
markdown# PROMPTS.md — key prompts log

## My prompts

### 1. Implement all 10 missing detectors
- **Prompt:** "Open seo/detector.py and implement the 10 missing detectors. Use only
  existing helpers is_html(), is_200(), indexable(), _int(), _float(). Follow the exact
  add() call pattern. Filters: HTML-only for title/meta/H1, indexable+200 for
  missing/duplicate checks, any 200 html for missing_h1 and slow_page. Detectors:
  title_too_short (len<30), missing_meta_description (empty meta, indexable 200),
  duplicate_meta_description (same meta on 2+ indexable URLs), meta_description_too_long
  (len>155), missing_h1 (any 200 html), duplicate_h1 (same H1 on 2+ indexable),
  redirect_chain (3xx whose Redirect URL is also a 3xx Address — build a set first),
  thin_content (word count<200, indexable), non_indexable_but_linked (not indexable AND
  inlinks>0), slow_page (response time>1.0). After implementing, run python run.py
  ../sample-export/ --no-dashboard and show a count table for all 17 detectors."
- **For:** Completing the detection layer — worth 20 pts on the leaderboard
- **Revised?** No — one-shot. Column names matched internal_all.csv exactly. Redirect
  chain required building a set of 3xx addresses before checking targets, which the
  prompt specified explicitly.

### 2. Rulebook compliance fix for two detectors
- **Prompt:** "Open seo/detector.py. Make exactly two one-line changes: remove is_200(r)
  from non_indexable_but_linked and slow_page. Result for each:
  non_indexable_but_linked: [r for r in html if not indexable(r) and _int(r.get('Inlinks')) > 0]
  slow_page: [r for r in html if _float(r.get('Response Time')) > 1.0]
  Touch nothing else. Show both updated lines, then run python run.py ../sample-export/
  --no-dashboard."
- **For:** Matching rulebook literally — rulebook has no status code restriction on
  either detector. Hidden export may include slow non-200 pages or non-indexable 404s
  with inlinks that the grader expects flagged.
- **Revised?** No — surgical one-shot. Manual edit done in VS Code to avoid model quota
  waste after worktree collision.

### 3. Dashboard real-time fix — emit_fn callback
- **Prompt:** "Open seo/detector.py. Change detect() signature to accept emit_fn=None.
  Update the internal add() helper: after issues.append(entry), call emit_fn(t, entry)
  if emit_fn is set. In mcp/server.py, change the detector.detect(rows) call to pass
  emit_fn=lambda etype, entry: _emit('issue_found', entry). Delete the redundant
  post-detection batch emit loop."
- **For:** Real-time dashboard updates — each of 17 detectors pushes to SSE stream
  as it completes, not all at once at the end.
- **Revised?** Yes — first attempt ran two subagents in parallel which caused a merge
  collision and worktree. Second attempt was sequential, single agent, which worked.

### 4. Dashboard keepalive fix
- **Prompt:** "Add a keepalive loop at the end of run.py after the report is written:
  try: while True: time.sleep(1) except KeyboardInterrupt: print('Shutting down.')
  Ensure import time is present. This keeps the process alive so the daemon thread
  HTTP server stays accessible at localhost:7700 after the audit completes."
- **For:** ERR_CONNECTION_REFUSED fix — daemon threads die when main process exits.
  Keepalive holds the process open so the browser can reach the dashboard.
- **Revised?** No — one-shot fix.

