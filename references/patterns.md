# Patterns — Evidence BI 关键模式 (8 大模式)

> ch01-ch10 实战 (5 大模式) + ch11-ch18 R 系列 (3 大模式)

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

## 4. Style Override 模式 — `theme.yaml` 5 层 + `{% html %}` block

**When to use**: Core v0.10.1+ 自定义 CSS (R12 推翻 Phase 1 旧路径)

**How**:
```yaml
# theme.yaml (项目根目录, 70% Pentagram 视觉原生)
colors:
  base: "#FFF1E5"
  card: "#FFFFFF"
  positive: "#0F5499"
  negative: "#990F3D"
fonts:
  heading: serif
radius: 8px
depth: subtle
density: comfortable
chart:
  barRadius: 4
table:
  rowLines: true
  hoverColor: "#F0F4F8"
  zebra: true
```

```markdown
<!-- page .md (30% 品牌专属细节) -->
{% html %}
<style>
  article h1 { border-bottom: 1px solid var(--color-card-border); }
  blockquote { border-left: 4px solid #C8A658; }
</style>
{% /html %}
```

**Trade-offs**:
- ✓ 70% Pentagram 视觉走 theme.yaml 原生,Phase 1 sales-bi.css 从 ~300 行缩到 ~80 行
- ✓ 5 层 inheritance (default → org → project → page frontmatter → component) 真有效
- ✗ **`var(--name)` 不可用** → Core 烘焙了 `:root` 块,优先级最高,var() fallback 被覆盖
- ✗ **`.hall-page` class 不存在** → Core wrap 是 `<article>`,用 article 选择器
- ✗ 改 theme.yaml 必 `force-recreate` 才生效

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
    - Cross-DB: cross-database references are not implemented
□ 4. 列字段 + 跑真值查询(不用模板语法)
□ 5. force-recreate Core + 等 5s 才测试
□ 6. user 硬刷新后报告 — 不要假设
```

**Trade-offs**:
- ✓ 防止 3 次 fake success 模式
- ✓ 真验证 = 至少 2 个独立手段互相印证
- ✗ 比"只 grep 1 个 pattern"慢,但 1 次真成功抵 10 次假成功
- ✗ SSR HTML 永远不显示 SQL error(用 placeholder 等 client-side fetch),必须直连 DB

## 6. Performance Tuning 模式 — 4 阶段 actionable (R11)

**When to use**: Core self-host 性能优化,28 page + FDW 场景

**How** (4 阶段):
```bash
# P1 (立刻 <4h)
# nginx 静态资源 immutable
location /_app/immutable/ {
    add_header Cache-Control "public, max-age=31536000, immutable";
    proxy_pass http://127.0.0.1:3002;
}

# P2 (本周)
psql -c "ALTER ROLE kizemo SET statement_timeout = '30s'";
psql -c "CREATE EXTENSION IF NOT EXISTS pg_stat_statements";

# P3 (下周)
# 每页 source 数审查 < 20,合 SQL file
# matview for daily_store_summary
# B-tree index ("订单日期", "店铺")

# P4 (并发 > 100)
# 横向 Evidence 实例 + PG read replica
# random_page_cost = 1.1 (SSD)
# effective_io_concurrency = 200
```

**Trade-offs**:
- ✓ PG 端优化是 Core 性能唯一杠杆(应用层无 cache)
- ✓ pgbouncer session-mode 兼容 search_path + statement_timeout
- ✗ Direct Connector 无应用层 cache(除了 BQ/Snowflake Sync 1-min)
- ✗ Models 是 Studio-only Beta,Self-host 不能用
- ✗ FDW 跨 DB 性能差,高频 JOIN 需预聚合本地

## 7. Self-host MCP 模式 — FastMCP + 4 道 SQL 防线 (R17)

**When to use**: Self-host Core 用户想让 Claude Code / Cursor / ChatGPT 调本地 Evidence

**How**:
```python
# evidence_mcp/server.py
from fastmcp import FastMCP
import psycopg2, sqlglot, yaml, os
from pathlib import Path

mcp = FastMCP("evidence-local")

@mcp.tool()
def list_pages(prefix: str = "pages/") -> list[dict]:
    return [{"title": p.stem, "path": str(p.relative_to("/opt/sales-bi/evidence-core"))}
            for p in Path("/opt/sales-bi/evidence-core").rglob(f"{prefix}*.md")]

@mcp.tool()
def run_query(sql: str) -> list[dict]:
    # [2] AST 解析
    parsed = sqlglot.parse_one(sql)
    if any(n.key.upper() in ("DROP","TRUNCATE","DELETE") for n in parsed.walk()):
        raise ValueError("Destructive SQL blocked")
    # [1] readonly role
    conn = psycopg2.connect(dsn=os.environ["EVIDENCE_READONLY_DSN"])
    # [3] statement_timeout
    with conn.cursor() as cur:
        cur.execute("SET LOCAL statement_timeout = '10s'")
        # [4] LIMIT clamp
        if "LIMIT" not in sql.upper():
            sql += " LIMIT 10000"
        cur.execute(sql)
        cols = [d[0] for d in cur.description]
        return [dict(zip(cols, row)) for row in cur.fetchall()]

@mcp.tool()
def get_connection_info() -> dict:
    cfg = yaml.safe_load(open("/opt/sales-bi/evidence-core/connection.yaml"))
    return {k: v for k, v in cfg.items() if "password" not in k.lower()}
```

**Trade-offs**:
- ✓ Self-host 无 MCP endpoint,自实现是唯一路径
- ✓ 4 道防线 (readonly role + sqlglot AST + statement_timeout + LIMIT) 互相兜底
- ✗ `get_connection_info` 必 strip password
- ✗ 必 systemd + Bearer token 鉴权
- ✗ `evidence serve` 不暴露 `/api/mcp`,必须独立 FastMCP 服务

## 8. Upgrade & Rollback 模式 — 7 步 SOP + 5 分钟回滚 (R20)

**When to use**: Core 2026 升级(无 SemVer, commit-based)

**How**:
```bash
# Step 1-3: 读 changelog + capture baseline + 备份
docs.evidence.dev/changelog/  # 搜 BREAKING + MIGRATE
evidence version > /var/log/evidence-baseline-20261014.txt
git -C /opt/sales-bi/evidence-core commit -am "pre-upgrade-snapshot-20261014"

# Step 4-5: pull + force-recreate
cd /opt/sales-bi/evidence-core
sed -i 's/v0.10.1/v0.10.2/' docker-compose.yaml
sudo docker compose up -d --force-recreate

# Step 6: smoke test
sleep 5
curl -s :3002/ | grep -oE 'connectionType:"postgres"'  # 应见

# Step 7: 24h 监控
journalctl -u sales-bi-core-monitor.service -n 50

# --- 5 分钟回滚 ---
sed -i 's/v0.10.2/v0.10.1/' docker-compose.yaml
sudo docker compose up -d --force-recreate  # 5min 内完成
```

**Trade-offs**:
- ✓ Core 无 SemVer,commit sha 是唯一内部锚点
- ✓ 14 天 hold-back window: 升级前等 2 周观察别人踩的坑
- ✗ `evidence upgrade` 在 CI/服务器必 `upgrade-disabled` 锁版本
- ✗ 升级后 24h 观察期不立刻 commit Pages 改动
- ✗ 任何 config 改动都需 force-recreate,restart 不够

## 反模式总表 (必避)

| 反模式 | 替代 |
|---|---|
| ❌ `docker compose restart` 改 Core config | ✅ `docker compose up -d --force-recreate` |
| ❌ 1 行 sed 切 Core | ✅ 5 处 nginx patch + 5 秒回滚脚本 |
| ❌ `connection.yaml` 写 list-of-dicts | ✅ FLAT top-level |
| ❌ `evidence.config.yaml` 写 object | ✅ STRING version |
| ❌ 中文列名无双引号 | ✅ `"店铺"` `"订单日期"` |
| ❌ `<div class="hall-kpi">` 包裹 component | ✅ `{% row %}...{% /row %}` |
| ❌ raw `<div>` / `<style>` markdown body | ✅ `{% html %}<block>{% /html %}` |
| ❌ `SUBSTR(date_col, 1, 7)` | ✅ `TO_CHAR(date_col, 'YYYY-MM')` |
| ❌ 3-segment path `taobao.bi.mart_X` | ✅ 2-segment `taobao.mart_X` |
| ❌ `var(--name)` CSS 引用 | ✅ `theme.yaml` 字段 + raw hex |
| ❌ `.hall-page` 选择器 | ✅ `article` |
| ❌ 只 grep 1 个 error pattern | ✅ 5+ pattern 同时 grep |
| ❌ 改 nginx config 不同步 git | ✅ `live/` + `sync_nginx_config_to_git.sh` |
| ❌ Playwright 不带 CF Access token | ✅ `CLOUDFLARE_ACCESS_TOKEN` env var + helper |
| ❌ CF URL 加 `:8080` | ✅ 不加端口(`https://bi.jury.xin/`) |
| ❌ "修一个发现另一个"继续 | ✅ 第 2 次 fix 后停,做 architecture review |
| ❌ Self-host 用 `{{ env.X_FORWARDED_USER }}` | ✅ Core 不读反代 header,改 Basic Auth / CF Access |
| ❌ `pgbouncer transaction-mode` | ✅ session-mode |
| ❌ Self-host `run_query` 用 superuser role | ✅ readonly role + 4 道防线 |
| ❌ `evidence upgrade` 在生产服务器 | ✅ `upgrade-disabled` 锁版本 |
| ❌ 升级前不 capture `evidence version` | ✅ commit sha 入库作 baseline |
| ❌ 升级后立刻 commit Pages 改动 | ✅ 24h 观察期 |
