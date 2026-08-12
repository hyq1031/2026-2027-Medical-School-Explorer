# Application Timeline Re-verification — 2026-08-12

**Scope:** Follow-up full-coverage pass over the same 24 schools covered by the
[2026-07-27 timeline audit](2026-07-27-timeline-audit.md) — Kevin asked to check the
sources again for accuracy. 4 research clusters run in parallel (NSW/UAC+ANU, VIC/GEMSAS,
QLD/QTAC+WA/TISC, SA/SATAC+Tasmania+NZ), each re-hitting live official sources.

Unlike the 2026-07-27 pass, this run also **applied all findings directly to
`index.html`/`medicine_explorer.html`** (per Kevin's "check again... update... commit and
push" instruction) rather than staying report-only — see the Corrections table below for
exactly what changed. `sweeper/state.json` ledger updated accordingly.

## Corrections applied

| School | Change | Source |
|---|---|---|
| **melbourne** | Added missing `opens`: **1 May 2026** (previously "TBC") | gemsas.edu.au/key-dates (direct GEMSAS quote, not third-party) |
| **anu** | `opens`/`closes` corrected to the direct-application route with an exact quote: **opens 11 Mar 2026, closes 8 May 2026**. Old "30 Nov 2026 (UAC)" figure flagged in `closesText` as likely non-functional (BHlth may be excluded from the Nov UAC reopening — WebSearch-sourced, not fully page-confirmed) | study.anu.edu.au/node/1245 |
| **bond** | Added missing `opens`: **5 January 2027, 8:30am QLD** (previously "TBC") | bond.edu.au own important-dates page |
| **tasmania** | `officialUrl` fixed — the "H7X" course code guessed as the fix last session was **also wrong**; correct current code is **H3X**. Dates unchanged, re-confirmed exact | utas.edu.au H3X course page |
| **cdu** | `officialUrl` fixed (old `wclsc1` URL 404'd) → working SMED01 course page. SMED01 vs WCSCI1 split re-confirmed on SATAC's own page, validating the 2026-07-27 call | satac.edu.au/key-dates, cdu.edu.au |
| **usc** | `name` updated to **"University of the Sunshine Coast (UniSC)"** — rebrand confirmed final and stable | unisc.edu.au |

## Re-confirmed unchanged (dates/URLs verified exact against a live source again)

unsw, newcastle-une (refined: JMP direct route separately opens in August), wsu, monash,
uq, adelaide (opens upgraded from estimated→exact), uwa, curtin, griffith, jcu, flinders
(double-confirmed via SATAC directly), auckland, otago (13 Aug 2026 confirmed correct and
current — **this deadline is tomorrow relative to today, 2026-08-12**, flagged for
Kevin's awareness, not a data error), cqu (dates solid; MMI sub-date still unconfirmed by
any live source).

## Investigated and cleared (no change needed)

- **sydney**: confirmed the app record correctly tracks USYD's Double Degree Medicine
  Program (small, ~30 places, UAC preference dates), not USYD's much larger separate
  graduate-entry GAMSAT MD (opens 21 Apr / closes 4 Jun 2026, direct application) — the
  existing `ucat`/`interview`/`program` fields already match DDMP specifically, so no
  schema mismatch exists here despite both programs surfacing in research.

## Still unresolved (unchanged from 2026-07-27, blocked again this session)

- **latrobe**: `latrobe.edu.au` is now hard-blocked to all fetch tooling used this
  session (WebFetch fails outright, proxy returns nav-shell only, WebSearch site: rejected)
  — worse than last time, when Firecrawl got through. The 6 Nov 2026 close date is carried
  forward unconfirmed. Needs a live browser check (Chrome extension/Playwright), not another
  automated-fetch attempt.
- **csu**: `study.csu.edu.au` still returns HTTP 403 on every path. Open date (~1 Jun 2026)
  remains third-party-sourced only.
- **notre-dame**: Confirmed as a genuine finding (not resolved) — Notre Dame's *separate*
  Graduate Entry MD is now directly confirmed GEMSAS-administered on both campuses
  (gemsas.edu.au has live pages for both), but that's not the pathway this record tracks.
  For the Assured Pathway itself: Fremantle likely defaults to TISC's standard final close
  (13 Jan 2027) since TISC's calendar has no Medicine-specific early date for Notre Dame
  (unlike UWA/Curtin); Sydney's close remains completely unresolved — notredame.edu.au
  403's to automated fetch every session so far.

## Recommended next step

The three items above (latrobe, csu, notre-dame) all share the same blocker — the target
site 403's/blocks automated fetch tools rather than the info being genuinely unpublished.
A manual browser pass (Chrome extension with a real session) is the likely unlock, worth
scheduling as a small follow-up rather than repeating the same automated WebFetch/WebSearch
approach a third time.
