# Glossary — Evidence Core 2026 / Sales BI 关键术语

> 按字母顺序排列。每条:term — 定义(chapter)

## A

- **AOT (Ahead-of-Time) compilation** — Core 是 Go/Rust 静态编译二进制,启动时把所有资源烘焙进 process。改 bind-mount 文件必须 `force-recreate`。(ch01, ch02)

- **AOT ELF binary** — `/usr/local/bin/evidence`,Core container 内的主进程,不是 Node 运行时。(ch01)

## B

- **backup chain** — `/etc/nginx/sites-enabled/.bak/sales-bi.legacy.YYYYMMDD-HHMMSS`,切 nginx 前自动备份,5 秒回滚用。(ch01, ch06, ch10)

- **`bi.mart_*` (mart tables)** — BI 层聚合表,列名 `source_year_month` (str 'YYYY-MM')、`店铺` 中文等。(ch05, ch07)

- **bind-mount** — Docker `-v ./:/project` 把 host 目录挂进容器,改 host 文件 → 容器内即时可见,但**进程不会自动 reload**。(ch02)

- **Bridge Error / Catalog Error** — Core Docker logs 扫描的关键 error pattern。(ch03, ch09)

## C

- **`connection.yaml`** — Core 主数据库配置,**FLAT top-level**,不能写 list of dicts。(ch02, ch04)

- **`connectionType: "postgres"`** — SSR HTML 里的字段,curl grep 验证 Core 真的连上 PG。(ch02, ch04)

- **Core 2026 v0.10.1** — Evidence 框架 2026 年版本,独立 Docker container 跑 AOT 二进制。(ch01)

- **`cross-database references are not implemented`** — PG 错误,用 3-segment path `taobao.bi.mart_X` 时报。(ch07)

- **CTE (Common Table Expression)** — `WITH taobao.mart_X AS (...)` 是 local scope,可用。(ch07)

## D

- **`date` type (PG)** — PG date 类型,不能直接 `SUBSTR`,需 `TO_CHAR(date, 'YYYY-MM')` 或 `::text` cast。(ch05)

- **docker compose up -d --force-recreate** — 完全销毁 + 重建 container,触发 bind-mount 重挂载 + 进程重启。Core 改 config 后**唯一可靠 reload 方式**。(ch02, ch04, ch10)

- **docker compose restart** — **不可靠**!StartedAt 不变,文件不重读,AOT binary 仍用旧 config。(ch02, ch04)

- **DSR (Detail Seller Rating)** — 店铺描述 / 服务 / 物流评分,`taobao.mart_shop_kpi_daily` 字段。(ch07)

## E

- **`evidence.config.yaml`** — Core 项目配置,`project.evidence: "0.10.1"` 必须是**字符串版本号**。(ch02, ch04)

- **Evidence Native Row** — `{% row %}...{% /row %}` Evidence 原生 Row 组件,替代 raw `<div class="hall-kpi">` 包裹 component。(ch05, ch08)

## F

- **FLAT top-level (connection.yaml)** — `type: postgres` 直接写在 YAML 顶层,不嵌套在 `connections: [list]`。(ch02, ch04)

- **force-recreate** — 见 `docker compose up -d --force-recreate`。(ch02)

- **foreign server** — `postgres_fdw` 指向远程 PG server 的连接定义(host、port、dbname)。(ch07)

- **foreign table** — 本地表定义但数据在远程 server,query 时 PG 自动代理。(ch07)

- **ForceExitStatus / CriticalExitStatus** — systemd service unit 的语义化 exit code 配置。(ch03)

## G

- **grep multi-pattern verification** — 真验证 4 协议之一,5+ pattern 同时 grep:SQL / Conn / HTML / Render。(ch09)

## H

- **handoff SOP** — 任务交接文档。本项目: `handoff-evidence-core-cutover-v2-2026-10-12.md`。(ch04)

- **hash-named static assets** — `/_app/immutable/HASH.css`,Core 静态资源命名,跟 Legacy `/var/www/evidence/_app/` 不匹配。(ch01, ch06)

- **`html-tags-not-allowed`** — Core 编译 error,触发后**替换整个父块**包括 component,不是 limitation。(ch05, ch08, ch09)

## I

- **information_schema.columns** — PG 系统视图,查表列名 / 数据类型。(ch07)

- **interspect proxy (vite dev)** — 5 个 `proxy_pass :3000` 块,切 Core 改 `:3002`。(ch01, ch06)

## L

- **Legacy 40.x** — Evidence 旧版本,SvelteKit vite dev + 静态 build 混合架构。退役 1-2 周后清。(ch01, ch10)

- **local scope CTE** — `WITH ...` CTE 是 local scope,不触发跨 DB ref。(ch07)

## M

- **mart table** — 聚合层表,`bi.mart_*`。列名差异于 raw 表(用 `source_year_month` 而非 `"订单日期"`)。(ch05, ch07)

- **`mart_sales_monthly` / `mart_after_sales_monthly`** — BI 层月聚合表,店名 `"店铺"`,时间 `source_year_month` (str 'YYYY-MM')。(ch05)

- **`mart_shop_kpi_daily` / `mart_customer_asset_daily`** — 店铺日 KPI / 客户资产日,店名 `"店铺"`,时间 `stat_date` (date)。(ch05)

## N

- **nginx 4 处 patch** — 切 Core 必改 5 处(实际 5 处):5 dev proxy + catch-all replace + `_app/` 删除 + `\.css?$` 加 proxy_pass + sub_filter 删除。(ch01, ch06, ch10)

- **`nginx: [emerg] duplicate location "/"`** — nginx 编译错,因 catch-all 用 `m.start()` insert 而非 `m.end()` replace 触发。(ch01, ch04, ch06)

- **`noscript`-tagged raw `<div>`** — markdown body 写 `<div>` 包裹 component 触发 `html-tags-not-allowed`。(ch05, ch09)

## P

- **Pentagram 编辑设计语言** — 深 navy `#0F2A52` + warm grey `#6B6B6B` + accent gold `#C8A658` + soft cream `#F5F2EC`,5 region 统一。(ch08)

- **Playwright 真渲染** — `CLOUDFLARE_ACCESS_TOKEN` env var + `tests/cf_access_auth.py` 跑 Playwright 5 page 截图,过 CF Access。(ch08, ch09, ch10)

- **postgres_fdw** — PostgreSQL Foreign Data Wrapper,把另一个 PG server 的表映射成本地表。(ch07)

- **`project.evidence` field** — Core 版本指纹字段,必须字符串,不是 object/databases list。(ch02, ch04)

- **proxy_pass :3002** — 切 Core 后 nginx → Core container 的目标地址,容器内 `:3000` → host `:3002`。(ch01, ch06)

## R

- **raw table** — 原始表,`erp.erp_result.daily_store_*` / `daily_sku_*`。列名 `"店铺"`、`"订单日期"` (date)。(ch05)

- **`relation.*does not exist` / `column.*not exist`** — SQL 错误 grep 模式。(ch09)

- **`rows=N` / `pageSize=N` / `sort=false`** — Core v0.10.1 不支持的 table 旧 API,要删。(ch04, ch05)

## S

- **Sales BI (sales-bi)** — 项目名,`bi.jury.xin`,Evidence Core 切 prod 实战项目。(ch01, ch10)

- **sandbox → direct prod evidence** — 5 秒回滚命令 `sudo bash /opt/sales-bi/deploy/scripts/nginx-cutover/nginx_switch_to_legacy.sh`。(ch04, ch06, ch10)

- **silent fallback** — Core 配错不报错,silent 退回默认 → "unsupported type undefined"。反模式。(ch02, ch04, ch09)

- **SSR HTML placeholder** — `<!--[-1-->` Core SSR 用 placeholder 等 client-side fetch,SQL error 不在 SSR 显示。(ch09)

- **systemd timer oneshot** — `Type=oneshot` + `OnUnitActiveSec=5min`,周期性短任务用 timer 不用 cron。(ch03)

## T

- **`taobao.mart_*` (2-segment path)** — `taobao.mart_X` 正确,`taobao` 是 erp DB 内的 local schema,装 foreign tables。(ch07)

- **`taobao.bi.mart_*` (3-segment path,错)** — PG 看到 `taobao` 当 database 名,跨 DB ref 不支持。(ch07)

- **theme.yaml** — Evidence 主题配置文件,**装饰性**,不覆盖 runtime CSS vars(Core v0.10.1 限制)。(ch08)

- **`TO_CHAR(date, 'YYYY-MM')`** — 标准 SQL 格式化 date → string,替代 `SUBSTR(date, 1, 7)`。(ch05)

## U

- **unsupported type undefined** — Core 不知道连什么数据库的典型报错,通常 silent fallback 触发。(ch02, ch04, ch09)

## V

- **vite dev (Legacy)** — Legacy 40.x 的 dev server,跑在 `:3000`,切 Core 后改 `:3002`。(ch01, ch06)

## 2

- **2-segment path** — `taobao.mart_X`,正确跨 DB 路径。(ch07)

## 3

- **3-segment path** — `taobao.bi.mart_X`,错误,触发 `cross-database references are not implemented`。(ch07)

## 中文

- **"店铺"** — PG 中文列名,raw 表 + mart 表都有。SQL 写法:`"店铺"` 双引号包。(ch05)

- **"订单日期"** — PG 中文 date 类型列,需 `TO_CHAR` 或 `::text` cast 才能 SUBSTR。(ch05)

- **"销售金额"** — PG 中文列名,numeric 类型,聚合 SUM 时双引号。(ch05)
