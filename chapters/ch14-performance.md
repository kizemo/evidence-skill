# Chapter 14: Performance & Caching (R11 — 应用层 cache 真空 + 4 阶段 actionable)

## Core Idea

**Direct Connector 无任何应用层 query cache / result cache** — 性能全靠 PG 端(matview / index / pool / replica)+ nginx 端(`/_app/immutable/` immutable cache)。bi.jury.xin 当前 28 页 + FDW 的瓶颈几乎可以确定是 PG 上的串行查询而非 Evidence 应用本身。

## 关键事实

- **Direct Connector: NO cache** — changelog 全量搜索 cache|TTL|invalidate 仅得 BigQuery/Snowflake Sync Connector 的 1-min incremental,**Postgres 不在此列**
- **Sync Connector: 1-min incremental (BQ/Snowflake only, Studio)**
- **Models: Studio-only Beta** (12h 默认 refresh,Enterprise 可配 1h–72h)
- **无内置监控** — 无 Prometheus / OTel / health endpoint / 慢查询日志,只能从 PG `pg_stat_statements` 看

## pgbouncer 铁律

- **必 `pool_mode = session`** — transaction-mode 会让 5 项失效:
  - non-default `search_path`
  - `statement_timeout`
  - FDW 事务
  - prepared statements
  - advisory locks
- 文档明示警告(dc-postgres.html)

## 4 阶段 actionable (bi.jury.xin 优先级)

### 阶段 1 (立刻, <4h)
- nginx `/_app/immutable/` 加 `Cache-Control: public, max-age=31536000, immutable`
- nginx `/api/` 加 `no-cache`
- `EVIDENCE_TELEMETRY_DISABLED=true`
- 删 nginx `ssl_protocols` 中的 TLSv1/TLSv1.1

### 阶段 2 (本周)
- PG 端: `ALTER ROLE kizemo SET statement_timeout = '30s'`
- 启用 `pg_stat_statements` extension
- pgbouncer session-mode (并发 > 50 时才上, < 30 可省)
- 建常用 JOIN 的 matview,`REFRESH MATERIALIZED VIEW CONCURRENTLY`

### 阶段 3 (下周)
- 每页 source 数审查,>20 的拆 partials、>10 的合 SQL file
- B-tree index `("订单日期", "店铺")`
- BRIN index on date column (大表)
- Partial index 90天 hot subset
- `LOWER("店铺")` 表达式索引

### 阶段 4 (并发 viewer > 100 时)
- 横向 Evidence 实例 + PG read replica
- 路由层 (nginx upstream load balance)
- `random_page_cost=1.1` + `effective_io_concurrency=200` (SSD 时代)
- FDW 高频 JOIN 预聚合到本地 PG 表,cron 刷新

## Page Load 性能基准

| Sources/page | 典型 load | 评估 |
|---|---|---|
| < 5 | < 1s | ✓ 健康 |
| 5–10 | ~5s | ⚠️ 接近上限 |
| 10–20 | ~10s | 🟡 拆 partials |
| 20–50 | > 15s | 🔴 必修(合 SQL file 或 matview) |
| 50+ | 不可用 | ❌ 架构问题 |

## Postgres Tuning 关键 4 杠杆

| 杠杆 | 推荐值 | 原因 |
|---|---|---|
| `statement_timeout` | 30s | 防止单 query 阻塞 |
| `random_page_cost` | 1.1 (SSD) | BI 索引扫描效率翻倍 |
| `effective_io_concurrency` | 200 (SSD) | 提升 bitmap scan 性能 |
| `work_mem` | 64MB+ | 大排序 / hash 聚合 |

## 反模式 (R9 社区 + 官方背书)

- ❌ SELECT * on large tables → 必须 explicit columns
- ❌ > 20 sources / page → 拆 partials
- ❌ unfiltered cross-joins → INCLUDE filter always
- ❌ unbounded date ranges → 必加 WHERE 时间范围
- ❌ 高频跨 DB FDW JOIN → 预聚合本地表
- ❌ `pgbouncer transaction-mode` → 必 session-mode

## Connects To

- **ch07** — FDW 性能 + session-mode 必选
- **ch06** — nginx `/_app/immutable/` cache header
- **ch10** — 监控 + 排错
- **ch19 (MCP self-host)** — `run_query` tool 必加 `SET LOCAL statement_timeout` 防护
