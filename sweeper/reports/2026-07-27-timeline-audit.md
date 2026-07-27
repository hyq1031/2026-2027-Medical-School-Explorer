# Application Timeline Audit — 2026-07-27

**Scope:** Special full-coverage pass (not the usual 4-school batch) — Kevin asked to confirm
application OPEN and CLOSE dates for all 24 schools and add sort-by-open/sort-by-close to the
UI. `uniData` currently has no `opens`/`opensText` field at all (only `closes`/`closesText`),
so every row below is a net-new field addition plus, where noted, a correction to the existing
`closes` value.

24 schools checked, split across 7 research clusters (UAC, VTAC, QTAC, TISC, SATAC, direct-AU,
NZ) via WebFetch-first / Firecrawl-fallback. 6 outright **data corrections** found (not just
fill-ins) and 2 **schema-level mismatches** (wrong portal modeled entirely). No proposals below
are pre-applied to `index.html`/`medicine_explorer.html` — this is the report-only step; nothing
gets written into the app until Kevin approves.

## Corrections (existing `closes` value was wrong or stale)

| School | Old `closes` | New `closes` | Note |
|---|---|---|---|
| **melbourne** | "28 September 2026" | **29 May 2026, 5pm AEST** | Wrong portal entirely — Melbourne MD is graduate-entry via **GEMSAS** (GAMSAT-based), not VTAC. 28 Sep was the VTAC undergrad date, doesn't apply here. North Western Pathway extension to 8 Jun. |
| **bond** | "TBC (Usually opens Jan, closes early Feb...)" | **19 January 2027** | For the May 2027 intake. Bond still routes through QTAC despite being private. Open date not found. |
| **sydney** | "TBC December 2026" | **13 Dec 2026** (assessment day 1) **/ 10 Jan 2027** (assessment day 2) | Refines vague TBC into the two actual UAC preference-deadline windows. |
| **flinders** | "TBC (SATAC deadlines apply)" | **1 December 2026** | Later than the Sep/Oct pattern most other SATAC schools follow — worth double-checking, but this is what SATAC's own key-dates page states verbatim. |
| **otago** | "10 December 2026" | Same date is real but **misleading** — 10 Dec is only the *Health Sciences First Year (HSFY)* enrolment deadline. The actual binding cutoff to be considered for **Medicine 2nd-year selection** is **13 August 2026** (opens 1 Jul 2026). Recommend recording both stages. |
| **tasmania** | course code M3N | **M3N page 404s** — UTAS has restructured to course code **H7X** ("Bachelor of Medical Science and Doctor of Medicine"). Dates (open 1 Aug 2026 / close 30 Sep 2026) are otherwise confirmed correct for the new code. |

## Schema mismatch (portal modeled wrong, not just a date)

| School | Currently modeled as | Actually is |
|---|---|---|
| **notre-dame** | "TBC (Usually late September via TISC/UAC or Direct)" | Both Sydney and Fremantle Notre Dame Medicine campuses are **GEMSAS graduate-entry**, not TISC/UAC/Direct at all. Same ~1 May–29 May 2026 GEMSAS window as Melbourne. (Third-party sourced, not yet cross-checked against gemsas.edu.au directly — treat as estimated pending that check.) A separate UAC-admitted "Assured Pathway" undergrad degree exists that guarantees eventual MD entry, but that's a different course, ordinary UAC dates. |
| **cdu** | closes "TBC October 2026 (SATAC)" | On-file URL (`wclsc1`) 404s. Two distinct SATAC course codes exist for CDU and it's ambiguous which one the app record is meant to track: **WCSCI1** "Bachelor of Clinical Sciences" (closes 1 Dec 2026) vs **SMED01** "Bachelor of Clinical Science Medicine/Doctor of Medicine" (closes 9 Oct 2026, matches the existing "October" hint more closely). Recommend fixing the URL and picking SMED01 given the closer date match, but Kevin should confirm which the app intends. |

## Confirmed (existing `closes` matched a live source) + new `opens`

| School | Opens | Closes | Confidence |
|---|---|---|---|
| unsw | ~8 Apr 2026 (general UAC, estimated) | **30 Sep 2026** (was "TBC" — now confirmed) | verified (close) |
| newcastle-une | ~8 Apr 2026 (general, estimated) | 30 Sep 2026 ✓ matches | verified |
| wsu | **8 Apr 2026** (WSU's own page) | 30 Sep 2026 ✓ matches | verified |
| monash | **3 Aug 2026, 9am** (VTAC) | 28 Sep 2026, 5pm ✓ matches | verified |
| uq | ~4 Aug 2026 (general QTAC) | **30 Sep 2026** (was "TBC" — now confirmed) | verified (close) |
| adelaide | ~3 Aug 2026 (general SATAC) | 30 Sep 2026 ✓ matches | verified |
| uwa | **2 Jun 2026** (TISC) | **30 Sep 2026** (was "TBC" — now confirmed) | verified |
| curtin | 2 Jun 2026 | 30 Sep 2026 ✓ matches | verified |
| griffith | ~4 Aug 2026 (general) | 30 Dec 2026 ✓ matches | verified |
| jcu | **4 Aug 2026** | **30 Sep 2026, 11:59pm** (was "TBC" — now confirmed; dual QTAC + JCU portal submission) | verified |
| anu | not re-verified this session | 30 Nov 2026 (structurally matches, not re-quoted) | estimated |
| usc (→ renamed **UniSC**) | ~4 Aug 2026 (general) | **30 Dec 2026** (was "TBC" — now confirmed) | verified |
| latrobe | **3 Aug 2026** | 6 Nov 2026, noon AEDT ✓ matches | verified |
| csu | ~1 Jun 2026 (third-party only, official page 403'd) | 25 Sep 2026 ✓ matches | close verified / open estimated |
| cqu | **4 Aug 2026** | 30 Sep 2026 ✓ matches (separate MMI 24–26 Nov 2026) | verified |
| auckland | not found (no MBChB-specific open date published) | 1 Jul 2026 ✓ matches (applies to both First-Year and Graduate Entry, one shared deadline) | close verified / open unresearched |

## Not yet independently re-confirmed this session
- **anu** direct-application deadline ("8 May 2026, closed") — only prior-year pattern found, not a fresh 2026/27 quote.

## Recommended next step
1. Kevin reviews corrections/schema-mismatch tables above (especially melbourne, notre-dame,
   bond, otago, cdu — these change real user-facing info, not just fill in blanks).
2. On approval, add `opens`/`opensText` fields to all 24 `uniData` entries in `index.html`
   (+ sync to `medicine_explorer.html`), applying the corrections above, and wire up
   "Sort by Application Opens" / "Sort by Application Closes" options in the existing
   `#sort-select` dropdown (see UI structure notes — `applyFilters()` sort block ~line 2846,
   new `parseAppDate()` helper needed near `formatDate()` ~line 2699 to turn free-text dates
   into a sortable value, with "TBC"/unresearched sorting last).
3. Local verify (`node --check`, `python -m http.server` + browser smoke check) before Kevin
   commits/pushes.
