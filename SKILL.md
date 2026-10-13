---
name: evidence-skill
description: Use when investigating Evidence BI (Core 2026) issues — connection errors ("unsupported type undefined"), SQL errors on Chinese schemas ("column ds does not exist", "function substr(date...)", fake names ds/shop_name/stat_date), cross-DB errors ("cross-database references are not implemented"), nginx cutover problems (duplicate location, _app/ 404, sub_filter conflict), component errors ("html-tags-not-allowed" replaces parent block), or 5-second rollback scenarios. Also use when writing connection.yaml (must be FLAT top-level), evidence.config.yaml (project.evidence must be string), nginx patches, or postgres_fdw setup.
---

<!-- argument-hint: [topic: connection|sql|nginx|fdw|cutover|monitor|style|postmortem|ops] -->

# Evidence Core 2026 — Sales BI

Production-tested knowledge from 2026-10-12 cutover of `bi.jury.xin` (Evidence 40.x → Core 2026).

## Quick Reference

| Need | Read |
|---|---|
| Configure `connection.yaml` / `evidence.config.yaml` / force-recreate | [ch02](chapters/ch02-config.md) |
| nginx cutover (4 required patches, not 1-line sed) | [ch06](chapters/ch06-nginx-patches.md) |
| SQL on Chinese PG schemas, fake column names, `date` SUBSTR trap | [ch05](chapters/ch05-sql-antipatterns.md) |
| Cross-DB query (`postgres_fdw`, 2-segment path) | [ch07](chapters/ch07-fdw-cross-db.md) |
| Cutover timeline + 5 Critical Bugs | [ch04](chapters/ch04-cutover-failures.md) |
| 3 false-success cases + 4 protocols + real-verification SOP | [ch09](chapters/ch09-postmortem.md) |
| Architecture: Legacy 40.x vs Core 2026 | [ch01](chapters/ch01-architecture.md) |
| systemd monitor + health checks | [ch03](chapters/ch03-monitoring.md) |
| CSS override (raw hex, `article` selector) | [ch08](chapters/ch08-style-unification.md) |
| Server paths, commits, next-session todos | [ch10](chapters/ch10-ops-quickref.md) |
| One-page decision table | [cheatsheet.md](cheatsheet.md) |
| Pattern catalog + anti-patterns | [patterns.md](patterns.md) |
| Term index | [glossary.md](glossary.md) |

## Core Principles (must remember)

1. **`connection.yaml` FLAT top-level, `evidence.config.yaml` STRING version** — Core silent-fails on wrong format → "unsupported type undefined".
2. **Config change requires `docker compose up -d --force-recreate`** — AOT binary caches on startup, `restart` doesn't reload.
3. **Chinese PG column names need double-quotes** — `SUM("销售金额")`, not `SUM(销售金额)`.
4. **Fake column names in migrated pages** — `ds` → `"订单日期"`, `shop_name` → `"店铺"`, `stat_date` → `"订单日期"` / `source_year_month` (mart).
5. **`date` type can't `SUBSTR`** — use `TO_CHAR("订单日期", 'YYYY-MM')` or `::text` cast.
6. **`<div class="hall-kpi">` triggers `html-tags-not-allowed`** — replaces entire parent block (including component). Use `{% row %}`.
7. **nginx cutover = 4 patches, not 1 sed** — 5 dev proxies, catch-all replace, `_app/` delete, `\.css?$` proxy_pass, sub_filter delete. 5-second rollback required.
8. **Cross-DB via `postgres_fdw`** — `taobao.mart_X` (2-segment) works; `taobao.bi.mart_X` (3-segment) fails. `CREATE SERVER` port must be quoted string `'5432'`.
9. **Real verification ≠ grep SSR HTML** — Core uses `<!--[-1-->` placeholder; SQL errors only appear post-hydration. Verify with: direct DB query + Python urllib + 5+ error-pattern grep + column-existence check.
10. **CF Tunnel uses 443 only** — `https://bi.jury.xin/` (no `:8080`). Playwright needs `CLOUDFLARE_ACCESS_TOKEN` env var.

## Out of Scope

- Evidence 40.x new development (retired, 1-2 week cleanup window)
- Evidence Cloud managed mode
- Non-Postgres databases
- Multi-Core deployment (single Core + `postgres_fdw` is sufficient)
- Deep Pentagram design work (only the 5 implemented regions)

## Known Follow-ups (from ch10)

- 13+ pages with column-name bugs still need fixing (sales-overview already done)
- Playwright real-render verification blocked on `CLOUDFLARE_ACCESS_TOKEN` from user
- Phase 2/3 (per-page tuning + briefing pages + DingTalk push) not yet planned
