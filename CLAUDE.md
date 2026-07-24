# Medical School Explorer — CLAUDE.md

Zero-dependency single-page HTML/CSS/JS app: AU/NZ undergrad medicine admissions explorer (24 universities, 2026/2027 cycle). No build step, no framework — all logic lives inline in `<script>`/`<style>` tags in the HTML file(s).

## Files
- `index.html` and `medicine_explorer.html` are **byte-identical duplicates** — Vercel serves `index.html` as the deployed root, but the original dev filename `medicine_explorer.html` is kept in sync. **Always edit one, then `cp` the other before committing.** Diverging them is the most likely self-inflicted bug in this repo.
- `med_explorer_banner.jpg` — hero background image, recolored via a `.hero::after` pseudo-element with `filter: sepia(0.85) saturate(1.15) brightness(0.55) contrast(1.08)` (not `hue-rotate` — an earlier attempt with `hue-rotate(-18deg)` pushed it green; pure sepia+saturate reads as warm engraving instead).
- `uniData` (top of the `<script>` block) is the single source of truth — a JS array of 24 university objects. All filtering/rendering derives from it.
- README.md describes an earlier build pass (old teal/purple palette, `hsl(222,28%,7%)` background) — **superseded**, not updated to match the current brass/verdigris/oxblood redesign. Don't trust its color/typography section.

## Data schema notes (per `uniData` entry)
- `confidence`: `"verified" | "estimated" | "unverified"` — only set `"verified"` after checking a live official source this session, not from forum/prep-site claims alone.
- `gateType`: `"score-driven" | "residency-gated" | "space-constrained" | "mixed" | "unresearched"` — governs match-estimator logic (`computeMatchStatus()`). **Default new/un-researched entries to `"unresearched"`, never guess `"score-driven"`** — see vault atom `medical-school-explorer-mixed-confidence-uni-schema`. As of 2026-07-21, 12 of 24 schools are still `unresearched`/`unverified`: melbourne, sydney, curtin, griffith, bond, anu, usc, latrobe, csu, cdu, auckland, otago.
- `weighting`: `{atar, ucat, interview}` percentages or `null` if not publicly disclosed by the university — don't invent numbers to fill this in.
- UCAT ANZ score fields assume the **current 0–2700 scale** (rescaled from 0–3600 starting the 2025 test cycle — SJT no longer contributes to the numeric total). See vault atom `ucat-anz-2025-rescale-3600-to-2700`. Any input `max`/placeholder using 3600-scale numbers is a bug.

## Workflow
- Local verification before every push: `python -m http.server <port>` + Chrome browser automation for a visual/console-error check. Validate `<script>` syntax with `node --check` on the extracted block if editing JS by hand risks a typo.
- Git: `main` branch directly, no feature branches. **Never push without the user explicitly saying so in that turn** — "commit and push" confirmation is required each time, not a standing permission.
- Remote: `https://github.com/hyq1031/2026-2027-Medical-School-Explorer.git`. Live: `https://2026-2027-medical-school-explorer.vercel.app/` (auto-deploys from `main` push; allow ~1 min, verify with incognito hard-refresh not curl).

## Design tokens (current palette, post 2026-07 redesign)
Brass/verdigris/oxblood on near-black — replaced the original teal/purple SaaS palette. All colors are CSS custom properties in `:root` (`--primary` = brass `hsl(40,68%,56%)`, `--secondary` = verdigris `hsl(168,42%,40%)`, `--accent` = oxblood `hsl(16,52%,42%)`). Fonts: Fraunces (display), Inter (body), JetBrains Mono (data/mono). Re-theme by editing tokens only — don't hardcode new color literals in component CSS.

## Data-verification sweeper
Say **"run the sweeper"** to re-check `uniData` school records against live official/TAC sources and burn down the unresearched backlog. Skill: `.claude/skills/sweeper/SKILL.md`. Design rationale: `EXPLORER-SWEEPER-SPEC.md`. Ledger/reports/proposals live in `sweeper/`. It is L1 (report-only) by default — it never edits the HTML files or touches git without Kevin approving specific proposals in-session, and never pushes without an explicit "commit and push" that turn.
