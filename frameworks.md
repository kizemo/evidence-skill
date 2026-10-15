# Core Frameworks & Mental Models (12 大)

> 详细内容,按需加载。SKILL.md 只列标题 + 章节。

## 1. **架构真相**:Core 是 AOT 编译的 Docker 二进制,不是 Node 服务

- 静态资源烘焙进 image,CSS 在 build time 烘焙
- `:3002` 是 Docker 暴露的端口(容器内 `:3000` → host `:3002`)
- 配置改完**必须** `docker compose up -d --force-recreate` — `restart` 不重读 bind-mount,AOT binary 启动时 cache connections
- nginx 切 Core 是**5 处 patch**,不是 1 行 sed(详见 ch06)
- **Core 2026 无 SemVer**,GitHub commit-based 迭代,~1.4 days/commit(详见 ch17)

## 2. **配置铁律**:`connection.yaml` FLAT,`evidence.config.yaml` 字符串

- `connection.yaml`:**顶层 `type: postgres`**,不是 `connections: [list]`。错格式 → silent fallback → `unsupported type undefined`
- `evidence.config.yaml`:`project.evidence: "0.10.1"`(**字符串版本号**),不是 object/databases
- Core 默认连 **1 个 PG database**;要跨 DB → 用 `postgres_fdw`(ch07),不要试图加多个 connection

## 3. **SQL 列名雷区**:中文 PG 列必须双引号,`ds`/`shop_name` 是假名

- 中文列名(`"店铺"` `"订单日期"` `"销售金额"`)必须 wrap 双引号
- 假名 → 真名映射: `ds` → `"订单日期"` | `shop_name` → `"店铺"` | `stat_date` → `"订单日期"` 或 `source_year_month`
- raw 表 vs mart 表列名差异:`daily_store_*` 用 `"订单日期"`,`mart_*` 用 `source_year_month`
- **date 类型不能 SUBSTR**:`SUBSTR("订单日期", 1, 7)` PG 报 `function does not exist` → 改 `TO_CHAR("订单日期", 'YYYY-MM')` 或 `"订单日期"::text`

## 4. **nginx 切 Core 必改 5 处**(不是 1 行 sed)

| # | 位置 | 操作 |
|---|---|---|
| 1 | 5 个 vite dev proxy `:3000` | 改 `:3002` |
| 2 | `location / { try_files ... }` | **replace 整块** 为 `proxy_pass :3002`(insert 会 duplicate location 报错) |
| 3 | `location ~ "^/_app/" { root /var/www/evidence; ... }` | **删除**(Core hash 命名不匹配) |
| 4 | `location ~* \.(js\|css\|...)$ { expires ... }` | 加 `proxy_pass :3002` 在 `expires` 前(regex 优先于 catch-all) |
| 5 | `location = / { sub_filter ... }` | 改 `proxy_pass :3002`,删 `sub_filter`(Legacy CSS 注入,Core 不需要) |

## 5. **postgres_fdw 跨 DB**:2-segment 不是 3-segment

- 架构:Core 连 1 个 DB(erp),erp DB 内建 foreign table 代理 taobao DB
- `taobao.mart_X` ✓(2-segment, schema.table)
- `taobao.bi.mart_X` ✗(3-segment 误认为跨 DB ref,PG 不支持)
- `CREATE SERVER` 的 `port` 必须 quoted string `'5432'`,不是 integer
- 当前 4 mart 表 142k 行,Core SQL 简单聚合可接受;大批聚合建议 taobao DB 物化 view

## 6. **html-tags-not-allowed 真阻塞**(不是 limitation) — 唯一解 `{% html %}`

- `<div class="hall-kpi">{% big_value ... %}</div>` → Core 编译时报 `html-tags-not-allowed` → **替换整个父块**包括 big_value → 用户看到 `<Error />` 红色 badge
- 修法 1:用 Evidence 原生 `{% row %}...{% /row %}` 而不是 raw `<div>`
- 修法 2(更通用):**`{% html %}<arbitrary HTML/JS/CSS>{% /html %}`** (2026-06-25 上线,唯一官方 HTML 注入入口)
- `.hall-page` class 在 Core HTML **不存在** — Core wrap 是 `<article>`,写 CSS 用 `article` 选择器

## 7. **`theme.yaml` 5 层 inheritance 真有效** (R12 推翻 ch08 旧结论)

- 5 层:default → org[Studio only] → project `theme.yaml` → page frontmatter → component `chart_options.color_palette`
- 绑定 chart series 色 / 字体 / 圆角 / 间距 / density / depth / table zebra
- **不是装饰性**(2026-06-24 "Expanded theme options" 接入字体/圆角/间距/chart defaults)
- Pentagram Phase 2 路径:70% 视觉走 theme.yaml + 30% 品牌细节走 `{% html %}` block → sales-bi.css 从 ~300 行缩到 ~80 行

## 8. **真验证 SOP**:不只 grep,要直连 DB

- 单独 grep SSR HTML = **虚假 clean**(SSR 用 `<!--[-1-->` placeholder,SQL error 只在 client-side hydration 后才出现)
- 必做 5 步:① 直连 PG 跑 page SQL ② Python `urllib.request` 真 GET HTTP ③ 多 pattern grep(5+ error types)④ 列字段 + 真值查询 ⑤ `force-recreate` 后等 5s 再测
- 失败案例:3 次 "已修复" 实际只修了一个 bug,用户看到的错在另一层(详见 ch09)

## 9. **Direct Connector 无应用层 query cache** (R11 真相)

- changelog 全量搜索 cache|TTL|invalidate 仅得 BigQuery/Snowflake Sync Connector 的 1-min incremental,**Postgres 不在此列**
- Models 是 Studio-only Beta (12h 默认 refresh),Self-host `evidence serve` 不支持
- 性能全靠 **PG 端**(matview / index / pgbouncer session-mode / `statement_timeout=30s`)+ **nginx 端**(`/_app/immutable/` immutable cache)
- pgbouncer **必 session-mode**,transaction-mode 破坏 5 项(search_path / statement_timeout / FDW 事务 / prepared statements / advisory locks)

## 10. **Self-host 认证上限 = Basic Auth** (R7)

- 单密码 `EVIDENCE_BASIC_USER` / `EVIDENCE_BASIC_PASSWORD`
- RLS / SSO / Access Controls / Page-level ACL 全部 **Studio-only**
- Core **不读反代注入 header**(官方原话 "Evidence does not read the authenticated user's identity from the proxy")
- 真 RLS 唯一 clean 方案 = **fork N 副本 + nginx 路由**(短期);长期 issue #3342 (Cube connector) 或迁 Studio Enterprise
- CF Access (Zero Trust) 是 self-host 最实用的 OAuth 增强层

## 11. **直接对手不是 Metabase,是 Lightdash** (R10 战略)

- Evidence 7 候选里**唯一 MIT + 工程师团队匹配的 BI**(Metabase AGPL, Superset/Streamlit Apache-2.0,Lightdash 需 dbt)
- Lightdash 2026 主推 "Agentic BI" + MCP + dbt YAML = "Evidence + dbt 治理层"
- **触发 BI 工具重评阈值**: 团队 ≥5 人 / 报表 >30 / 任一 RLS/SSO/邮件订阅
- 当前 bi.jury.xin 维持 Evidence Core 即可,**当前最佳无对手**

## 12. **AI Agent 调 Self-host Evidence** (R17 MCP 自实现)

- **Studio MCP 是 SaaS-only** (14 tools at `evidence.studio/api/mcp`),**Self-host Core 无 MCP 端点**
- 自实现方案:**Python `mcp` 2.3.0 或独立 `fastmcp` 4.1.0** (推荐 FastMCP,1M 下载/天)
- 8 个自托管 tools: list_pages / get_page_source / get_page_rendered / run_query / list_sources / search_content / get_connection_info / visualize
- **4 道 SQL 防线 (任一被攻破另几层兜底)**:
  1. PG role = readonly (CREATE/INSERT/UPDATE/DELETE/DROP 全部被拒)
  2. sqlglot AST 解析 (识别 DROP/TRUNCATE/DELETE 等危险语句)
  3. SET LOCAL statement_timeout = '10s' (防止慢查询拖死 DB)
  4. LIMIT 10000 (默认上限,require_explicit_higher_limit)
- 集成: Claude Code (stdio) / Cursor (SSE) / Continue (HTTP) / ChatGPT (HTTP+Bearer)
