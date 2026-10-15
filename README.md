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
├── SKILL.md                   # Master entry: 18 core frameworks + chapter + topic index
├── glossary.md                # Alphabetical term index
├── patterns.md                # 5 patterns + anti-patterns
├── cheatsheet.md              # One-page decision tables (highest priority)
├── CHANGELOG.md               # Skill evolution (v1 → v3.1)
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
│   ├── ch10-ops-quickref.md
│   ├── ch11-components.md             # R3 — Components Framework
│   ├── ch12-deployment.md             # R2 — Deployment
│   ├── ch13-auth-rbac.md              # R7 — Auth & RBAC
│   ├── ch14-performance.md            # R11/R19 — Performance
│   ├── ch15-theme-truth.md            # R12/R15/R21 — Theme & Pentagram P2
│   ├── ch16-mcp-selfhost.md           # R17 — MCP Self-Host
│   ├── ch17-upgrade-mgmt.md           # R20 — Upgrade & Change Mgmt
│   ├── ch18-bi-tools.md               # R10/R13 — BI Tool Comparison
│   ├── ch19-components-full-reference.md  # **2026-10-15** R3 升级 — 94 组件全量
│   └── ch20-evidence-warehouse-truth.md   # **2026-10-15** R14 — Evidence Warehouse 单页真相
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

### 2026-10-15 docs mirror fold-in (168 页)

抓取了 docs.evidence.dev **sitemap 153 URL 全部**(BFS + sitemap 遍历混合策略),12 主题沉淀到 `E:/obsidian/evidence/`:

| 主题 | 页数 | 位置 |
|---|---|---|
| **components** (按 sidebar 8 大类) | 94 | `evidence/components/{Values,Tables,Charts,Maps,Inputs,Custom,UI,Logic}/` |
| data-sources | 28 | `evidence/_sources/evidence-docs/` |
| features | 14 | `evidence/features/` |
| core-concepts | 8 | `evidence/core-concepts/` |
| direct-connectors | 8 | `evidence/Direct Connectors/` |
| self-host | 6 | `evidence/self-host/` |
| cli / MCP | 各 3 | `evidence/cli/` `evidence/MCP/` |
| publishing / editing / warehouse | 各 1 | 混合位置 |

**关键发现 (R14)**:
- Evidence Warehouse sidebar 列 14 项,但 **sitemap 只有 1 页**(`/evidence-warehouse`),其余全是页内 anchor
- sidebar 列出的 12 个连接器(Athena / Redshift / S3 / etc.)全部独立成页,已在 `data-sources/` 主题中
- 完整 chapter 见 [ch19](chapters/ch19-components-full-reference.md) + [ch20](chapters/ch20-evidence-warehouse-truth.md)

## Note on content

This skill contains **synthesized summaries**, not raw source text. All content is the user's own project notes from 2026-10-12 production cutover + 2026-10-14 R-series research + 2026-10-15 docs mirror, distilled via book-to-skill's extract-structure-not-summarizes principle.

## License

MIT (matches the source notes' project context).
