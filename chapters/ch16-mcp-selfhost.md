# Chapter 16: MCP for Self-Host (R17 — FastMCP 自实现 + 8 tools + 4 道 SQL 防线)

## Core Idea

**Studio MCP 是 SaaS-only** (14 tools at `evidence.studio/api/mcp`),**Self-host Core 无 MCP 端点** — `evidence serve` 是 plain HTTP,无 `/api/mcp`。要让 Claude Code / Cursor / Continue / ChatGPT 用本地 Evidence 报表,必须**自写 MCP middleware**(Python FastMCP 推荐)。

## 4 步实施路径

1. **Python `mcp` 2.3.0** (FastMCP 已合入官方 SDK)
2. **独立 `fastmcp` PyPI 4.1.0** (1M 下载/天,承载 70% MCP server,更简单)
3. **TS SDK v2** `@modelcontextprotocol/server` + `/client`
4. **官方 Postgres reference server** (archived) — 1 tool `query` + 表 schema resource + READ ONLY 事务

## 8 个自托管 MCP tools 设计

| Tool | 描述 | 数据源 |
|---|---|---|
| `list_pages` | 列出所有 Evidence pages (title/path/last_modified) | evidence/pages/ dir |
| `get_page_source` | 返回 page markdown 源码 | .md 文件 |
| `get_page_rendered` | 返回渲染 HTML 或截图 (Playwright) | nginx + screenshot |
| `run_query` | 执行 SQL (READ ONLY 角色, 4 道防线) | PG via psycopg2 |
| `list_sources` | 列出所有 source files + SQL | evidence/sources/ dir |
| `search_content` | 跨 page/source 搜索 | grep / FTS |
| `get_connection_info` | 返回 connection.yaml 摘要(无密码) | yaml.safe_load |
| `visualize` | run_query + 格式化数据,适配 chart input | 同 run_query |

## 4 道 SQL 安全防线 (任一被攻破另几层兜底)

```
[1] PG role = readonly (CREATE/INSERT/UPDATE/DELETE/DROP 全部被拒)
[2] sqlglot AST 解析 (识别 DROP/TRUNCATE/DELETE FROM 等危险语句)
[3] SET LOCAL statement_timeout = '10s' (防止慢查询拖死 DB)
[4] LIMIT 10000 (默认上限, 强制 require_explicit_higher_limit)
```

## Sample Code 骨架 (FastMCP 4.1.0)

```python
# evidence_mcp/server.py
from fastmcp import FastMCP
import psycopg2
import sqlglot
import yaml, os, logging
from pathlib import Path

mcp = FastMCP("evidence-local")

# 1. list_pages
@mcp.tool()
def list_pages(prefix: str = "pages/") -> list[dict]:
    """List all Evidence pages."""
    pages = []
    for p in Path("/opt/sales-bi/evidence-core").rglob(f"{prefix}*.md"):
        pages.append({"title": p.stem, "path": str(p.relative_to("/opt/sales-bi/evidence-core"))})
    return pages

# 2. run_query (4 道防线)
@mcp.tool()
def run_query(sql: str) -> list[dict]:
    """Execute SQL against evidence PG (READ ONLY, 4 defenses)."""
    # [2] AST 解析
    parsed = sqlglot.parse_one(sql)
    if any(node.key.upper() in ("DROP", "TRUNCATE", "DELETE") for node in parsed.walk()):
        raise ValueError("Destructive SQL blocked: DROP/TRUNCATE/DELETE")
    # [1] readonly role via PG DSN
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
    conn.close()

# 3. get_connection_info (永不含 password)
@mcp.tool()
def get_connection_info() -> dict:
    """Return connection.yaml summary (no secrets)."""
    cfg = yaml.safe_load(open("/opt/sales-bi/evidence-core/connection.yaml"))
    return {k: v for k, v in cfg.items() if "password" not in k.lower()}

# ... 其他 5 tools 略
```

## 4 种客户端集成模式

```bash
# Claude Code (stdio)
claude mcp add evidence-local --transport stdio python -m evidence_mcp

# Cursor (SSE)
# .cursor/mcp.json
{"mcpServers": {"evidence-local": {"url": "https://mcp.bi.jury.xin/sse"}}}

# Continue.dev (HTTP)
# ~/.continue/config.json
{"mcp": [{"name": "evidence-local", "url": "https://mcp.bi.jury.xin/mcp"}]}

# ChatGPT (HTTP + Bearer)
# /v1/mcp/connect?server=https://mcp.bi.jury.xin/mcp
```

## 部署 (systemd + nginx)

```ini
# /etc/systemd/system/evidence-mcp.service
[Service]
ExecStart=/opt/sales-bi/.venv/bin/python -m evidence_mcp --port 8081
Environment=EVIDENCE_READONLY_DSN=postgresql://readonly:***@127.0.0.1/erp
Environment=PYTHONUNBUFFERED=1
Restart=on-failure
User=www-data
```

```nginx
# nginx conf 段
location /mcp/sse {
    proxy_pass http://127.0.0.1:8081;
    proxy_http_version 1.1;
    proxy_set_header Connection '';
    proxy_buffering off;
}
```

## 监控 + 审计

- 全部 tool calls 写 journald + Loki
- `run_query` 每次记 SQL + duration + row count
- 速率限制: `MCP_RATE_LIMIT=60 calls/min` per connection
- 错误注入测试: 攻击者尝试 `DROP TABLE` → [2] AST 拦
- 慢查询: `SET LOCAL statement_timeout` → [3] 兜底

## Anti-patterns (MCP self-host)

- ❌ 用 superuser 角色做 `run_query` → 必 readonly role
- ❌ 暴露 password / secrets in `get_connection_info` → yaml.safe_load + 字段黑名单
- ❌ `EVIDENCE_TELEMETRY_DISABLED` 未设 → MCP server 也吃 telemetry 心跳
- ❌ 无速率限制 → 攻击者 1 分钟 10000 次 `run_query` 拖死 PG
- ❌ 用 `evidence serve` 暴露 `/api/mcp` → Core 无此端点,必须独立 FastMCP 服务

## Connects To

- **ch13** — auth 拦截 MCP 调用 (Bearer token)
- **ch14** — `run_query` 复用 `statement_timeout=30s` 规则
- **ch10** — 监控 MCP 调用日志到 journald
