# Chapter 10: Operations Quick Reference

## Core Idea

Evidence Core 2026 切 prod 后的运维速查:**6 个服务器路径 + 11 个核心 commit + 4 关键经验 + 5 个下个会话必做**。这是从整个项目笔记抽取的"一页纸 cheat sheet",日常切 / 排错 / 续接会话时第一查这个。

## Frameworks Introduced

- **6 个服务器路径速查**:Core workdir / Legacy workdir / nginx live / 切脚本 / systemd timer / backup 链
  - When to use: 任何 Core 运维操作
  - How: 路径不熟先查本章,不要靠记忆

- **11 个核心 commit 链**:docs/core → 9 pages → 6 complex → 6 medium → 6 simple → pilot home → Docker base
  - When to use: 理解 Core migration 进度 / revert / rebase
  - How: `git log --oneline | head -20` 对照

- **4 关键经验(用户必读)**:
  - When to use: 任何新会话开篇,先看这 4 条避免重蹈覆辙
  - How: nginx 4 patch + force-recreate + 中文列双引号 + CF Tunnel 不加端口

- **5 个下个会话必做**:
  - When to use: 续接 Core 维护 / 优化
  - How: 列名 fix 推广 + 多 DB 架构(FDW 已做) + Playwright 真渲染 + 5 秒回滚保留

## Key Concepts

- **`/opt/sales-bi/evidence-core/`**:Core workdir(Docker 挂载)
- **`/opt/sales-bi/evidence/`**:Legacy workdir(切完保留 1-2 周)
- **`/etc/nginx/sites-enabled/sales-bi`**:nginx live config
- **`/opt/sales-bi/deploy/scripts/nginx-cutover/`**:切/回滚/监控脚本
- **`/etc/systemd/system/sales-bi-core-monitor.{service,timer}`**:Core 健康 timer
- **`/etc/nginx/sites-enabled/.bak/sales-bi.legacy.YYYYMMDD-HHMMSS`**:切前 backup
- **CF Tunnel 不加端口**:`https://bi.jury.xin/`(不加 `:8080`),443 上加端口通常不通

## Mental Models

- **Core 路径 ≠ Legacy 路径** —— Core 在 `/opt/sales-bi/evidence-core/`,Legacy 在 `/opt/sales-bi/evidence/`
- **nginx 是 SPOF** —— live config 改前必 backup
- **commit 链反映 migration 阶段** —— 9d99bc5 (goal) → fcda4c2 (Task 11 handoff) → 601ac00 (Bug #3-#5) → 后续 fix
- **下个会话必做 = 优先级清单** —— 13+ page 列名 fix 优先,然后架构 / 验证 / 回滚

## Anti-patterns

- **❌ 改 Core config 不 force-recreate** —— 详见 ch02
- **❌ nginx 1 行 sed 切 Core** —— 4 处 patch,详见 ch06
- **❌ 中文列无双引号** —— PG identifier quote,详见 ch05
- **❌ CF URL 加 `:8080`** —— 公网 443 上端口通常不通
- **❌ 假设"修一个发现另一个"是正常** —— 实际是 fake success 模式,详见 ch09

## Code Examples

**服务器路径速查**:
```bash
# Core workdir
cd /opt/sales-bi/evidence-core

# Legacy workdir(切完保留 1-2 周)
ls /opt/sales-bi/evidence/

# nginx live config
cat /etc/nginx/sites-enabled/sales-bi

# 切/回滚/监控脚本
ls /opt/sales-bi/deploy/scripts/nginx-cutover/
# nginx_switch_to_core.sh
# nginx_switch_to_legacy.sh (5 秒回滚)
# monitor_core.sh (3 维健康检查)

# systemd timer
systemctl list-timers --all | grep sales-bi-core-monitor
journalctl -u sales-bi-core-monitor.service -n 20

# backup 链(切前自动)
ls /etc/nginx/sites-enabled/.bak/ | tail -3
```

**核心 commit 链(main)**:
```bash
git log --oneline | head -20
# 9d99bc5 docs(core): update goal — cut to Core is the objective, not a choice
# fcda4c2 docs(core): Task 11 — Core migration guide + final handoff
# 601ac00 fix(core): Bug #3 #4 #5 — date_filter hardcode
# b8d04a6 fix(core): Task 7 — connection.yaml + evidence.config (WRONG,superseded)
# b86614f feat(core): 9 pages
# 7f0c649 feat(core): 6 complex pages
# 015a2d4 feat(core): 6 medium pages
# c528fc9 feat(core): 6 simple pages
# 3c8c326 feat(core): pilot page home
# d08ddda feat(core): Docker + base
# 2f28213 docs(superpowers/plans)

# 后续 fix(cutover 实战)
# b3bba28 feat(cutover): nginx switch + monitor + corrected handoff
# ebe871f fix(cutover): replace (not insert) catch-all + remove _app/ static rule
# fcb6bda fix(core): evidence.config.yaml project.evidence must be STRING not object
# a141446 fix(core): connection.yaml must be FLAT top-level
# f18a74d fix(core): Bug A SUBSTR(date) → TO_CHAR + Bug B <div class="hall-kpi"> → {% row %}
```

**4 关键经验(用户必读)**:
```markdown
### 1. nginx 切 Core,需要 4 处 patch 而非 1 行 sed
- nginx `root /var/www/evidence` (Legacy 静态 build) — **必须**换
- 5 个 vite-dev proxy `proxy_pass :3000` — 改 `:3002`
- 1 个 `location ~ "^/_app/" { root /var/www/evidence; ... }` — **删除**
- 1 个 `location ~* \.(js|css|...)$` — 加 `proxy_pass :3002`
- 1 个 `location = / { try_files /index.html }` + `sub_filter` — 改 `proxy_pass :3002`
- 1 个 `location / { try_files ... }` — **替换**为 `proxy_pass :3002` 整块

### 2. config 改完必须 `docker compose up -d --force-recreate`
`docker compose restart` 不会重读 bind-mount,**StartedAt 也不变**。Core AOT binary 在进程启动时 cache connections,改文件不通知。

### 3. 中文列名必须双引号
Postgres 用中文列名(如 `"店铺"`, `"订单日期"`, `"销售金额"`)必须 wrap 在双引号里。
SQL 写法: `SELECT SUM("销售金额") FROM erp.erp_result.daily_store_dd_2026 WHERE SUBSTR("订单日期"::text, 1, 7) = '2026-01'`

### 4. CF Tunnel 转发 443 → server:8080
Public URL `https://bi.jury.xin/`(不加端口)→ CF Tunnel → server nginx :8080。**别加 `:8080`**,公网 443 上加端口通常不通。
```

**5 个下个会话必做**:
```markdown
1. 把 `ds`/`shop_name` 等列名 fix 推广到全部 28 page(本会话只改了 sales-overview.md)
2. 多 DB 架构改造:`erp` DB + `taobao` DB → Core 默认连 1 DB,**shop_* / 店铺运营 page 会持续报 connection error**(架构限制)
   — 实际已通过 postgres_fdw 完成,见 ch07
3. Playwright 真渲染验证(目前仅 grep 验证 SSR HTML)
4. 5 秒回滚命令保留:`sudo bash /opt/sales-bi/deploy/scripts/nginx-cutover/nginx_switch_to_legacy.sh`
5. (Phase 2/3) 逐报表优化 + 简报页面 + DingTalk 推送 — 见 ch08 后续阶段
```

## Reference Tables

### 服务器路径速查

| 路径 | 用途 |
|---|---|
| `/opt/sales-bi/evidence-core/` | Core workdir(Docker) |
| `/opt/sales-bi/evidence/` | Legacy workdir(切完保留 1-2 周) |
| `/etc/nginx/sites-enabled/sales-bi` | nginx live(切完 nginx config) |
| `/opt/sales-bi/deploy/scripts/nginx-cutover/` | 切/回滚/监控脚本 |
| `/etc/systemd/system/sales-bi-core-monitor.{service,timer}` | Core 健康 timer |
| `/etc/nginx/sites-enabled/.bak/sales-bi.legacy.YYYYMMDD-HHMMSS` | 切前自动 backup |

### 5 个下个会话必做(优先级)

| 优先级 | 任务 | 状态 |
|---|---|---|
| P0 | 13+ page 列名 fix 推广(sales-overview 已修) | 🟡 进行中 |
| P0 | 5 秒回滚保留 | ✅ 完成 |
| P1 | 多 DB 架构(FDW 已做) | ✅ 完成(ch07) |
| P1 | Playwright 真渲染验证(需 user 设 CF Access token) | 🟡 等待 |
| P2 | Phase 2/3 逐报表优化 + 简报 + DingTalk | 🔵 规划中 |

## Key Takeaways

1. **6 个服务器路径 + 11 commit + 4 经验 + 5 必做** = 一页纸 cheat sheet
2. **新会话开篇**先看 4 关键经验,避免重蹈覆辙
3. **列名 fix 推广是 P0** —— 13+ page 待修,只 sales-overview 已修
4. **FDW 已完成多 DB 架构** —— shop_* page 已恢复
5. **Phase 2/3 是规划中** —— 简报 + DingTalk + 逐报表优化

## Connects To

- **ch01-09**:每章都有速查,但**本章是一页纸汇总**
- **ch04**:5 critical bugs + commit 链
- **ch06**:nginx 4 patch 路径
- **ch09**:真验证 SOP 强制 6 步
