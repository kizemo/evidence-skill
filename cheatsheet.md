# Cheatsheet — Evidence Core 2026 / Sales BI 决策快查

> 一页纸决策表 — 切 / 配 / 排错时第一查这个。**本文件优先级最高**。

---

## 1. 切 prod 决策规则

| 触发 | 立即做 | 因为 |
|---|---|---|
| 准备切 Core | 写切脚本 + 5 秒回滚脚本 + 自动 backup 链 | nginx 切错 5 秒可回 |
| 改 Core config | `docker compose up -d --force-recreate`(不 `restart`) | AOT binary cache 启动时一次 |
| 改 nginx config | `nginx -t` + `nginx -s reload` + 5 page 烟雾测试 | 编译错立即回滚 |
| 改 nginx 块时插入 | **用 `m.end()` 替换,不用 `m.start()` insert** | duplicate `location /` 编译失败 |
| 部署新 page | grep 5+ error patterns + psycopg2 直连测 query | 防止 fake success |
| 监控 silent fallback | 查 `connectionType:postgres` + PG 直连 query | Core silent fallback 不报 SQL error |

## 2. SQL 决策规则

| 触发 | 立即做 | 因为 |
|---|---|---|
| 写中文列名 SQL | 双引号包:`"店铺"` `"订单日期"` `"销售金额"` | PG identifier quote |
| `ds` 假名出现 | 改成 `"订单日期"`(raw 表)或 `source_year_month` (mart 表) | 假名在 PG 不存在 |
| `shop_name` 假名出现 | 改成 `"店铺"` | 13+ page 待修 |
| `stat_date` 假名出现 | 改成 `"订单日期"` 或 `source_year_month` 视 mart 而定 | mart vs raw 表列名差异 |
| 按月聚合 date 列 | `TO_CHAR("订单日期", 'YYYY-MM')` 或 `"订单日期"::text` | `SUBSTR(date, 1, 7)` PG 不支持 |
| 跨 DB 查询 | 2-segment path `taobao.mart_X`,**不**写 `taobao.bi.mart_X` | 3-segment 误认为跨 DB ref |
| 跨 DB new mart | 5 步 SQL:EXTENSION + SERVER + USER MAPPING + SCHEMA + FOREIGN TABLE | erp DB 内装 foreign tables |
| `CREATE SERVER` port | quoted string `'5432'`,**不** integer `5432` | syntax error at or near "5432" |

## 3. nginx 决策规则

| 触发 | 立即做 | 因为 |
|---|---|---|
| 切 Core 改 nginx | 5 处 patch 必做(dev proxy / catch-all / `_app/` / `\.css?$` / sub_filter) | 不是 1 行 sed |
| `location / { try_files ... }` | **REPLACE 整块**为 `proxy_pass :3002` | nginx 不允许 duplicate `location /` |
| `location ~ "^/_app/"` 静态规则 | **删除** | Core hash 名不匹配 → 404 |
| `location ~* \.(js\|css\|...)$ { expires ... }` | 加 `proxy_pass :3002` 在 `expires` 前 | regex 优先于 catch-all |
| `location = / { sub_filter ... }` | 改 `proxy_pass :3002`,删 sub_filter | Legacy CSS 注入,Core 不需要 |
| 切 prod 前 | `cp /etc/nginx/sites-enabled/sales-bi /etc/nginx/sites-enabled/.bak/sales-bi.legacy.YYYYMMDD-HHMMSS` | 5 秒回滚用 |
| 切 prod 失败 | `bash /opt/sales-bi/deploy/scripts/nginx-cutover/nginx_switch_to_legacy.sh` | < 5 秒回滚 |

## 4. CSS 决策规则

| 触发 | 立即做 | 因为 |
|---|---|---|
| Core 写自定义 CSS | **raw hex 值**,不用 `var(--name)` | Core 烘焙 `:root` 优先级最高,var() fallback 失效 |
| Core HTML wrap 选择器 | **`article h1`**,不用 `.hall-page h1` | Core wrap 是 `<article>`,`.hall-page` 不存在 |
| KPI 数字块 | `{% row %}{% big_value ... /%}{% /row %}`,**不**用 `<div class="hall-kpi">` | `<div>` 触发 `html-tags-not-allowed` 替换整块 |
| table API | `<DataTable data={x} />`,**不**用 `rows=N` / `pageSize=N` / `sort=false` | Core v0.10.1 不支持旧 API |
| 改完 nginx config | `bash deploy/scripts/sync_nginx_config_to_git.sh` | 后续 phase 可 reference |
| Playwright 跑 prod | 设 `CLOUDFLARE_ACCESS_TOKEN` env var + 跑 `tests/cf_access_auth.py` | CF Access 拦自动化 |
| 改 theme.yaml | 知道它是**装饰性**,不覆盖 runtime CSS vars | AOT 烘焙 runtime 不重读 |

## 5. 监控 + 排错决策规则

| 触发 | 立即做 | 因为 |
|---|---|---|
| Core silent fallback | `docker logs sales-bi-core --tail 20` 查 `Failed to load` | 配错不抛,silent |
| Core 连不上 PG | `curl :8080/ \| grep -oE 'connectionType:[^,}]*'` | 应见 `connectionType:"postgres"` |
| SQL error 排查 | psycopg2 直连 PG 跑 page SQL(query 真执行) | HTML grep 看 SSR 看不到 client-side error |
| 批量错误排查 | 5+ pattern 同时 grep(SQL / Conn / HTML / Render) | 单 pattern 漏 90% error |
| Core 容器挂 | `bash /opt/sales-bi/deploy/scripts/nginx-cutover/nginx_switch_to_legacy.sh` 5 秒回滚 | Legacy 保留 1-2 周 |
| monitor 报警 | `journalctl -u sales-bi-core-monitor.service -n 20` | 监控只写 journal |
| 修 bug 2 次未成 | **停**,做 `git log` + page 全量审计 + architecture review | "修一个发现另一个"是 fake success 模式 |

## 6. 服务器路径快查

| 用途 | 路径 |
|---|---|
| Core workdir | `/opt/sales-bi/evidence-core/` |
| Legacy workdir | `/opt/sales-bi/evidence/` |
| nginx live config | `/etc/nginx/sites-enabled/sales-bi` |
| nginx backup 链 | `/etc/nginx/sites-enabled/.bak/sales-bi.legacy.YYYYMMDD-HHMMSS` |
| 切脚本 | `/opt/sales-bi/deploy/scripts/nginx-cutover/nginx_switch_to_core.sh` |
| 回滚脚本 | `/opt/sales-bi/deploy/scripts/nginx-cutover/nginx_switch_to_legacy.sh` |
| 监控脚本 | `/opt/sales-bi/deploy/scripts/nginx-cutover/monitor_core.sh` |
| systemd service | `/etc/systemd/system/sales-bi-core-monitor.service` |
| systemd timer | `/etc/systemd/system/sales-bi-core-monitor.timer` |

## 7. CF Tunnel 决策

| 触发 | 立即做 | 因为 |
|---|---|---|
| 公网 URL | `https://bi.jury.xin/`(不加端口)→ CF Tunnel → server :8080 | 443 上加端口通常不通 |
| Playwright 测试 | 设 `CLOUDFLARE_ACCESS_TOKEN` env var | CF Access 拦自动化 |

## 8. 错误模式 grep 5 pattern(同时跑)

| Pattern | 检测层 |
|---|---|
| `does not exist` / `relation.*does not exist` / `column.*not exist` | SQL 层 |
| `unsupported type` / `connection.yaml:` | Connection 层 |
| `html-tags-not-allowed` / `Raw HTML` | HTML 组件层 |
| `Error[^A-Za-z]` / `ErrorBadge` / `error: null` | Runtime 层 |
| `cross-database references are not implemented` | 跨 DB 层 |

任一非 0 → **不 commit**。

## 9. 必读 4 关键经验(任何新会话开篇)

1. **nginx 切 Core 4 处 patch**(不是 1 行 sed):5 dev proxy / catch-all replace / `_app/` 删除 / `\.css?$` 加 proxy_pass / sub_filter 删除
2. **Core config 改完 `force-recreate`**: `docker compose restart` 不可靠,StartedAt 不变
3. **中文列双引号 + `ds`/`shop_name` 是假名**:`"店铺"` `"订单日期"` `"销售金额"`
4. **CF Tunnel 不加端口**:`https://bi.jury.xin/`(不加 `:8080`)

## 10. 5 个下个会话必做(优先级)

| 优先级 | 任务 | 状态 |
|---|---|---|
| P0 | 13+ page 列名 fix 推广(sales-overview 已修) | 🟡 进行中 |
| P0 | 5 秒回滚保留 | ✅ |
| P1 | 多 DB 架构(FDW 已做) | ✅ |
| P1 | Playwright 真渲染(需 user 设 CF Access token) | 🟡 |
| P2 | Phase 2/3 报表优化 + 简报 + DingTalk | 🔵 |
