---
name: evidence-skill
description: Use when configuring, debugging, or upgrading Evidence BI Core 2026 v0.10.1 (bi.jury.xin production deployment) — connecting via connection.yaml / evidence.config.yaml / theme.yaml, encountering SQL `column "ds" does not exist` / `cross-database references are not implemented` / `html-tags-not-allowed` / `unsupported type undefined` / `SUBSTR(date)` errors on Chinese schemas (双引号 `"店铺"` `"订单日期"` `"销售金额"`), patching nginx 5 locations to migrate from Legacy 40.x to Core, setting up postgres_fdw for cross-DB queries, applying Pentagram 5-region CSS, writing `{% html %}` blocks, tuning pgbouncer + statement_timeout + matview, evaluating against Metabase / Lightdash, or implementing FastMCP server for self-hosted AI agent access. Also covers **94 components full reference** (sidebar 8 大类), **Evidence Warehouse single-page truth** (anchor-only), **12 connector → data-sources mapping**. Triggers: 'evidence', 'bi.jury.xin', 'sales-bi', 'cutover', 'connection.yaml FLAT', 'theme.yaml 5 层', 'html block', 'mcp selfhost', 'pgbouncer', 'cloudflare access', 'bi tool selection', 'components 全量', 'evidence-warehouse single page', 'docs mirror', 'sidebar 不等于 sitemap'.
---

<!-- argument-hint: [topic: connection|sql|nginx|fdw|cutover|theme|mcp|upgrade|components|warehouse] -->

# Evidence Core 2026 — Sales BI

**Source**: `E:/obsidian/evidence/` (10 实战笔记) + `E:/obsidian/evidence/_research/` (19 R 系列, 7,150+ 行) + `E:/obsidian/evidence/_sources/evidence-docs/` (168 页 docs mirror, 12 主题) + GBrain (6 pages, 11 facts)
**Generated**: 2026-10-13 (初始) → 2026-10-14 fold-in (R1-R21) → **2026-10-15 v3.1** (168 页 docs mirror 落地 + ch19/ch20)

## Quick Decision Table

| 触发 | 立即做 | 详见 |
|---|---|---|
| 改 Core config | `docker compose up -d --force-recreate` (不是 `restart`) | ch02 |
| 中文 PG 列名 | 双引号包: `"店铺"` `"订单日期"` `"销售金额"` | ch05 |
| date 字段 | `TO_CHAR(date, 'YYYY-MM')` 不是 `SUBSTR(date, ...)` | ch05 |
| 跨 DB | 2-segment path `taobao.mart_X` 不是 3-segment | ch07 |
| 切 nginx | 5 处 patch (不是 1 行 sed) | ch06 |
| 升级 Core | 7 步 SOP + 5 分钟回滚, `upgrade-disabled` 锁 | ch17 |
| html-tags-not-allowed | 改用 `{% html %}<block>{% /html %}` 或 `{% row %}` | ch15 |
| 改主题 | `theme.yaml` 5 层 inheritance 真有效(不是装饰性) | ch15 |
| 性能 | Direct Connector 无 cache → PG 端 + nginx immutable | ch14 |
| Auth 上限 | Self-host 只有 Basic Auth, RLS/SSO 全 Studio | ch13 |
| AI 调 Self-host | 自写 FastMCP + 4 道 SQL 防线 | ch16 |
| 改 BI 工具? | 团队 ≥5 / 报表 >30 / 任一 RLS-SSO-邮件订阅 | ch18 |
| 查具体组件 attrs | 本地 mirror `components/<Class>/components-<name>.md` | ch19 |
| Evidence Warehouse 详情 | 单页 anchor,连接器都在 `data-sources/` | ch20 |
| Sidebar 计数 ≠ sitemap | 先查 sitemap 再决定 BFS 策略 | ch20 |

## Chapter Index (20 章,on-demand 加载)

### 实战篇 (ch01-ch10) — 2026-10-12 切 prod 笔记

| # | 主题 | 关键内容 |
|---|---|---|
| [ch01](chapters/ch01-architecture.md) | Architecture | 40.x vs Core 2026, 5 处 nginx patch, 备份链 |
| [ch02](chapters/ch02-config.md) | Configuration | connection.yaml FLAT, evidence.config.yaml STRING, force-recreate |
| [ch03](chapters/ch03-monitoring.md) | Monitoring | systemd timer, 3 维健康检查 |
| [ch04](chapters/ch04-cutover-failures.md) | Cutover | 时间线, 5 Critical Bug, nginx 切/回滚脚本 |
| [ch05](chapters/ch05-sql-antipatterns.md) | SQL Anti-patterns | 中文列双引号, `ds`/`shop_name` 假名, date SUBSTR 陷阱, html-tags |
| [ch06](chapters/ch06-nginx-patches.md) | nginx Patches | 5 处 patch 详细 diff, 5 秒回滚脚本 |
| [ch07](chapters/ch07-fdw-cross-db.md) | postgres_fdw | 2-segment path, 5 步建表 SQL, 4 mart 表 × 142k 行 |
| [ch08](chapters/ch08-style-unification.md) | Style & UI | 5 region Pentagram CSS Phase 1 |
| [ch09](chapters/ch09-postmortem.md) | Postmortem | 假成功分析, 真验证 SOP 6 步 |
| [ch10](chapters/ch10-ops-quickref.md) | Ops Quick Ref | 服务器路径, commit 链 |

### R 系列篇 (ch11-ch18) — 2026-10-14 全面调研

| # | 主题 | 关键内容 | R 来源 |
|---|---|---|---|
| [ch11](chapters/ch11-components.md) | Components Framework | 7 大类 + Markup 语法 + 雷区 | R3 |
| [ch12](chapters/ch12-deployment.md) | Deployment | 3 模式 + 4 平台 + CLI 13 命令 | R2 |
| [ch13](chapters/ch13-auth-rbac.md) | Auth & RBAC | 7 反代 + self-host 上限 + RLS | R7 |
| [ch14](chapters/ch14-performance.md) | Performance | Direct Connector 无 cache + 4 阶段 actionable | R11, R19 |
| [ch15](chapters/ch15-theme-truth.md) | Theme & Pentagram P2 | theme.yaml 5 层 + 推翻 ch08 + `{% html %}` | R12, R15, R21 |
| [ch16](chapters/ch16-mcp-selfhost.md) | MCP Self-Host | FastMCP + 8 tools + 4 道 SQL 防线 | R17 |
| [ch17](chapters/ch17-upgrade-mgmt.md) | Upgrade & Change Mgmt | 7 步 SOP + 5 分钟回滚 + Obsidian 双向 | R20 |
| [ch18](chapters/ch18-bi-tools.md) | BI Tool Comparison | vs Metabase/Lightdash + 触发重评阈值 | R10, R13 |

### Docs Mirror 篇 (ch19-ch20) — 2026-10-15 168 页 docs mirror 落地

| # | 主题 | 关键内容 | 来源 |
|---|---|---|---|
| [ch19](chapters/ch19-components-full-reference.md) | Components Full Reference | **94 个组件全量索引**(按 sidebar 8 大类) + Quick Decision + Anti-pattern | R3 升级版 |
| [ch20](chapters/ch20-evidence-warehouse-truth.md) | Evidence Warehouse Truth | Single-page + 4 anchor + 12 connector → data-sources mapping | R14 |

## Supporting Files

| 文件 | 用途 | 何时读 |
|---|---|---|
| [frameworks.md](frameworks.md) | 12 大核心 framework 详解 | 需要解释"为什么"时 |
| [topic-index.md](topic-index.md) | 80+ 主题 → chapter 映射 | grep 不到时翻 |
| [glossary.md](glossary.md) | 关键术语定义 (R 系列扩展) | 概念陌生时 |
| [patterns.md](patterns.md) | 8 大模式 (含反模式表) | 复用现成方案时 |
| [cheatsheet.md](cheatsheet.md) | 14 大决策表 + 12 阶段 actionable | 决策快查 |
| [CHANGELOG.md](CHANGELOG.md) | skill 演进历史 | 想知道"为什么改"时 |

## 调研 / 沉淀入口 (外部引用)

- **本地 R 系列**: `E:/obsidian/evidence/_research/` (19 R, 7,150+ 行) — INDEX: `_research/INDEX.md`
- **168 页 docs mirror** (2026-10-15 全量抓取): `E:/obsidian/evidence/_sources/evidence-docs/` — INDEX: `_INDEX.md`
  - 8 个主题独立子目录: `features/` (14) · `core-concepts/` (8) · `components/{8 大类}/` (94) · `cli/` (3) · `MCP/` (3) · `self-host/` (6) · `Direct Connectors/` (8) · `evidence-warehouse/` (1)
  - 抓取脚本: `C:/Temp/evidence_crawler.py` (BFS) + `C:/Temp/backfill_components.py` (sitemap 遍历)
  - sitemap 全覆盖 153 URL,**sidebar ≠ sitemap** 案例见 ch20
- **长期 KB**: `E:/obsidian/_knowledge_base/evidence/2026-10-product-blueprint.md`
- **GBrain recall** (keyless, keyword):
  - `recall projects/sales-bi` — 项目 + 5 facts
  - `recall concepts/pentagram-editorial` — 3 facts
  - `recall concepts/postgres-fdw` — 1 fact
  - `recall companies/evidence-dev` — 1 fact
  - `search "Evidence Core 2026"` — 跨 page 关键字

## Scope & Limits

**覆盖**: Core 2026 v0.10.1 配置/部署/排错/升级/性能/主题/MCP/战略对比; 中文 PG schema; nginx patch; postgres_fdw; theme.yaml 5 层; FastMCP 4 道 SQL 防线; vs Metabase/Lightdash 决策; **94 components 全量 + Evidence Warehouse 真相 (168 页 docs mirror)**

**不覆盖**: Legacy 40.x 新功能; Studio 完整功能; 非 PG 数据库; 多 Core; Pentagram 深度设计(只覆盖 5 region + Phase 2 路径)

**已知限制**: `cryptography` 包损坏 (Python 3.14); 13+ page 待修列名 bug; Browser 真渲染缺 CF Access token; gbrain keyless 无 semantic search; `add_link` 暂时不接未声明 link_type; **gbrain 当前被另一 session 锁定 (PID 864412),write 操作需先 close_owner**; docs mirror `html2text` 转换有标题/段落空行(原 Mintlify 站特性)

**升级频率**: 每周 changelog BREAKING; 每月 PG slow query; 每季 Core 升级 (14 天 hold-back); 每半年 BI 工具重评

**真验证 SOP** (任何"修完宣布成功"前必做): ① 直连 PG 跑 page SQL ② 真 HTTP GET ③ 5+ pattern grep ④ 列字段+真值 ⑤ force-recreate 后等 5s

## Skill 反馈 (踩坑/调研/决策回写)

答 Evidence BI 相关问题、或完成大决策/调研/踩坑时,**主动引导回写到 Obsidian**。

### 触发条件

| 触发 | 立即做 | 模板 |
|---|---|---|
| 切 prod / 升级失败 / ≥2 次重试未修 | 复制 `_playbook/_踩坑模板.md` 写 `E:/obsidian/evidence/YYYY-MM-DD-bug-<short>.md` | postmortem 4 section |
| 新技术 / 新工具 / 重大变更前调研 | 复制 `_playbook/_调研模板.md` 写 `E:/obsidian/evidence/_research/R<NN>-<topic>.md` | R 系列 7 section |
| 不可逆 / ≥1 月影响决策 | 复制 `_playbook/_决策模板.md` 写 `E:/obsidian/evidence/_decisions/YYYY-MM-DD-<short>.md` | ADR 6 section |
| 完成后 | (可选) `gbrain remember fact="..." entity=projects/sales-bi` | — |

### 模板位置 (Obsidian)

- 踩坑模板: `E:/obsidian/evidence/_playbook/_踩坑模板.md`
- 调研模板: `E:/obsidian/evidence/_playbook/_调研模板.md`
- 决策模板: `E:/obsidian/evidence/_playbook/_决策模板.md`
- 入口索引: `E:/obsidian/evidence/_playbook/_README.md`

### 门槛(满足 ≥ 2 条才用)

1. 不可逆(切 prod / 选 BI 工具 / 改认证策略)
2. 影响 ≥ 1 个月
3. 涉及多个有道理的方案
4. 未来 agent 重做时必须知道当时为什么

### 周回写 SOP

每周五或下次 session 开头前自审 3 问:
1. 本周 sales-bi 项目上发生了什么值得沉淀的?
2. 事件类型 = 踩坑 / 调研 / 决策?
3. ≥1 个 → 复制对应模板填骨架,标 TODO 等实测结果

