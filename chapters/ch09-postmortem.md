# Chapter 9: Postmortem — 3 False Successes + 4 Protocols

## Core Idea

切 prod 时序复盘:**3 次"声称成功但用户硬刷仍报错"** + 1 次真成功。根因不是某次 fix 写错,而是**每次 fix 只看了自己改的那层,没看用户实际看到的层**。改进:claim 完成前必做 4 协议(直连 DB + 多 pattern grep + 列字段真值 + force-recreate 5s)。

## Frameworks Introduced

- **真验证 4 协议**:
  - When to use: 任何"修完宣布成功"前
  - How: ① 直连 DB 跑 page SQL ② Python urllib 真 GET HTTP ③ 多 pattern grep(5+ error types)④ 列字段 + 真值查询

- **多 pattern grep 一次够**:
  - When to use: 任何 SSR HTML 验证
  - How: 5+ pattern 同时 grep:SQL `does not exist`、Conn `unsupported type`、HTML `html-tags-not-allowed`、Runtime `ErrorBadge`

- **memory 不全信**:
  - When to use: 引用过去的 memory 决策
  - How: 必须标 "verified 在 page X 真不阻塞,不是全局 conclusion"

- **架构问题 → 提早 question architecture**:
  - When to use: 多次"修一个发现另一个"模式
  - How: 第 2 次 fix 后停下,做 `git log` + page 全量审计 + architecture review

## Key Concepts

- **fake success**:声称完成 + 自我感觉 OK + 验证手段只覆盖自己改的层,但用户实际看到的层还有 bug
- **silent compile error**:`html-tags-not-allowed` 不阻塞 render,但悄悄替换 component
- **SSR HTML 永远不显示 SQL column error**:Core 用 `<!--[-1-->` placeholder 等 client-side fetch
- **client-side hydration 才是 SQL 错误第一显示点**:浏览器 fetch 后注入 DOM
- **direct DB query 必要性**:HTML grep 是必要的,但单独 grep 是 insufficient
- **3 false success 模式**:
  1. `fcb6bda` — 改 `evidence.config.yaml STRING`,验 docker logs 无 error → 漏 SQL error
  2. `a141446` — 改 `connection.yaml FLAT`,验 `connectionType:postgres` → 漏 `pnl_kpis` 真查询
  3. `f1ea24e` — 改 `ds` → `订单日期`,验 `does not exist` 0 命中 → 漏 html-tags-not-allowed + date SUBSTR

## Mental Models

- **3 层错误独立 grep**:
  1. **Connection layer**:`unsupported type` / `connection.yaml:`
  2. **SQL layer**:`does not exist` / `relation.*does not exist` / `column.*not exist`
  3. **Component layer**:`html-tags-not-allowed` / `Raw HTML`
  4. **Runtime layer**:`Error[^A-Za-z]` / `ErrorBadge` / `error: null`
- **SSR HTML 不可信**:SSR 用 placeholder,真实错误在 client-side hydration 后才显示
- **memory 可能错**:写 memory 时标"已接受 limitation"要小心,可能实际真阻塞
- **真成功 = 多手段互相印证**:至少 2 个独立验证手段

## Anti-patterns

- **❌ 只 grep 单一 error pattern**(如只 grep `unsupported type`):漏 90% error
- **❌ 看 `docker logs` 就宣布 OK**:Core silent fallback 不报 SQL error
- **❌ 看 `connectionType:postgres` 1 处匹配就宣布 OK**:只验证 Core 读到 connection.yaml,没验证 query 真执行
- **❌ memory 写"已接受 limitation"**:实际可能真阻塞,必须 with 验证 evidence
- **❌ "修一个发现另一个"模式继续**:第 2 次 fix 后必须停下做 architecture review

## Code Examples

**改进的验证 SOP(6 步强制 checklist)**:
```markdown
□ 直接 PG 跑所有 page SQL(psycopg2 + real DB connection)
□ 真 curl 整页 → Python urllib.real → HTTP status
□ 多 pattern grep:
    - SQL: does not exist | relation.*does not exist | column.*not exist
    - Conn: unsupported type | connection.yaml:
    - HTML: html-tags-not-allowed | Raw HTML
    - Render: ErrorBadge | error: null vs error: {...}
□ 列字段 + 跑真值查询(不用模板语法)
□ force-recreate Core + 等 5s 才测试
□ user 硬刷新后报告 — 不要假设
```

**真验证(直接 PG 测 query)**:
```python
import psycopg2
import urllib.request

# 1. 直连 PG 跑所有 page SQL
conn = psycopg2.connect(host='124.223.115.201', dbname='erp', user='kizemo', password='...')
cur = conn.cursor()
cur.execute("""
    SELECT SUM("销售金额") AS actual_sales
    FROM erp.erp_result.daily_store_dd_2026
    WHERE TO_CHAR("订单日期", 'YYYY-MM') = '2026-01'
""")
print('actual_sales:', cur.fetchone())
# ✓ (98264901,)

# 2. 真 curl 整页
req = urllib.request.Request('http://127.0.0.1:8080/大盘/sales-overview')
with urllib.request.urlopen(req) as resp:
    html = resp.read().decode('utf-8')

# 3. 多 pattern grep
import re
patterns = {
    'SQL': r'does not exist|relation.*does not exist|column.*not exist',
    'Conn': r'unsupported type|connection.yaml:',
    'HTML': r'html-tags-not-allowed|Raw HTML',
    'Render': r'ErrorBadge|error: null',
}
for label, pat in patterns.items():
    matches = re.findall(pat, html)
    if matches:
        print(f'FAIL {label}: {len(matches)} matches')
    else:
        print(f'PASS {label}: 0 matches')
```

**3 次 fake success 复盘**:

| 提交 | 当时验证 | 漏了什么 |
|---|---|---|
| `fcb6bda` evidence.config.yaml STRING | `docker logs` 无 error | Core silent fallback,**不报 SQL error** |
| `a141446` FLAT top-level | `grep "connectionType:postgres"` 1 处 | 漏 `pnl_kpis` query 真实执行结果 |
| `f1ea24e` ds → 订单日期 | `grep "does not exist"` 空 | 漏 Bug A `SUBSTR(date)` + Bug B html-tags-not-allowed |

**第 4 次真成功(`f18a74d`)**:
- 修法: `SUBSTR(date) → TO_CHAR` + `<div class="hall-kpi"> → {% row %}`
- 验证: 直接 PG 测 + Python urllib 真 GET + 5+ error pattern grep
- 验证矩阵:
  ```
  unsupported type       0
  connection.yaml        0
  does not exist         0
  html-tags-not-allowed  0
  Error[^A-Za-z]         0
  ErrorBadge             0
  直接 PG 测试 pnl_kpis:   ¥98M, ¥66M, ¥26M, ¥26M
  ```

## Reference Tables

### 协议改进对比

| 之前 | 现在 |
|---|---|
| grep `unsupported type` 1 pattern | grep 5+ error patterns in one pass |
| 只看 server-side HTTP code | HTTP + grep + 直接 DB 测 + column 真存在性 |
| 一发现错误立刻改 | 一发现错误**先列 bug catalog**,按依赖序改 |
| 改后自我感觉"完成" | 改后**对照 fix 前的真实错误模式**,用同一工具确认 0 出现 |
| claim 成功 → 真验证 | claim 完成前**至少 2 个独立验证手段**,互相印证 |

### 我没意识到的 3 个事实(关键 reflection)

| 事实 | 误解 | 真相 |
|---|---|---|
| A. grep clean 是虚假 clean | 0 = OK | SSR HTML 不含 client-side error,只看 grep = 假 clean |
| B. memory 可能错 | "Limitation 2 已接受" | 实际真阻塞(替换整块包括 component) |
| C. 多次"修一个发现另一个" | 继续 fix | 必须停,做 architecture review |

## Key Takeaways

1. **3 次 fake success 根因**:每次只看自己改的那层,不看用户实际看到的层
2. **真验证 4 协议**:直连 DB + 真 GET + 多 pattern grep + 列字段真值
3. **memory 不全信**:写 "已接受 limitation" 要 with 验证 evidence
4. **架构问题提早 question**:`修一个发现另一个` 模式必须停,做 architecture review
5. **5 pattern 同时 grep**:SQL / Conn / HTML / Render,任一非 0 都不算完成

## Connects To

- **ch04**:5 critical bugs 切 prod 时序复盘
- **ch05**:Bug A (SUBSTR) + Bug B (html-tags) 真根因
- **ch07**:反思 "我之前说不可能" —— 跟 fake success 同根
- **ch10**:下个会话必做 + 5 秒回滚保留
