# Spec — Medical School Explorer Data-Verification Sweeper

Loop-engineering spec for keeping `medical-school-explorer`'s `uniData` (24 AU/NZ schools, 2026/2027 cycle) accurate, and burning down the 12-school unresearched backlog. Status: **spec only, not built**.

## 1. Problem

- `uniData` is the single source of truth for a live, deployed site — and 12 of 24 entries are `unresearched`/`unverified` (melbourne, sydney, curtin, griffith, bond, anu, usc, latrobe, csu, cdu, auckland, otago).
- Verified entries decay: unis revise deadlines, places, schemes on their own timetables, and the Sep–Dec 2026 application window makes `closes` dates the highest-stakes field.
- Verification today is manual, per-session, and unrecorded — no `lastVerified` anywhere in the schema, so there is no way to know what's stale.

## 2. Loop shape

```
select batch → fetch official sources (maker) → extract claims → propose uniData diff
    → independent re-check (checker) → drift/finding report → HUMAN GATE → apply + push (Kevin)
    → post-deploy verify (existing Vercel gotcha loop)
```

**Cadence**: weekly. Escalate nothing automatically — this is report-driven at every level (see §6).

**Batch size**: 4 schools per run. Selection priority, in order:
1. Backlog (`confidence: "unverified"` / `gateType: "unresearched"`)
2. Deadline proximity — any school whose `closes` date is within 60 days gets checked every run regardless of confidence
3. Staleness — verified entries not re-checked in 30+ days

At 4/run, the 12-school backlog clears in 3 weeks; steady-state is drift-watch.

## 3. State (new files, live in the explorer repo)

```
sweeper/
  state.json          # per-school ledger — the loop's memory
  reports/            # one report per run, YYYY-MM-DD.md
  proposed/           # diff proposals awaiting the human gate
```

`state.json` per-school record:

```json
{
  "id": "curtin",
  "lastChecked": "2026-07-30",
  "sourcesChecked": ["<officialUrl>", "<tisc/uac page>"],
  "pageHash": "sha256 of extracted admissions section",
  "result": "verified | changed | unreachable | ambiguous",
  "openFindings": ["closes date on page conflicts with uniData"]
}
```

`pageHash` is the cheap drift detector: hash the *extracted admissions facts* (not raw HTML — page chrome churns). Unchanged hash on a verified school = skip deep re-verification, log the check, move on. This is what keeps weekly runs cheap.

**Deliberately NOT adding `lastVerified` to `uniData` itself** — keeps app schema untouched, avoids widening the byte-identical `index.html`/`medicine_explorer.html` sync surface. The ledger is sidecar state.

## 4. Maker / checker

**Maker** (main loop agent):
- Fetch each school's `officialUrl` + the relevant TAC page (UAC/VTAC/QTAC/TISC/SATAC) via **WebFetch first** (free). Firecrawl fallback only for JS-rendered pages — costs ~1 credit/scrape, log every fallback use in the report.
- Extract the watched fields: `minAtar(Text)`, `closes(Text)`, `ucat`, `interview(Text)`, `places`, `cost`, `schemes`, `weighting`, `prereqs`, `timeline` dates, plus `officialUrl` liveness (link-rot check).
- Emit a proposed diff per school into `sweeper/proposed/<id>-YYYY-MM-DD.md` — human-readable field-by-field: current value → proposed value → source URL → quote supporting it.

**Checker** (verifier subagent, fresh context, no access to maker's reasoning):
- Re-fetch each cited source; confirm each proposed change is literally supported by it.
- Enforce the repo's hard data rules (these are the checker's rubric, all from the explorer's CLAUDE.md):
  - `confidence: "verified"` only if a **live official source** was checked this run — TAC pages count, prep-site/forum claims never do.
  - `weighting` stays `null` unless the university publicly discloses percentages — never inferred.
  - `gateType` is never guessed — a school stays `"unresearched"` until the mechanism is actually documented.
  - Any UCAT number must be on the **0–2700 scale** (2025 rescale) — a 3600-scale figure in a source must be flagged, not converted silently.
- Checker verdict per field: CONFIRMED / UNSUPPORTED / SOURCE-AMBIGUOUS. Only CONFIRMED fields survive into the final report.

## 5. Human gate (mandatory, never removed)

Repo rule is absolute: **never push without Kevin's explicit say-so in that turn**. The gate:

1. Kevin reads `sweeper/reports/<date>.md` (summary: N checked, N drifted, N proposals, N blocked).
2. Approves proposals per school.
3. Apply step edits **both** `index.html` and `medicine_explorer.html` (byte-identical rule), runs `node --check` on the extracted script block, local `python -m http.server` + browser smoke.
4. Kevin commits and pushes himself (or says "commit and push" explicitly).
5. Post-push: existing deploy-verification ritual — incognito hard-refresh on the Vercel URL, confirm a changed field renders (not curl).

## 6. Rollout — L1 → L2, no L3

- **L1 (weeks 1–3, backlog burn-down)**: report-only. Sweeper writes reports + proposals; touches nothing in the app. Run manually: open the explorer project and say "run the sweeper" (or via `/loop` self-paced within one session).
- **L2 (steady state)**: after approval in-session, sweeper applies the approved diff to both HTML files locally (uncommitted), runs the syntax/smoke checks, and stops. Kevin reviews `git diff` and pushes. Optionally move the L1 fetch/report step to a **scheduled cloud agent** (weekly routine) at this point.
- **L3 (auto-apply/auto-push): explicitly out of scope, permanently.** The no-unprompted-push rule and the "verified means a human-accountable source check" bar make full autonomy wrong for this data — a wrong `closes` date on a live admissions site harms real applicants.

## 7. Circuit breakers

- **Fetch failures ≥ 3 in one run** → stop, report "sources unreachable", don't mark anything unverified/stale on the basis of a network problem.
- **Diff touches > 4 fields on a previously-verified school** → suspect page redesign or extraction error, not real drift. Flag for manual review, propose nothing.
- **Firecrawl fallback > 4 scrapes in one run** → stop and report; credit burn signal.
- **Contradiction between official uni page and TAC page** → SOURCE-AMBIGUOUS, human decides; the sweeper never picks a winner.

## 8. Cost

- **L1 manual run**: WebFetch is free; ~8–10 page fetches + extraction + checker pass is an ordinary Claude Code session on the existing plan — no extra API spend. Firecrawl fallback ~1 credit/scrape, expected rarely, always logged.
- **Scheduled cloud agent (L2 option)**: draws from usage credits per run. I'm not sure of the exact per-run credit cost of a scheduled routine — check the first run's usage before committing to weekly.

## 9. Cross-links

- Data corrections found here must be cross-checked against `C:\Users\sccm\claudeCode\UCAT\ucat-medical-interview-notes.md` where they overlap (per that project's CLAUDE.md), and vice versa — the report gets a "UCAT notes impact" line when applicable.
- Vault atoms already governing this data: `medical-school-explorer-mixed-confidence-uni-schema`, `ucat-anz-2025-rescale-3600-to-2700`.

## 10. Build checklist (when approved)

1. `sweeper/` scaffold + `state.json` seeded from current `uniData` confidence values.
2. Sweeper skill/prompt file in the explorer repo (`.claude/` or CLAUDE.md section) encoding §2–§7 so "run the sweeper" works from a cold session.
3. First L1 run on the 4 highest-priority backlog schools (suggest: curtin, griffith, bond, anu — all AU, all likely straightforward official pages).
4. Review report format with Kevin; adjust; then weekly.
