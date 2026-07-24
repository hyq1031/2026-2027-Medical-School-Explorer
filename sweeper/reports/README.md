# Sweeper reports

One file per run: `YYYY-MM-DD.md`.

Each report summarizes: N schools checked, N drifted from `uniData`, N proposals written to `../proposed/`, N blocked (see spec §7 circuit breakers), and any Firecrawl fallback usage. Kevin reads this to decide what to approve — see `EXPLORER-SWEEPER-SPEC.md` §5 (human gate) and `.claude/skills/sweeper/SKILL.md`.
