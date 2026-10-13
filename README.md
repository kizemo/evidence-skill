# evidence-skill

> Agent skill distilled from **2026-10-12 Evidence Core 2026 production cutover** (`bi.jury.xin` — Sales BI).

Distilled with [book-to-skill](https://github.com/virgiliojr94/book-to-skill).

## What this skill does

Turns an agent into a **professional Evidence BI (Core 2026) operator** by encoding 10 distilled chapters of production-tested knowledge:

- **Configuration** — `connection.yaml` FLAT, `evidence.config.yaml` STRING, `docker compose up -d --force-recreate` on every change
- **nginx cutover** — 5 required patches (not 1-line sed), 5-second rollback, backup chain
- **SQL anti-patterns** — Chinese column quoting, fake column names (`ds`/`shop_name` → real PG columns), `date` SUBSTR trap, `html-tags-not-allowed` truly-blocking
- **Cross-DB** — `postgres_fdw` with 2-segment path (not 3-segment), 5-step setup SQL
- **Style override** — raw hex (not `var()`), `<article>` selector (not `.hall-page`)
- **Real verification** — direct DB query, multi-pattern grep, never trust SSR HTML alone
- **Postmortem** — 3 false-success case study, 4 protocols, 6-step verification SOP
- **Operations quickref** — server paths, commit chain, key experience, next-session todos

## Install on any Agent Skills host

```bash
# via npx skills (any host)
npx skills add https://github.com/kizemo/evidence-skill --skill evidence-skill
```

Or copy `evidence-skill/` into your host's skill root:
- GitHub Copilot CLI / Codex / Amp: `~/.agents/skills/`
- Claude Code: `~/.claude/skills/`
- OpenCode: `~/.config/opencode/skills/` or `~/.agents/skills/`

## File inventory

```
evidence-skill/
├── SKILL.md                   # Master entry: 10 core frameworks + chapter + topic index
├── glossary.md                # Alphabetical term index
├── patterns.md                # 5 patterns + anti-patterns
├── cheatsheet.md              # One-page decision tables (highest priority)
├── chapters/
│   ├── ch01-architecture.md
│   ├── ch02-config.md
│   ├── ch03-monitoring.md
│   ├── ch04-cutover-failures.md
│   ├── ch05-sql-antipatterns.md
│   ├── ch06-nginx-patches.md
│   ├── ch07-fdw-cross-db.md
│   ├── ch08-style-unification.md
│   ├── ch09-postmortem.md
│   └── ch10-ops-quickref.md
└── README.md                  # this file
```

## Source

Distilled from `E:/obsidian/evidence/` — 10 production markdown notes (1330 lines, 5947 words, ~10K tokens) covering the full cutover arc:

| Note | Topic |
|---|---|
| 01-architecture | Legacy 40.x vs Core 2026 + nginx switch reality |
| 02-config-format | `connection.yaml` + `evidence.config.yaml` correct format (lessons from pitfalls) |
| 03-systemd-monitor | Core health timer + cut/rollback scripts |
| 04-cutover-failures | Cutover timeline + 5 Critical Bug lessons |
| 05-sql-column-bugs | Chinese column name traps + `ds`/`shop_name` false name mapping |
| 06-nginx-patches | 4 nginx patches required to switch to Core |
| 07-postgres-fdw-cross-DB | postgres_fdw in erp DB building foreign server + 4 foreign tables proxying taobao.bi.* |
| 08-style-unification-phase1 | 5 region (Hero/KPI/Chart/Table/Spacing) Pentagram editorial CSS, 11 commits, 5/5 page test pass |
| POSTMORTEM-3-false-success | Retrospective: 3 false "fixed" + 4 protocol improvements |
| README | Project context, key experience, server paths, next-session todos |

## Note on content

This skill contains **synthesized summaries**, not raw source text. All content is the user's own project notes from 2026-10-12 production cutover, distilled via book-to-skill's extract-structure-not-summarizes principle.

## License

MIT (matches the source notes' project context).
