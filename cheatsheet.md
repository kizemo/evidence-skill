# Cheatsheet — Evidence BI 决策快查

> 一页纸决策表 — 切 / 配 / 排错 / 升级 / 性能时第一查这个。**本文件优先级最高**。
> 覆盖: ch01-ch10 实战 + ch11-ch18 R 系列系统全貌

---

## 1. 切 prod 决策规则

| 触发 | 立即做 | 因为 |
|---|---|---|
| 准备切 Core | 写切脚本 + 5 秒回滚脚本 + 自动 backup 链 | nginx 切错 5 秒可回 |
| 改 Core config | `docker compose up -d --force-create`(不 `restart`) | AOT binary cache 启动时一次 |
| 改 nginx config | `nginx -t` + `nginx -s reload` + 5 page 烟雾测试 | 编译错立即回滚 |
| 改 nginx 块时插入 | **用 `m.end()` 替换,不用 `m.start()` insert** | duplicate `location /` 编译失败 |
| 部署新 page | grep 5+ error patterns + psycopg2 直连测 query | 防止 fake success |
| 监控 silent fallback | 查 `connectionType:postgres` + PG 直连 query | Core silent fallback 不报 SQL error |
| 升级 Core | 7 步 SOP + 14 天 hold-back + commit sha baseline | Core 无 SemVer,commit-based |
| 升级后回滚 | 改 tag + force-recreate (5min 内) | `upgrade-disabled` 锁版本 |

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
| MCP `run_query` | 4 道防线 (readonly + sqlglot + statement_timeout + LIMIT) | Self-host MCP 必安全 |
| PG 性能问题 | `statement_timeout=30s` + matview + B-tree index | Core 无应用层 cache |

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
| 静态资源 cache | `/_app/immutable/` 加 `Cache-Control: public, max-age=31536000, immutable` | 唯一免费优化杠杆 |
| 禁用旧 TLS | 删 `ssl_protocols` 中的 TLSv1/TLSv1.1 | PCI 6.5.4 + 安全扫描要求 |

## 4. CSS / Theme 决策规则 (R12 推翻 ch08 旧结论)

| 触发 | 立即做 | 因为 |
|---|---|---|
| 改主题颜色/字体/圆角 | **`theme.yaml`** (5 层 inheritance 真有效) | Core 2026-06-24 后支持 |
| 改 Pentagram 70% 视觉 | `colors.base` / `radius` / `depth` / `density` / `chart.barRadius` | theme.yaml 原生 |
| 改 Pentagram 品牌专属 | `{% html %}<style>{% /html %}` 块 | 唯一官方 HTML 注入入口 |
| raw `<div>` / `<style>` markdown body | 改用 `{% html %}` block | 触发 `html-tags-not-allowed` 替换整块 |
| 旧路径 raw hex + article 选择器 | 改为 theme.yaml + `{% html %}` | sales-bi.css 从 ~300 行缩到 ~80 行 |
| `var(--name)` 期待 fallback | **放弃**,改用 theme.yaml 字段 | Core 烘焙 `:root` 优先级最高,var() 失效 |
| `.hall-page` 选择器 | **放弃**,改用 `article` | Core wrap 是 `<article>`,`.hall-page` 不存在 |
| KPI 数字块 | `{% row %}{% big_value ... /%}{% /row %}`,**不**用 `<div class="hall-kpi">` | `<div>` 触发 `html-tags-not-allowed` |
| table API | `<DataTable data={x} />`,**不**用 `rows=N` / `pageSize=N` / `sort=false` | Core v0.10.1 不支持旧 API |
| 改完 nginx config | `bash deploy/scripts/sync_nginx_config_to_git.sh` | 后续 phase 可 reference |
| Playwright 跑 prod | 设 `CLOUDFLARE_ACCESS_TOKEN` env var + 跑 `tests/cf_access_auth.py` | CF Access 拦自动化 |

## 5. Components 决策规则 (R3)

| 触发 | 立即做 | 因为 |
|---|---|---|
| KPI 大数字 | `{% big_value %}` + `{% delta %}` | 大号 + 同比 |
| 中文表头 | 加 `format_titles=false` 属性 | Title Case 破坏中文别名 |
| 唯一 date picker | `{% range_calendar %}` | `/components/date_picker` 404 |
| Checkbox 标签 | `{% toggle %}` | `/components/checkbox` URL 错位 |
| 高级图表 (box_plot / waterfall) | `{% custom_echart %}` | 内置缺失,逃逸 ECharts |
| URL 路径 | snake_case `/components/scatter_chart`,**不** kebab-case | Mintlify 404 模式 |

## 6. Auth 决策规则 (R7)

| 触发 | 立即做 | 因为 |
|---|---|---|
| Self-host Basic Auth 密码 | `openssl rand -base64 32` (≥24 chars) | 单密码共享,弱密码全员漏 |
| Self-host `--host 0.0.0.0` | **必设 Basic Auth** 或 `EVIDENCE_AUTH_DISABLED=true` | Core 拒绝裸跑 |
| 反代 OAuth (Google Workspace) | CF Access + email allowlist + MFA | Self-host 不读反代 header |
| 真 RLS 需求 | fork N 副本 + nginx 路由,**不**期望 Core 读 header | Core 明确不读 |
| Studio 模式 | access.yaml + RBAC + RLS + SSO (需 Enterprise) | Self-host 无 RLS |
| 改 Basic Auth 密码 | 通知 viewer 全员,浏览器无登出按钮 | 改密码 = 全员下线 |

## 7. 监控 + 排错决策规则

| 触发 | 立即做 | 因为 |
|---|---|---|
| Core silent fallback | `docker logs sales-bi-core --tail 20` 查 `Failed to load` | 配错不抛,silent |
| Core 连不上 PG | `curl :8080/ \| grep -oE 'connectionType:[^,}]*'` | 应见 `connectionType:"postgres"` |
| SQL error 排查 | psycopg2 直连 PG 跑 page SQL(query 真执行) | HTML grep 看 SSR 看不到 client-side error |
| 批量错误排查 | 5+ pattern 同时 grep(SQL / Conn / HTML / Render / Cross-DB) | 单 pattern 漏 90% error |
| Core 容器挂 | `bash /opt/sales-bi/deploy/scripts/nginx-cutover/nginx_switch_to_legacy.sh` 5 秒回滚 | Legacy 保留 1-2 周 |
| monitor 报警 | `journalctl -u sales-bi-core-monitor.service -n 20` | 监控只写 journal |
| 修 bug 2 次未成 | **停**,做 `git log` + page 全量审计 + architecture review | "修一个发现另一个"是 fake success 模式 |

## 8. Performance 决策规则 (R11)

| 触发 | 立即做 | 因为 |
|---|---|---|
| Core 慢查询 | 先 PG `pg_stat_statements` 查慢 query | Core 无内置监控 |
| 每页 source > 20 | 拆 partials + 合 SQL file | Core 串行执行,> 20 不可接受 |
| 高频跨 DB JOIN | 预聚合本地 PG 表,cron 刷新 | FDW 性能差,silent bug 多 |
| pgbouncer 必设 | `pool_mode = session` | transaction-mode 破坏 5 项 |
| 静态资源 cache | nginx `/_app/immutable/` immutable header | 唯一免费杠杆 |
| 内存 ≥ 512MB | Fly.io 256MB OOM,Docker 推荐 512MB+ | Docs 明示踩坑 |
| `pg_stat_statements` | `shared_preload_libraries = 'pg_stat_statements'` | 看慢 query 唯一手段 |
| 物化 view | `REFRESH MATERIALIZED VIEW CONCURRENTLY` | 不锁表,生产可用 |

## 9. MCP Self-Host 决策规则 (R17)

| 触发 | 立即做 | 因为 |
|---|---|---|
| AI agent 调 Evidence | 自写 FastMCP (Python) server | Self-host 无 MCP 端点 |
| `run_query` tool 暴露 | 4 道 SQL 防线 (readonly + sqlglot + timeout + LIMIT) | 任何 1 道被攻破另几层兜底 |
| `get_connection_info` | yaml.safe_load + strip password | 永不含 password |
| 客户端集成 | Claude Code (stdio) / Cursor (SSE) / Continue (HTTP) / ChatGPT (HTTP+Bearer) | 4 种模式 |
| 部署 | systemd + nginx `proxy_pass` | Bearer token 鉴权 |

## 10. 服务器路径快查

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
| 本地调研笔记 | `E:/obsidian/evidence/_research/` (19 R 系列) |
| 长期 KB | `E:/obsidian/_knowledge_base/evidence/2026-10-product-blueprint.md` |
| GBrain recall 入口 | `projects/sales-bi` / `concepts/pentagram-editorial` / `concepts/postgres-fdw` |

## 11. CF Tunnel 决策

| 触发 | 立即做 | 因为 |
|---|---|---|
| 公网 URL | `https://bi.jury.xin/`(不加端口)→ CF Tunnel → server :8080 | 443 上加端口通常不通 |
| Playwright 测试 | 设 `CLOUDFLARE_ACCESS_TOKEN` env var | CF Access 拦自动化 |
| OAuth 增强 | CF Access + email allowlist + MFA | Self-host 不读反代 header |

## 12. 错误模式 grep 5 pattern(同时跑)

| Pattern | 检测层 |
|---|---|
| `does not exist` / `relation.*does not exist` / `column.*not exist` | SQL 层 |
| `unsupported type` / `connection.yaml:` | Connection 层 |
| `html-tags-not-allowed` / `Raw HTML` | HTML 组件层 |
| `Error[^A-Za-z]` / `ErrorBadge` / `error: null` | Runtime 层 |
| `cross-database references are not implemented` | 跨 DB 层 |

任一非 0 → **不 commit**。

## 13. 必读 12 关键经验(任何新会话开篇)

1. **nginx 切 Core 4 处 patch**(不是 1 行 sed):5 dev proxy / catch-all replace / `_app/` 删除 / `\.css?$` 加 proxy_pass / sub_filter 删除
2. **Core config 改完 `force-recreate`**: `docker compose restart` 不可靠,StartedAt 不变
3. **中文列双引号 + `ds`/`shop_name` 是假名**:`"店铺"` `"订单日期"` `"销售金额"`
4. **CF Tunnel 不加端口**:`https://bi.jury.xin/`(不加 `:8080`)
5. **`theme.yaml` 5 层 inheritance 真有效** (R12 推翻 ch08 旧结论):70% Pentagram 视觉原生
6. **`{% html %}` block 是 html-tags 唯一解** (2026-06-25 上线):`{% html %}<style>{% /html %}` 替代 raw `<div>`
7. **Direct Connector 无应用层 query cache** (R11):PG 端 matview/index + nginx immutable 是唯一杠杆
8. **Self-host 只有 Basic Auth** (R7):RLS / SSO 全部 Studio-only;真 RLS 要 fork N 副本 + nginx 路由
9. **Core 无 SemVer** (R1):commit-based,周更级,`evidence version` + commit sha 入库作内部锚点
10. **升级 7 步 SOP + 5 分钟回滚** (R20):force-recreate + 14 天 hold-back
11. **AI agent 调 Self-host Evidence** (R17):自实现 FastMCP + 4 道 SQL 防线
12. **直接对手不是 Metabase,是 Lightdash** (R10):触发 BI 工具重评阈值 = 团队 ≥5 / 报表 >30 / 任一 RLS-SSO-邮件订阅

## 14. 12 大 Phase actionable (5 阶段路线)

| 优先级 | 任务 | 状态 | 引用 |
|---|---|---|---|
| **P1 本周** | nginx immutable + telemetry off + Basic Auth 强密码 + audit log | 🟡 进行中 | ch14 + ch13 |
| **P1 本周** | 删 nginx `ssl_protocols` TLSv1/1.1 | 🔵 未做 | ch14 |
| **P2 本月** | PG `statement_timeout=30s` + `pg_stat_statements` + matview | 🔵 未做 | ch14 |
| **P2 本月** | theme.yaml 接 Pentagram 70% 视觉,`{% html %}` 收品牌专属 | 🔵 未做 | ch15 |
| **P2 本月** | 每页 source 数审查,>20 拆 partials、>10 合 SQL file | 🔵 未做 | ch14 + ch11 |
| **P3 下月** | CF Access (Google Workspace + MFA) + Basic Auth 保留作 defense | 🔵 未做 | ch13 |
| **P3 下月** | 修 13+ page 待修列名 bug 推广 | 🟡 进行中 | ch05 |
| **P3 下月** | Playwright 真渲染(需 user 设 CF Access token) | 🔵 未做 | ch09 |
| **P4 评估期** | `queries/*.sql` 抽 7+ 页复用 + `partials/kpi/` 抽象 | 🔵 未做 | ch12 |
| **P4 评估期** | Pentagram Phase 2 实测(CSS 减负) | 🔵 未做 | ch15 |
| **P4 评估期** | MCP FastMCP 自实现(Claude Code / Cursor) | 🔵 未做 | ch16 |
| **P5 3-6 月** | 团队 ≥5 或报表 >30 触发 BI 工具重评 | 🔵 监控 | ch18 |

## 15. 调研 / 沉淀入口

- **本地 R 系列**: `E:/obsidian/evidence/_research/` (19 R, 7,150+ 行)
- **INDEX**: `E:/obsidian/evidence/_research/INDEX.md` (入口)
- **长期 KB**: `E:/obsidian/_knowledge_base/evidence/2026-10-product-blueprint.md`
- **GBrain recall**:
  - `recall projects/sales-bi` — 项目卡片 + 5 facts
  - `recall concepts/pentagram-editorial` — 3 facts
  - `recall concepts/postgres-fdw` — 1 fact
  - `recall companies/evidence-dev` — 1 fact
  - `search "Evidence Core 2026"` — 跨 page 关键字
- **Skill 本体**: `~/.claude/skills/evidence-skill/` (ch01-ch18, glossary, patterns, cheatsheet)
