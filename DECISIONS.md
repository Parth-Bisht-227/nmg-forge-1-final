# DECISIONS.md — decision & learnings log

A short running note of the real choices you made: what you tried, what failed and why, what
you changed. This is your engineering judgement on the record — it is what separates a builder
from a button-presser, and it is graded (challenge brief section 08).

Append a 1–2 line entry whenever you make a real decision or hit/fix a wall. Add a timestamp.

Format:
`[HH:MM] <decision or problem> → <what you did and why>`

---

## Example (replace with your own)
- `[10:20]` Chose plain-csv parsing over pandas → fewer deps, fast enough for 5k rows, model
  quota saved for the fixer.
- `[11:05]` Title detector over-counted duplicates → realized non-indexable pages were
  included; added an indexable+200 filter (per rulebook).
- `[12:40]` Dashboard wasn't updating live → MCP tool wasn't emitting the SSE event; added
  `_emit("issue", row)` in extract.

---

## My log
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