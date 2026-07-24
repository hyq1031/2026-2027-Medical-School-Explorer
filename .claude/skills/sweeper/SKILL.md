---
name: sweeper
description: Triggered by "run the sweeper" (or /sweeper) in the medical-school-explorer project. Report-only (L1) data-verification loop that re-checks uniData school records (24 AU/NZ medical schools) against live official + TAC admissions pages via an independent maker/checker split, and writes findings + proposed diffs for Kevin to review — never edits index.html/medicine_explorer.html or touches git on its own. See EXPLORER-SWEEPER-SPEC.md for full design rationale.
---

# sweeper

Keeps `uniData` (24 AU/NZ medical schools, 2026/2027 cycle) accurate by periodically
re-checking each school's admissions facts against its official page and the relevant
Tertiary Admission Centre (TAC) page, and burns down the 12-school unresearched backlog.
This is L1 (report-only): it fetches, verifies, and reports — it does not edit the app or
push. Full design: `EXPLORER-SWEEPER-SPEC.md` in this repo.

## When to use

- Kevin says "run the sweeper" (or `/sweeper`) from within this project.
- Working the unresearched backlog down, or it's been a while since verified schools were
  last re-checked.

## Batch selection (4 schools per run)

Read `uniData` live from `index.html` for current values — `sweeper/state.json`'s
`seedConfidence`/`seedGateType` are a one-time snapshot for reference only, never authoritative.

1. Backlog first — `confidence: "unverified"` or `gateType: "unresearched"`.
2. Any school whose `closes` date is within 60 days — check every run regardless of confidence.
3. Staleness — verified entries with `lastChecked` 30+ days ago in `sweeper/state.json`.

## Step 1 — Maker pass

For each selected school:

- Fetch `uni.officialUrl` + the relevant TAC page (NSW/ACT → UAC, VIC → VTAC, QLD → QTAC,
  WA → TISC, SA → SATAC) via **WebFetch first** (free). Only fall back to Firecrawl for
  JS-rendered pages WebFetch can't extract usably from — log every fallback use, it costs
  ~1 credit/scrape.
- Extract the watched fields: `minAtar(Text)`, `closes(Text)`, `ucat`, `interview(Text)`,
  `places`, `cost`, `schemes`, `weighting`, `prereqs`, `timeline` dates, and confirm
  `officialUrl` still resolves (link-rot check).
- Write `sweeper/proposed/<id>-YYYY-MM-DD.md`: one row per changed/newly-found field —
  current value → proposed value → source URL → the exact quote supporting it.

## Step 2 — Checker pass (independent, fresh subagent)

Spawn a fresh subagent with **no access to the maker's reasoning** — only the proposal file
and the source URLs. It must:

- Re-fetch every cited source itself and confirm each proposed value is literally supported.
- Enforce these rules (from this project's `CLAUDE.md`), every time:
  - `confidence: "verified"` only if a **live official or TAC source** was checked this run
    — prep-site/forum claims never qualify.
  - `weighting` stays `null` unless the university publicly discloses percentages — never
    inferred.
  - `gateType` is never guessed — a school stays `"unresearched"` until the mechanism is
    actually documented.
  - Any UCAT figure must be on the **0–2700 scale** (2025 rescale) — a 3600-scale number
    found in a source gets flagged, not silently converted.
- Verdict per field: `CONFIRMED` / `UNSUPPORTED` / `SOURCE-AMBIGUOUS`. Only `CONFIRMED`
  fields survive into the report.

## Step 3 — Report and update the ledger

- Write `sweeper/reports/YYYY-MM-DD.md`: N checked, N drifted, N proposals, N blocked,
  Firecrawl fallback count.
- Update `sweeper/state.json` for every school checked this run: `lastChecked`,
  `sourcesChecked`, `pageHash` (hash of the *extracted admissions facts*, not raw HTML —
  page chrome churns; an unchanged hash on a verified school means skip deep re-verification
  next time), `result` (`verified | changed | unreachable | ambiguous`), `openFindings`.
- If any finding overlaps `C:\Users\sccm\claudeCode\UCAT\ucat-medical-interview-notes.md`,
  note it in the report as a "UCAT notes impact" line (per that project's CLAUDE.md).

## Step 4 — Stop and hand off to Kevin

Default (L1): **stop here**. Never edit `index.html`/`medicine_explorer.html`, never touch
git. Present the report summary and wait.

If Kevin approves specific proposals in-session (L2):

1. Apply the approved diff to **both** `index.html` and `medicine_explorer.html`
   (byte-identical rule — edit one, `cp` the other).
2. Run `node --check` on the extracted `<script>` block.
3. Local `python -m http.server` + browser smoke check (visual + console errors).
4. Stop. Kevin reviews `git diff` and commits/pushes himself, or says "commit and push"
   explicitly that turn — never push on an earlier approval carrying forward.

## Circuit breakers

- **≥3 fetch failures in one run** → stop, report "sources unreachable"; don't mark anything
  stale/unverified on the basis of a network problem.
- **Diff touches >4 fields on a previously-verified school** → suspect page redesign or
  extraction error, not real drift. Flag for manual review, propose nothing for that school.
- **Firecrawl fallback >4 scrapes in one run** → stop and report (credit burn signal).
- **Contradiction between the official uni page and the TAC page** → `SOURCE-AMBIGUOUS`,
  human decides. The sweeper never picks a winner.

## Guardrails

- L3 (auto-apply/auto-push) is permanently out of scope — never edit the live HTML files or
  push without Kevin's explicit say-so in that turn, every time, no exceptions for a prior
  approval.
- Never guess `gateType` or `weighting`. `"unresearched"`/`null` is always safer than a
  plausible-sounding guess — see vault atom `medical-school-explorer-mixed-confidence-uni-schema`.
- Full design rationale, `state.json` schema detail, and cost notes:
  `EXPLORER-SWEEPER-SPEC.md` in this repo.
