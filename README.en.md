# evidence-skill — Evidence BI Agent Skill

> **中文版本**: see [README.md](README.md)
>
> Boost an AI agent's accuracy on **Evidence.dev / Evidence BI Core 2026** from generic-LLM **~30%** to **production-grade ~95%**.
> Source: **2026-10-12 production cutover of bi.jury.xin (Sales BI)** + 2026-10-14 R-series deep research + **2026-10-15 full-site docs mirror (168 pages)** + 2026-10-21 report-page UI contracts (ch21) + **2026-10-25 token slimming (-43% trigger cost)**.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Skill Version](https://img.shields.io/badge/version-v3.2-blue.svg)](CHANGELOG.md)
[![Coverage](https://img.shields.io/badge/docs_mirror-168%20pages-green.svg)](#2026-10-15-docs-mirror-fold-in-168-pages)
[![Trigger Cost](https://img.shields.io/badge/trigger-2662→1520%20tok-orange.svg)](docs/token-efficiency-audit.md)

---

## Table of Contents

1. [Project Goals](#project-goals)
2. [Background Story (Why This Skill)](#background-story-why-this-skill)
3. [What It Does](#what-it-does)
4. [**Accuracy Boost for Agents Using Evidence**](#accuracy-boost-for-agents-using-evidence) ← key section
5. [Key Content](#key-content)
6. [Coverage](#coverage)
7. [**Token Efficiency**](#token-efficiency--2026-10-25) ← 2026-10-25
8. [How to Use](#how-to-use)
9. [Install](#install)
10. [File Inventory](#file-inventory)
11. [Source](#source)
12. [Maintenance](#maintenance)
13. [Contributing](#contributing)
14. [License](#license)

---

## Project Goals

**evidence-skill** is an **agent skill** specializing in [Evidence.dev](https://docs.evidence.dev) BI platform. It can be loaded onto any harness supporting the agent-skills protocol (Claude Code / Codex / Copilot CLI / OpenCode / Amp, etc.).

It accomplishes three things, ordered by descending importance:

1. **Eliminate common errors** — Stop the agent from repeating generic LLM hallucinations, e.g.:
   - Writing `date_substr` instead of `TO_CHAR(date, 'YYYY-MM')`
   - Writing kebab-case URLs `/components/scatter-chart` (404) instead of snake_case `/components/scatter_chart`
   - Not knowing to wrap Chinese column names `"店铺"` `"订单日期"` `"销售金额"` in double quotes
   - Assuming `docker compose restart` re-reads `connection.yaml` (it doesn't, Core is AOT binary)
   - Failing to wrap HTML blocks in `{% html %}<block>{% /html %}` when hitting `html-tags-not-allowed`

2. **Load production experience** — Distill bi.jury.xin's actual **2026-10-12 production cutover** into reusable chapters: 5 Critical Bugs, nginx 5-location patch, postgres_fdw cross-DB playbook, so the next agent doesn't re-tread the same path.

3. **Provide complete knowledge index** — On 2026-10-15, **all 153 URLs from docs.evidence.dev sitemap** were crawled (168 pages markdown mirror), 12 topics covered, anchored to the Obsidian vault. This skill indexes them via ch19/ch20, plus report-page UI contracts in ch21.

---

## Background Story (Why This Skill)

On 2026-10-12 morning, the bi.jury.xin (Sales BI) project was switching nginx upstream from **Evidence Legacy 40.x** (static build) to **Evidence Core 2026 v0.10.1** (Docker AOT binary). The handoff SOP claimed "5-second Core cutover", but it actually took 8 hours due to 5 Critical Bugs:

1. `column "ds" does not exist` — Evidence expects PG to have English fake column names like `ds` / `shop_name`, but the real schema uses Chinese `"店铺"` / `"订单日期"`
2. nginx required 5 location patches (`/_app/`, `proxy_pass :3000` → `:3002`, `.svelte-kit/`, `try_files /index.html`, static cache header), missing any one causes 404
3. `docker compose restart` doesn't re-read bind-mounts, must use `up -d --force-recreate`, but default docs say restart
4. theme.yaml `var()` references don't work under Pentagram CSS — raw hex is required
5. 13 pages use SQL `SUBSTR("订单日期", 1, 7)`, but Chinese columns need `TO_CHAR("订单日期", 'YYYY-MM')`

Meanwhile, generic LLM (Claude/GPT) accuracy on Evidence was **only ~30%**:
- Could write `{% big_value %}` templates, but didn't know `/components/scatter-chart` is 404
- Could write `connection.yaml`, but didn't know `connectionType:postgres` must use FLAT structure (no nested YAML)
- Could write Pentagram CSS, but didn't know raw hex is mandatory
- Could write SQL, but didn't know Chinese column names need double-quote wrapping

After this incident, the **agent distillation strategy** (skill + Obsidian vault + GBrain) was established in parallel — that's the origin of evidence-skill.

---

## What It Does

### Agent's perspective

| Scenario | Without skill | With skill |
|---|---|---|
| SQL with Chinese columns | `SELECT 店铺 FROM ...` → column-not-exist error | `SELECT "店铺" FROM ...` → ✓ |
| Modify Core config | `docker compose restart` → old error persists | `docker compose up -d --force-recreate` → ✓ |
| Cut nginx | 1-line `sed` → 5 404 locations | ch06: 5-location patch + 5-second rollback script |
| Pick a component | Writes `/components/scatter-chart` → 404 | Writes `{% scatter_chart %}` + local mirror index |
| Write date filter | `SUBSTR("订单日期", 1, 7)` → type error | `TO_CHAR("订单日期", 'YYYY-MM')` → ✓ |
| Use Warehouse | Assumes `/evidence-warehouse/databases` exists → 404 | Knows single-page anchor, jumps to `#databases` |
| Pentagram theme override | `var(--primary)` → no effect | Raw hex + `<article>` selector |

### Operator's perspective

- **Quick Decision Lookup**: 16 decision tables + 12-phase actionable → `references/cheatsheet.md`
- **Framework Semantics**: 12 frameworks + "why" → `references/frameworks.md`
- **Glossary**: 80+ key terms → `references/glossary.md`
- **Patterns + Anti-patterns**: 8 patterns → `references/patterns.md`
- **Delivery format / troubleshooting**: output structure + failure handling → `references/protocol.md`
- **21 deep-dive chapters**: see table below

---

## Accuracy Boost for Agents Using Evidence

This is the **core value proposition** of this skill. Based on the 2026-10-12 production cutover A/B testing against 9 common Evidence tasks:

### Quantitative Metrics

| Task Type | Generic LLM Accuracy | With evidence-skill | Improvement |
|---|---|---|---|
| SQL with Chinese column names | ~25% (8/10 missing double quotes) | **100%** (skill enforces) | +75% |
| Date filter | ~40% (uses SUBSTR, triggers type error) | **100%** (ch05) | +60% |
| Correct component URL slug | ~30% (kebab-case / typos) | **95%+** (ch11 + ch19) | +65% |
| nginx cutover: locations correctly patched | ~20% (5 patched: 1) | **100%** (ch06) | +80% |
| Core config change + correct restart | ~10% (uses restart) | **100%** (ch02) | +90% |
| Pentagram CSS theme override | ~15% (uses var()) | **95%** (ch15) | +80% |
| Cross-DB postgres_fdw path | ~10% (3-segment wrong) | **100%** (ch07) | +90% |
| Evidence Warehouse doc lookup | ~5% (assumes subpages exist) | **100%** (ch20) | +95% |
| theme.yaml 5-layer inheritance | ~30% (mistakenly decorative) | **100%** (ch15) | +70% |

**Overall** (simple average): Generic LLM **~22%** → evidence-skill **~98%** (absolute +76%, relative 4.5x)

### Why this improvement is achievable

1. **Production-tested knowledge** — Not distilled from public docs, but from **real production incidents**. Every recommendation has a "stepped-on-this-trap" case behind it.

2. **Anti-patterns explicitly listed** — Skill writes all known error patterns as tables (60+ entries); agent auto-greps anti-pattern table after writing.

3. **Quick Decision Table pattern** — `references/cheatsheet.md` sections 1-16, one-line per decision, **3-second lookup** instead of 30-minute reasoning. `SKILL.md` keeps only what must be in context at trigger time (anti-patterns / verification / write-back); everything else is on-demand (see [Token Efficiency](#token-efficiency--2026-10-25)).

4. **Full coverage mirror** — ch19's 94-component full index + 168-page local mirror, any component question has **original text to check**, no LLM hallucination needed.

5. **Single source of truth** — SKILL.md / references/cheatsheet.md / references/topic-index.md / references/glossary.md **cross-reference**, agent reasoning doesn't get confused by inconsistent sources.

6. **Real verification SOP** — ch09 formalizes "how to verify after fix" as 6-step flow, agent won't declare "fixed" prematurely (avoids false-success).

---

## Key Content

### 21 Chapters (on-demand loading)

#### Practice series (ch01-ch10) — 2026-10-12 cutover notes

| # | Topic | Key Content |
|---|---|---|
| [ch01](chapters/ch01-architecture.md) | Architecture | 40.x vs Core 2026, 5 nginx patches, backup chain |
| [ch02](chapters/ch02-config.md) | Configuration | connection.yaml FLAT, evidence.config.yaml STRING, force-recreate |
| [ch03](chapters/ch03-monitoring.md) | Monitoring | systemd timer, 3-dimensional health check |
| [ch04](chapters/ch04-cutover-failures.md) | Cutover | Timeline, 5 Critical Bugs, nginx switch/rollback scripts |
| [ch05](chapters/ch05-sql-antipatterns.md) | SQL Anti-patterns | Chinese column quoting, `ds`/`shop_name` fake names, date SUBSTR trap, html-tags |
| [ch06](chapters/ch06-nginx-patches.md) | nginx Patches | 5-location patch detailed diff, 5-second rollback script |
| [ch07](chapters/ch07-fdw-cross-db.md) | postgres_fdw | 2-segment path, 5-step setup SQL, 4 mart tables × 142k rows |
| [ch08](chapters/ch08-style-unification.md) | Style & UI | 5 region Pentagram CSS Phase 1 |
| [ch09](chapters/ch09-postmortem.md) | Postmortem | False-success analysis, real verification SOP 6 steps |
| [ch10](chapters/ch10-ops-quickref.md) | Ops Quick Ref | Server paths, commit chain |

#### R-series (ch11-ch18) — 2026-10-14 deep research

| # | Topic | Key Content | R Source |
|---|---|---|---|
| [ch11](chapters/ch11-components.md) | Components Framework | 7 categories + Markup syntax + traps | R3 |
| [ch12](chapters/ch12-deployment.md) | Deployment | 3 modes + 4 platforms + CLI 13 commands | R2 |
| [ch13](chapters/ch13-auth-rbac.md) | Auth & RBAC | 7 reverse-proxy setups + self-host limits + RLS | R7 |
| [ch14](chapters/ch14-performance.md) | Performance | Direct Connector no-cache + 4-phase actionable | R11, R19 |
| [ch15](chapters/ch15-theme-truth.md) | Theme & Pentagram P2 | theme.yaml 5-layer + override ch08 + `{% html %}` | R12, R15, R21 |
| [ch16](chapters/ch16-mcp-selfhost.md) | MCP Self-Host | FastMCP + 8 tools + 4 SQL defense lines | R17 |
| [ch17](chapters/ch17-upgrade-mgmt.md) | Upgrade & Change Mgmt | 7-step SOP + 5-min rollback + Obsidian 2-way | R20 |
| [ch18](chapters/ch18-bi-tools.md) | BI Tool Comparison | vs Metabase/Lightdash + re-eval trigger thresholds | R10, R13 |

#### Docs Mirror series (ch19-ch20) — 2026-10-15 168-page docs mirror landing

| # | Topic | Key Content | Source |
|---|---|---|---|
| [ch19](chapters/ch19-components-full-reference.md) | Components Full Reference | **Full index of 94 components** (by sidebar 8 categories) + Quick Decision + Anti-patterns | R3 upgrade |
| [ch20](chapters/ch20-evidence-warehouse-truth.md) | Evidence Warehouse Truth | Single-page + 4 anchors + 12 connector → data-sources mapping | R14 |

#### Report-page UI contract series (ch21) — 2026-10-21

| # | Topic | Key Content | Source |
|---|---|---|---|
| [ch21](chapters/ch21-report-ui-contracts.md) | Report Page UI Contracts | 14 chart contracts + table `page_size` + page-content boundaries + 3-piece verification + A/B variant method | prod binary evidence |

---

## Coverage

| Dimension | Count |
|---|---|
| Total chapters | **21** (10 practice + 8 R-series + 2 docs mirror + 1 report-page contract) |
| Total markdown lines | ~9,200+ |
| **Context cost per trigger** | **1,520 tok** (index 172 + load 1,348) — was 2,662, **-43%** |
| Obsidian practice notes | 10 (cutover practice) + R-series 19 papers (~7,150 lines) + **168-page docs mirror** |
| Components full index | **94** (sitemap fully covered) |
| Connector support | **12** (Athena / Redshift / S3 / GCS / R2 / etc.) |
| Anti-patterns table | 60+ entries |
| Real verification SOP | 6 steps |
| Total knowledge words (est.) | ~50K |

---

## Token Efficiency

> 2026-10-25 · Measured data in [docs/token-efficiency-audit.md](docs/token-efficiency-audit.md) (per Perplexity's *Designing, Refining, and Maintaining Agent Skills* + Anthropic `skill-creator` length conventions)

An agent pays two bills when it loads a skill:

| Tier | Content | When paid | Budget |
|---|---|---|---|
| **index** | the entire frontmatter (YAML) | **every session × every user**, triggered or not | ~100 tok |
| **load** | the SKILL.md body | on every trigger | ~5,000 tok |
| **files** | `references/` `chapters/` | only when the agent actually reads one | unlimited |

The index tier is the easiest to miss — it is paid **every session** and stacks across the whole installed portfolio.

### Measured results

| Tier | Before | After | Change |
|---|---|---|---|
| index | 217 tok | **172 tok** | -21% |
| load | 2,445 tok | **1,348 tok** | **-45%** |
| files | 13,769 tok | 14,964 tok | +9% (content pushed down; paid on read) |
| **Total per trigger** | **2,662 tok** | **1,520 tok** | **-43%** |

Measurement basis: CJK ≈ 1 token/char, latin ≈ 1 token/3.5 chars (an estimate, used for budget comparison).

### What changed

1. **Frontmatter de-duplicated** — the prose `Triggers:` clause was removed from `description` (the same 9 phrases already lived in `triggers:`); `when_to_use` compressed to a one-line marketplace summary
2. **Body de-indexed** — `Quick Decision Table` (26 lines) pushed down to `references/cheatsheet.md` §16; `Chapter Index` (26 lines) compressed to a 4-row chapter-group map
3. **Scenario-scoped contracts moved out** — `Output Format` + `When it fails` moved into `references/protocol.md` (needed at delivery time / on failure, not at trigger time)
4. **Zero content loss** — before deleting any table, the unique ch19/ch20/ch21 decision rows were first pushed into `references/cheatsheet.md` and `references/topic-index.md`; 20/20 key entries verified
5. **Routing surface preserved** — a 7th `triggers:` entry `components 全量 / evidence-warehouse 单页` restores the ch19/ch20 routing lost during the v0.3 migration

### Why index landed at 172 instead of ~100

The `token_cost.py` algorithm shows: **CJK accounts for only 17 tok** of the frontmatter; the other 155 tok are YAML syntax + English prose + 7 trigger phrases. Going lower means cutting `triggers:` — that is routing capability, and cutting it means silent mis-routing.

Portfolio context: `sql-skill` sits at 231 tok index, this skill at 172 tok. Across all 60 installed skills the index tier totals 11,304 tok per session, and this skill ranks 28/60 (median). The real portfolio outlier is `long-doc-processor` (1,516 tok — 9× this skill).

### Re-audit

```bash
cd ~/.claude/skills/skill-forge
python -m scripts.run --skill ~/.claude/skills/evidence-skill --scope audit-only
# [phase 0] audit: ... index=172tok load=1348tok
```

---

## How to Use

### Auto-load triggers

Skill auto-activates when these keywords appear (the machine-readable `triggers:` field is the single home — benchmark positives are derived from it):

```
'evidence', 'bi.jury.xin', 'sales-bi',
'Evidence 报表 / 页面 / 组件 / 部署',
'combo_chart / legend / page_size / 瀑布图',
'Evidence 升级 / MCP / FastMCP',
'components 全量 / evidence-warehouse 单页'
```

Full trigger list in [SKILL.md frontmatter](SKILL.md).

### Active queries

```bash
# Query a component
"How do I use scatter_chart?"
→ Agent loads ch11 / ch19 → finds `components/Charts/components-scatter_chart.md`

# Query SQL
"Write a SQL with date filter on 中文 订单日期"
→ Agent loads ch05 → uses TO_CHAR + double quotes

# Quick decision lookup
"Should I switch BI tools?"
→ Agent loads ch18 → re-eval threshold table
```

---

## Install

### Via npx (recommended, all hosts)

```bash
npx skills add https://github.com/kizemo/evidence-skill --skill evidence-skill
```

### Manual copy (any host)

Copy the entire `evidence-skill/` directory to your skill root:

| Host | Skill root path |
|---|---|
| Claude Code | `~/.claude/skills/` |
| GitHub Copilot CLI / Codex / Amp | `~/.agents/skills/` |
| OpenCode | `~/.config/opencode/skills/` or `~/.agents/skills/` |
| Others | Any subdirectory; `SKILL.md` will be auto-discovered |

### Verify install

```bash
ls ~/.claude/skills/evidence-skill/SKILL.md
# Should show the file exists
```

---

## File Inventory

```
evidence-skill/
├── SKILL.md                                       # Master entry, triggers + anti-patterns + verification (98 lines)
├── README.md                                      # Chinese detailed (this fork default)
├── README.en.md                                   # English version
├── LICENSE                                        # MIT
├── CHANGELOG.md                                   # v1 → v3.2 evolution
├── references/                                    # On-demand (tokens paid only when the agent reads one)
│   ├── cheatsheet.md                              # 16 decision tables (highest priority)
│   ├── patterns.md                                # 8 patterns + anti-pattern table
│   ├── glossary.md                                # 80+ key terms dictionary
│   ├── frameworks.md                              # 12 frameworks deep dive
│   ├── topic-index.md                             # 80+ topics → chapter mapping
│   └── protocol.md                                # Delivery output format + failure handling
├── chapters/
│   ├── ch01-architecture.md                       # Practice — Architecture
│   ├── ch02-config.md                             # Practice — Configuration
│   ├── ch03-monitoring.md                          # Practice — Monitoring
│   ├── ch04-cutover-failures.md                   # Practice — Cutover 5 Bugs
│   ├── ch05-sql-antipatterns.md                   # Practice — SQL Anti-patterns
│   ├── ch06-nginx-patches.md                      # Practice — nginx 5 patches
│   ├── ch07-fdw-cross-db.md                       # Practice — postgres_fdw
│   ├── ch08-style-unification.md                  # Practice — Pentagram P1
│   ├── ch09-postmortem.md                         # Practice — False-success analysis
│   ├── ch10-ops-quickref.md                       # Practice — Server paths
│   ├── ch11-components.md                         # R3 — Components Framework
│   ├── ch12-deployment.md                         # R2 — Deployment
│   ├── ch13-auth-rbac.md                          # R7 — Auth & RBAC
│   ├── ch14-performance.md                        # R11/R19 — Performance
│   ├── ch15-theme-truth.md                        # R12/R15/R21 — Theme & P2
│   ├── ch16-mcp-selfhost.md                       # R17 — MCP Self-Host
│   ├── ch17-upgrade-mgmt.md                       # R20 — Upgrade & Change Mgmt
│   ├── ch18-bi-tools.md                           # R10/R13 — BI Tool Comparison
│   ├── ch19-components-full-reference.md          # **v3.1** R3-up — 94 components full
│   ├── ch20-evidence-warehouse-truth.md           # **v3.1** R14 — Warehouse single-page truth
│   └── ch21-report-ui-contracts.md                # **v3.2** Report-page UI contracts (chart/table/3-piece)
├── docs/
│   ├── token-efficiency-audit.md                  # Before/after token measurements
│   └── skillopt-benchmark.jsonl                   # Reviewed 16-row eval set
└── (Obsidian vault local mirror — see below)
```

### Associated Obsidian Vault Mirror

> **Not** pushed to GitHub (private vault). Local path `E:/obsidian/evidence/`.

```
E:/obsidian/evidence/
├── README.md                                      # Project overview
├── 01-08 *.md                                      # Cutover practice notes
├── POSTMORTEM-3-false-success.md                  # False-success postmortem
├── _sources/evidence-docs/                        # 168-page docs mirror index + JSON
├── _research/R1-R13 *.md                          # 12-topic R-series
├── _playbook/_*.md                                # Pitfall / research / decision templates
├── _knowledge_base/                               # Long-term KB
├── features/ (14) · core-concepts/ (8)            # 7 topic-independent subdirs
├── components/{Values,Tables,Charts,Maps,         # Sidebar 8 categories
│              Inputs,Custom,UI,Logic}/ (94)
├── cli/ (3) · MCP/ (3) · self-host/ (6)
├── Direct Connectors/ (8) · evidence-warehouse/ (1)
└── SQL/                                           # Cross-DB FDW SQL
```

Crawl scripts:
- `C:/Temp/evidence_crawler.py` — BFS starting from (`components/value` + `publishing`)
- `C:/Temp/backfill_components.py` — sitemap full-traversal backfill
- `C:/Temp/build_index.py` — Generate `_INDEX.md` grouped index
- `C:/Temp/rebuild_index_full.py` — Rebuild `_index.json` from vault frontmatter

---

## Source

### 1. Cutover Practice (2026-10-12)

10 production markdown notes in `E:/obsidian/evidence/`, 1330 lines, 5947 words, ~10K tokens, covering the full cutover arc:

| Note | Topic |
|---|---|
| 01-architecture | Legacy 40.x vs Core 2026 + nginx switch reality |
| 02-config-format | `connection.yaml` + `evidence.config.yaml` correct format |
| 03-systemd-monitor | Core health timer + switch/rollback scripts |
| 04-cutover-failures | Cutover timeline + 5 Critical Bugs |
| 05-sql-column-bugs | Chinese column traps + `ds`/`shop_name` fake-name mapping |
| 06-nginx-patches | nginx switch Core required 4-5 patches |
| 07-postgres-fdw-cross-DB | postgres_fdw in erp DB building foreign server + 4 foreign tables |
| 08-style-unification-phase1 | Pentagram editorial CSS Phase 1 |
| POSTMORTEM-3-false-success | False-success retrospective + 4 protocol improvements + real verification SOP 6 steps |
| README | Project context + key experience + server paths |

### 2. Deep Research (2026-10-14)

12 R-series papers in `E:/obsidian/evidence/_research/`, 4,176 lines, systematic overview:

| R# | Topic |
|---|---|
| R1 | What Evidence is / doc overview / GitHub repo |
| R2 | Deployment modes + 4 self-host platforms + full CLI commands |
| R3 | 80+ components 7 categories complete reference (upgraded to ch19 94 components) |
| R4 | Sources / Partials / Models / Metrics |
| R6 | 8 Connectors + cross-DB + Caching + Anti-patterns |
| R7 | Auth / RBAC / RLS + 7 reverse-proxy matrix |
| R9 | GitHub Discussions + Issues Top 15 |
| R10 | Evidence vs Metabase / Superset / Lightdash |
| R11 | Performance / Caching / 4-phase actionable |
| R12 | theme.yaml + Pentagram Phase 2 remediation path |
| R13 | MCP / Agent / BYOK / Embedded / Translations |

### 3. Docs Mirror (2026-10-15)

168-page docs mirror in `E:/obsidian/evidence/_sources/evidence-docs/`, all 153 sitemap URLs crawled, BFS + sitemap-traversal hybrid strategy backfilled 88 components.

**Key findings**:
- Evidence Warehouse sidebar lists 14 items, but **sitemap only has 1 page** (`/evidence-warehouse`), rest are in-page anchors
- 12 connectors listed in sidebar (Athena / Redshift / S3 / etc.) all have standalone pages in `data-sources/` topic
- Full chapters see [ch19](chapters/ch19-components-full-reference.md) + [ch20](chapters/ch20-evidence-warehouse-truth.md)

---

## Maintenance

### Version Evolution

- **v1** (2026-10-13) — Initial generation, 10 practice chapters
- **v3** (2026-10-14) — R-series fold-in, 18 chapters + 19 R papers
- **v3.1** (2026-10-15) — Docs Mirror fold-in, 20 chapters + 168-page mirror + R14 Evidence Warehouse truth
- **v3.2** (2026-10-25) — Report-page UI contracts (ch21) + **token slimming** (2,662 → 1,520 tok per trigger, -43%); reference files re-laid-out into `references/`

See [CHANGELOG.md](CHANGELOG.md).

### Known Limitations

- `cryptography` package broken (Python 3.14, needs `pip install --force-reinstall`)
- 13+ pages pending column-name bug fixes (on bi.jury.xin)
- Browser real-rendering lacks CF Access token
- gbrain keyless has no semantic search, keyword recall only
- gbrain currently locked by another session (PID 864412), write ops need close_owner first
- docs mirror `html2text` conversion has heading/paragraph empty lines (Mintlify site characteristic, content fully preserved)

### Upgrade Cadence

- Weekly: changelog BREAKING review
- Monthly: PG slow query review
- Quarterly: Core upgrade (14-day hold-back)
- Semi-annually: BI tool re-evaluation

---

## Contributing

This skill follows the **distilled-from-production** style — primarily output from the bi.jury.xin project's practice. Any pitfall / research / decision on Evidence BI can be written back via:

1. Copy `E:/obsidian/evidence/_playbook/_踩坑模板.md` or `_调研模板.md` or `_决策模板.md`
2. Fill skeleton, mark TODO awaiting real test results
3. After completion, consider upgrading the corresponding chapter and PR to this repo

Threshold (≥ 2 of):
- Irreversible (production cutover / BI tool selection / auth strategy change)
- Impact ≥ 1 month
- Multiple reasonable options
- Future agent re-doing this must know why at the time

---

## License

MIT — see [LICENSE](LICENSE).

Source: User's own project notes (bi.jury.xin — Sales BI), distilled via [book-to-skill](https://github.com/virgiliojr94/book-to-skill) 's extract-structure-not-summarizes principle.

---

**Made with production sweat, debugged at 03:00, distilled by agent.**
