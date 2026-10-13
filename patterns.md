# Patterns — Evidence Core 2026 / Sales BI 关键模式

> 5 大模式:Configuration / nginx Patch / Cross-DB FDW / Style Override / Real Verification

## 1. Configuration 模式 — FLAT + STRING + force-recreate

**When to use**: 任何 Core 配置改动(connection.yaml / evidence.config.yaml / theme.yaml / env)

**How**:
```yaml
# connection.yaml — FLAT top-level
type: postgres
host: 124.223.115.201
port: 5432
user: kizemo
password: ...
database: erp
schema: bi
sslmode: disable
```

```yaml
# evidence.config.yaml — STRING version
project:
  name: sales-bi
  evidence: "0.10.1"
```

```bash
# 改完强制 reload
cd /opt/sales-bi/evidence-core
sudo docker compose up -d --force-recreate
```

**Trade-offs**:
- ✓ 简单明确,符合 Core 官方文档
- ✗ 改完**必须 force-recreate**(`restart` 不够,AOT binary 启动时 cache)
- ✗ silent fallback 模式下不会报错,需外部验证 (`curl grep connectionType:postgres`)

## 2. nginx Patch 模式 — 5 处 modify 而非 1 行 sed

**When to use**: 任何 Legacy 40.x → Core 切 prod

**How** (5 处):
```bash
# Patch 1: 5 个 vite dev proxy :3000 → :3002
# Patch 2: location / { try_files ... } REPLACE 整块为 catch-all proxy_pass :3002
# Patch 3: 删除 location ~ "^/_app/" 静态规则
# Patch 4: location ~* \.(js|css|...)$ 加 proxy_pass :3002
# Patch 5: location = / { sub_filter ... } 改 proxy_pass :3002 + 删 sub_filter
```

**Trade-offs**:
- ✓ 兼容 Legacy + Core 静态资源差异
- ✗ 5 处 patch 比 1 行 sed 复杂
- ✗ Patch 2 必须 `m.end()` replace 整块,`m.start()` insert 触发 `duplicate location /`
- ✗ 必须 5 秒回滚脚本预先准备(bash + auto-backup)

## 3. Cross-DB FDW 模式 — 1 Core + postgres_fdw 跨多 DB

**When to use**: 1 个 Core deployment 要读多个 PG database 的表

**How**:
```sql
-- 在主 DB (erp) 上 5 步
CREATE EXTENSION IF NOT EXISTS postgres_fdw;
CREATE SERVER taobao_pg FOREIGN DATA WRAPPER postgres_fdw OPTIONS (
    host '124.223.115.201', port '5432', dbname 'taobao'
);
CREATE USER MAPPING FOR kizemo SERVER taobao_pg OPTIONS (user 'kizemo', password '...');
CREATE SCHEMA IF NOT EXISTS taobao;
CREATE FOREIGN TABLE taobao.mart_shop_kpi_daily (
    "shop_name" text, "stat_date" date, ...
) SERVER taobao_pg OPTIONS (schema_name 'bi', table_name 'mart_shop_kpi_daily');
```

**Trade-offs**:
- ✓ 1 Core deployment 够用(不需 multi-deploy)
- ✓ 跨 DB 透明,Core 不用知道 taobao DB 存在
- ✓ 4 mart 表 142k 行 Core SQL 简单聚合可接受
- ✗ 每 query 走网络,大批聚合需 taobao DB 物化 view
- ✗ 新增 mart 表要重新建 foreign table + 同步 page SQL + force-recreate
- ✗ port 必须 quoted string `'5432'`,不是 integer

## 4. Style Override 模式 — raw hex + article 选择器

**When to use**: Core v0.10.1 上自定义 CSS(取代 theme.yaml var())

**How**:
```css
/* Region A. Hero */
article h1 { color: #0F2A52; }
article .iss-meta { color: #6B6B6B; }
article blockquote, article .callout {
    background: #F5F2EC;
    border-left: 4px solid #C8A658;
}

/* Region B. KPI Strip */
article [class*="block-big-value"] {
    background: #FFFFFF;
    border: 1px solid #E5E2DC;
    color: #0F2A52;
}

/* Region C. Chart Block */
article [class*="chart"], article .chart-container {
    background: #FFFFFF;
    border: 1px solid #E5E2DC;
    padding: 24px;
}

/* Region D. Table Block */
article table { background: #FFFFFF; }
article table thead th { background: #F5F2EC; }
article table tbody tr:nth-child(even) { background: #FAF9F5; }
article table tbody tr:hover { background: #F0F4F8; }

/* Region E. Section Spacing */
article h2 { margin-top: 32px; margin-bottom: 16px; }
article h3 { margin-top: 24px; }
```

**Trade-offs**:
- ✓ raw hex 在 AOT 烘焙后稳定生效
- ✓ 5 region 独立,Phase 2/3 逐报表优化时各 region 独立演进
- ✗ **`var(--name)` 不可用** —— Core 烘焙了 `:root` 块,优先级最高,var() fallback 被覆盖
- ✗ **`.hall-page` class 不存在** —— Core wrap 是 `<article>`,用 article 选择器
- ✗ 改 Core source 后必须重建 image,不在 phase 1 范围

## 5. Real Verification 模式 — 6 步强制 checklist

**When to use**: 任何"修完宣布成功"前(切 prod / 修 bug / 部署新 config)

**How**:
```markdown
□ 1. 直连 PG 跑所有 page SQL(psycopg2 + real DB connection)
□ 2. 真 curl 整页 → Python urllib.real → HTTP status
□ 3. 多 pattern grep (5+ error types in one pass):
    - SQL: does not exist | relation.*does not exist | column.*not exist
    - Conn: unsupported type | connection.yaml:
    - HTML: html-tags-not-allowed | Raw HTML
    - Render: ErrorBadge | error: null vs error: {...}
□ 4. 列字段 + 跑真值查询(不用模板语法)
□ 5. force-recreate Core + 等 5s 才测试
□ 6. user 硬刷新后报告 — 不要假设
```

**Trade-offs**:
- ✓ 防止 3 次 fake success 模式
- ✓ 真验证 = 至少 2 个独立手段互相印证
- ✗ 比"只 grep 1 个 pattern"慢,但 1 次真成功抵 10 次假成功
- ✗ SSR HTML 永远不显示 SQL error(用 placeholder 等 client-side fetch),必须直连 DB

## 反模式(必避)

| 反模式 | 替代 |
|---|---|
| ❌ `docker compose restart` 改 Core config | ✅ `docker compose up -d --force-recreate` |
| ❌ 1 行 sed 切 Core | ✅ 5 处 nginx patch + 5 秒回滚脚本 |
| ❌ `connection.yaml` 写 list-of-dicts | ✅ FLAT top-level |
| ❌ `evidence.config.yaml` 写 object | ✅ STRING version |
| ❌ 中文列名无双引号 | ✅ `"店铺"` `"订单日期"` |
| ❌ `<div class="hall-kpi">` 包裹 component | ✅ `{% row %}...{% /row %}` |
| ❌ `SUBSTR(date_col, 1, 7)` | ✅ `TO_CHAR(date_col, 'YYYY-MM')` |
| ❌ 3-segment path `taobao.bi.mart_X` | ✅ 2-segment `taobao.mart_X` |
| ❌ `var(--name)` CSS 引用 | ✅ raw hex |
| ❌ `.hall-page` 选择器 | ✅ `article` |
| ❌ 只 grep 1 个 error pattern | ✅ 5+ pattern 同时 grep |
| ❌ 改 nginx config 不同步 git | ✅ `live/` + `sync_nginx_config_to_git.sh` |
| ❌ Playwright 不带 CF Access token | ✅ `CLOUDFLARE_ACCESS_TOKEN` env var + helper |
| ❌ CF URL 加 `:8080` | ✅ 不加端口(`https://bi.jury.xin/`) |
| ❌ "修一个发现另一个"继续 | ✅ 第 2 次 fix 后停,做 architecture review |
