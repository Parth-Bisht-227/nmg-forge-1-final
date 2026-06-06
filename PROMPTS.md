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

### 1. Implement all 10 missing detectors
- **Prompt:** "Open seo/detector.py and implement the 10 missing detectors. Use only existing
  helpers is_html(), is_200(), indexable(), _int(), _float(). Follow the exact add() call
  pattern. Filters: HTML-only for title/meta/H1, indexable+200 for missing/duplicate checks,
  any 200 html for missing_h1 and slow_page. Detectors: title_too_short (len<30), 
  missing_meta_description (empty meta, indexable 200), duplicate_meta_description (same meta
  on 2+ indexable URLs), meta_description_too_long (len>155), missing_h1 (any 200 html),
  duplicate_h1 (same H1 on 2+ indexable), redirect_chain (3xx whose Redirect URL is also a
  3xx Address — build a set first), thin_content (word count<200, indexable), 
  non_indexable_but_linked (not indexable AND inlinks>0), slow_page (response time>1.0).
  After implementing, run python run.py ../sample-export/ --no-dashboard and show a count
  table for all 17 detectors."
- **For:** Completing the detection layer — worth 20 pts on the leaderboard
- **Revised?** No — one-shot. Column names matched internal_all.csv exactly. Redirect chain
  required building a set of 3xx addresses before checking targets, which the prompt specified.

