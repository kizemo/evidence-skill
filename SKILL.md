---
name: evidence-skill
description: "Evidence BI (Core 2026) professional operations knowledge distilled from production cutover. Use when configuring Evidence Core (connection.yaml / evidence.config.yaml), debugging SQL column errors on Chinese schemas, patching nginx for Core cutover, setting up postgres_fdw for cross-DB queries, troubleshooting 'unsupported type undefined' / 'column does not exist' / 'html-tags-not-allowed' errors, or running /verifying cutovers. Triggers: 'evidence', 'core 2026', 'connection.yaml', 'evidence.config', 'cutover', 'bi.jury.xin', 'pentagram theme', 'sales-bi', 'postgres_fdw', 'substr date', 'shop_name 假列名'."
---

<!-- argument-hint: [topic: connection|sql|nginx|fdw|cutover|monitor|style|postmortem] -->

# Evidence Core 2026 — Sales BI 实战知识库

**Source**: 2026-10-12 prod 切 prod 实战笔记(E:/obsidian/evidence/,10 文件)
**Generated**: 2026-10-13 | **Purpose**: 让 agent 更专业地使用和配置 Evidence BI(尤其 Core 2026 切 prod 后的运维 / 配置 / 排错)

## How to Use This Skill

- **Without args** — load core 知识做参考
- **With topic** — 问 `connection` / `sql` / `nginx` / `fdw` / `cutover` / `monitor` / `style` / `postmortem`,我读对应 chapter
- **With chapter** — `ch05` 直接加载第 5 章
- **Browse** — 问 "你有哪些 chapter?"

问 Core 2026 任意细节,我会先读相关 chapter 再答。

---

## Core Frameworks & Mental Models

### 1. **架构真相**:Core 是 AOT 编译的 Docker 二进制,不是 Node 服务

- 静态资源烘焙进 image,CSS 在 build time 烘焙,`theme.yaml` 只装饰性,**不覆盖 runtime CSS vars**
- `:3002` 是 Docker 暴露的端口(容器内 `:3000` → host `:3002`)
- 配置改完**必须** `docker compose up -d --force-recreate` —— `restart` 不重读 bind-mount,AOT binary 启动时 cache connections
- nginx 切 Core 是**4 处 patch**,不是 1 行 sed(详见 ch06)

### 2. **配置铁律**:`connection.yaml` FLAT,`evidence.config.yaml` 字符串

- `connection.yaml`:**顶层 `type: postgres`**,不是 `connections: [list]`。错格式 → silent fallback → `unsupported type undefined`
- `evidence.config.yaml`:`project.evidence: "0.10.1"`(**字符串版本号**),不是 object/databases
- Core 默认连 **1 个 PG database**;要跨 DB → 用 `postgres_fdw`(ch07),不要试图加多个 connection

### 3. **SQL 列名雷区**:中文 PG 列必须双引号,`ds`/`shop_name` 是假名

- 中文列名(`"店铺"` `"订单日期"` `"销售金额"`)必须 wrap 双引号
- 假名 → 真名映射:
  - `ds` → `"订单日期"`
  - `shop_name` → `"店铺"`
  - `stat_date` → `"订单日期"` 或 `source_year_month` (mart 表)
- raw 表 vs mart 表列名差异:`daily_store_*` 用 `"订单日期"`,`mart_*` 用 `source_year_month` (str 'YYYY-MM')
- **date 类型不能 SUBSTR**:`SUBSTR("订单日期", 1, 7)` PG 报 `function substr(date, integer, integer) does not exist`
- 修法:`TO_CHAR("订单日期", 'YYYY-MM')` 或 `"订单日期"::text`

### 4. **nginx 切 Core 必改 4 处**(不是 1 行 sed)

| # | 位置 | 操作 |
|---|---|---|
| 1 | 5 个 vite dev proxy `:3000` | 改 `:3002` |
| 2 | `location / { try_files ... }` | **replace 整块** 为 `proxy_pass :3002`(insert 会 duplicate location 报错) |
| 3 | `location ~ "^/_app/" { root /var/www/evidence; ... }` | **删除**(Core hash 命名不匹配) |
| 4 | `location ~* \.(js\|css\|...)$ { expires ... }` | 加 `proxy_pass :3002` 在 `expires` 前(regex 优先于 catch-all) |
| 5 | `location = / { sub_filter ... }` | 改 `proxy_pass :3002`,删 `sub_filter`(Legacy CSS 注入,Core 不需要) |

### 5. **postgres_fdw 跨 DB**:2-segment 不是 3-segment

- 架构:Core 连 1 个 DB(erp),erp DB 内建 foreign table 代理 taobao DB
- `taobao.mart_X` ✓(2-segment, schema.table)
- `taobao.bi.mart_X` ✗(3-segment 误认为跨 DB ref,PG 不支持)
- `CREATE SERVER` 的 `port` 必须 quoted string `'5432'`,不是 integer
- 当前 4 mart 表 142k 行,Core SQL 简单聚合可接受;大批聚合建议 taobao DB 物化 view

### 6. **html-tags-not-allowed 真阻塞**(不是 limitation)

- `<div class="hall-kpi">{% big_value ... %}</div>` → Core 编译时报 `html-tags-not-allowed` → **替换整个父块包括 big_value** → 用户看到 `<Error />` 红色 badge
- 修法:用 Evidence 原生 `{% row %}...{% /row %}` 而不是 raw `<div>`
- `.hall-page` class 在 Core HTML **不存在** — Core wrap 是 `<article>`,写 CSS 用 `article` 选择器

### 7. **真验证 SOP**:不只 grep,要直连 DB

- 单独 grep SSR HTML = **虚假 clean**(SSR 用 `<!--[-1-->` placeholder,SQL error 只在 client-side hydration 后才出现)
- 必做 5 步:① 直连 PG 跑 page SQL ② Python `urllib.request` 真 GET HTTP ③ 多 pattern grep(5+ error types)④ 列字段 + 真值查询 ⑤ `force-recreate` 后等 5s 再测
- 失败案例:3 次 "已修复" 实际只修了一个 bug,用户看到的错在另一层(详见 ch09)

### 8. **6 项 git commit 经验教训**(ch04 + ch09 综合)

- `a141446` fix `connection.yaml FLAT top-level` (Bug #1)
- `fcb6bda` fix `evidence.config.yaml project.evidence STRING` (Bug #2)
- `601ac00` fix `date_filter hardcode` (Bug #3-#5: `{{ date_filter.start }}` 错语法,`sort=false`/`pageSize=N` 不支持,raw `<div>` warning)
- `f18a74d` fix `SUBSTR(date) → TO_CHAR` + `<div class="hall-kpi"> → {% row %}`(Bug A 真根因 + Bug B)
- `b3bba28` feat nginx switch + monitor
- `ebe871f` fix replace (not insert) catch-all

### 9. **3 大服务器路径**(速查)

- Core workdir: `/opt/sales-bi/evidence-core/` (Docker)
- Legacy workdir: `/opt/sales-bi/evidence/`(切完才能清,1-2 周观察期)
- nginx live: `/etc/nginx/sites-enabled/sales-bi`
- 切/回滚/监控脚本: `/opt/sales-bi/deploy/scripts/nginx-cutover/`
- systemd timer: `/etc/systemd/system/sales-bi-core-monitor.{service,timer}`
- backup 链: `/etc/nginx/sites-enabled/.bak/sales-bi.legacy.YYYYMMDD-HHMMSS`

### 10. **Cloudflare Tunnel 转发**

- Public URL: `https://bi.jury.xin/`(不加端口)→ CF Tunnel → server nginx :8080
- **别加 `:8080` 到公网 URL**,443 上加端口通常不通
- Playwright 截图需 `CLOUDFLARE_ACCESS_TOKEN` env var + `tests/cf_access_auth.py`

---

## Chapter Index

| # | Title | 关键内容 |
|---|---|---|
| [ch01](chapters/ch01-architecture.md) | Architecture: 40.x vs Core 2026 | Legacy 静态 build vs Core AOT binary,4 处 nginx patch,备份链 |
| [ch02](chapters/ch02-config.md) | Configuration Files | connection.yaml FLAT,evidence.config.yaml STRING,force-recreate |
| [ch03](chapters/ch03-monitoring.md) | Monitoring & Operations | systemd timer,3 维健康检查,monitor 脚本微 bug |
| [ch04](chapters/ch04-cutover-failures.md) | Cutover Methodology | 时间线,5 Critical Bug,nginx 切/回滚脚本 |
| [ch05](chapters/ch05-sql-antipatterns.md) | SQL Anti-patterns | 中文列双引号,`ds`/`shop_name` 假名,date SUBSTR 陷阱,html-tags-not-allowed |
| [ch06](chapters/ch06-nginx-patches.md) | nginx Patches | 5 处 patch 详细 diff,5 秒回滚脚本,保留的旧块 |
| [ch07](chapters/ch07-fdw-cross-db.md) | Cross-DB with postgres_fdw | 2-segment path,5 步建表 SQL,4 mart 表 × 142k 行,反思 |
| [ch08](chapters/ch08-style-unification.md) | Style & UI | 5 region Pentagram CSS,11 commits,raw hex 不用 var() |
| [ch09](chapters/ch09-postmortem.md) | Postmortem: 3 False Successes | 假成功分析,4 协议改进,真验证 SOP 6 步 |
| [ch10](chapters/ch10-ops-quickref.md) | Operations Quick Reference | 服务器路径,commit 链,4 关键经验,下个会话必做 |

## Topic Index

- **AOT binary** → ch01, ch02
- **架构(40.x vs Core 2026)** → ch01
- **backup 链** → ch01
- **bugs (5 critical)** → ch04
- **CF Tunnel** → ch10
- **`column "ds" does not exist`** → ch05
- **commit 链** → ch10
- **config 改完不生效** → ch02
- **`connection.yaml` FLAT** → ch02, ch04
- **`connectionType:postgres`** → ch02
- **Core 启动 cache connections** → ch02
- **cross-DB query** → ch07
- **cutover 时间线** → ch04
- **date SUBSTR 陷阱** → ch05
- **`docker compose up -d --force-recreate`** → ch02, ch04
- **evidence.config.yaml STRING** → ch02, ch04
- **`unsupported type undefined`** → ch02, ch04
- **5 秒回滚** → ch06
- **fake success** → ch09
- **false column names** → ch05
- **grep multi-pattern** → ch09
- **`html-tags-not-allowed` 真阻塞** → ch05, ch09
- **monitor timer** → ch03
- **nginx 4 处 patch** → ch01, ch06
- **`pkill -f vite dev`** → ch10 (legacy cleanup)
- **`postgres_fdw`** → ch07
- **真验证 SOP** → ch09
- **silent fallback** → ch02
- **server paths** → ch10
- **shop_name 假名** → ch05
- **systemd timer** → ch03
- **3-segment path 错** → ch07
- **TO_CHAR date 替代** → ch05
- **2-segment path 对** → ch07
- **Pentagram 主题** → ch08
- **Row 组件 替代 `<div>`** → ch05
- **真验证 = 直连 DB + 多 pattern grep** → ch09

## Supporting Files

- [glossary.md](glossary.md) — Evidence Core / FDW / nginx / SQL 关键术语
- [patterns.md](patterns.md) — 5 大模式(connection / nginx patch / FDW setup / style / verification)
- [cheatsheet.md](cheatsheet.md) — 决策快查 + 假名映射 + 修法对照表

---

## Scope & Limits

**覆盖**:
- Evidence Core 2026 v0.10.1 的**配置 / 部署 / 排错** 实战知识
- 中文 PG schema 的 SQL 写法 + 列名映射
- nginx 切 Core 的 patch 模式
- postgres_fdw 跨 DB 查询 setup
- 切 prod 的真验证 SOP

**不覆盖**:
- Evidence 40.x (Legacy) 新功能开发(已退役,1-2 周清)
- Evidence Cloud 模式
- 非 PG 数据库(MySQL / SQLite / BigQuery)
- 多 Core deployment(单 Core + FDW 已够用)
- 样式 / Pentagram 主题深度设计(只覆盖已实现的 5 region)

**已知**:
- `cryptography` 包损坏(本机 Python 3.14 metadata bug,非 Evidence 问题)
- 13+ page 待修列名 bug(下一步工作,见 ch10)
- Browser 真渲染验证缺 CF Access token(用户设 `CLOUDFLARE_ACCESS_TOKEN` 后可跑)
