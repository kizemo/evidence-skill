# Chapter 5: SQL Anti-patterns — Chinese Columns, Fake Names, date SUBSTR, html-tags, 跨块引用, git-tracked ≠ prod 源

## Core Idea

Core 28 page migration 完成后,所有 SQL 用的列名是**真 PG 列**(`"店铺"` `"订单日期"` `"销售金额"` 中文),但很多 page 写的是 **English 假名**(`shop_name`, `ds`, `stat_date`)。假列在 PG 不存在 → `column "ds" does not exist` 报错。还有 4 个真根因:
1. **`date` 类型不能 `SUBSTR`**(要用 `TO_CHAR`)
2. **raw `<div>` 包裹 component 触发 `html-tags-not-allowed` 替换整块**(要用 `{% row %}`)
3. **跨 SQL block 引用红线 (Phase 7.36)**: 在 Core 0.10.1 DuckDB WASM 架构下,SQL block 之间**没有跨引用机制**, `(SELECT X FROM other_block_name)` 报 `relation "other_block_name" does not exist` (Phase 7.36 sales-overview pnl_waterfall 5 天 broken 根因)
4. **git-tracked ≠ prod 源 (Phase 7.37)**: `evidence/pages/.../sales-overview.md` git-tracked 是 outdated, prod `/opt/sales-bi/evidence-core/pages/.../sales-overview.md` 来自 `deploy/evidence-core-migration/` (untracked) 才是真实源; 改 sales-overview.md 必先 sync 两路径, 否则 scp 会覆盖 prod 的真实现 (Phase 7.37 5b702cf 真瀑布图被 86ad9fe + f176603 退化为 bar_chart 5 天根因)

## Core Idea

Core 28 page migration 完成后,所有 SQL 用的列名是**真 PG 列**(`"店铺"` `"订单日期"` `"销售金额"` 中文),但很多 page 写的是 **English 假名**(`shop_name`, `ds`, `stat_date`)。假列在 PG 不存在 → `column "ds" does not exist` 报错。还有 3 个真根因:
1. **`date` 类型不能 `SUBSTR`**(要用 `TO_CHAR`)
2. **raw `<div>` 包裹 component 触发 `html-tags-not-allowed` 替换整块**(要用 `{% row %}`)
3. **跨 SQL block 引用红线 (Phase 7.36 新增)**: 在 Core 0.10.1 DuckDB WASM 架构下,SQL block 之间**没有跨引用机制**, `(SELECT X FROM other_block_name)` 报 `relation "other_block_name" does not exist` (Phase 7.36 sales-overview pnl_waterfall 5 天 broken 根因)

## Frameworks Introduced

- **中文 PG 列名必须双引号 wrap**:
  - When to use: PG 表含中文列名(常见于中国 SaaS 数据沉淀)
  - How: `SELECT SUM("销售金额") FROM ...` 不要 `SUM(销售金额)`

- **raw 表 vs mart 表列名差异**:
  - When to use: 写跨 raw/mart 表的 SQL
  - How: 查 `information_schema.columns` 确认列名,不要靠 spec 记忆

- **`date` 类型不能 `SUBSTR(date, 1, 7)`**:
  - When to use: 按月聚合 date 列
  - How: `TO_CHAR("订单日期", 'YYYY-MM')` 或 `"订单日期"::text` cast

- **`<div class="hall-kpi">` 触发整块替换**:
  - When to use: 写 KPI 数字块
  - How: 用 Evidence 原生 `{% row %}...{% /row %}`,不要 raw `<div>`

- **跨 SQL block 引用红线 (Phase 7.36)**:
  - When to use: 当一个 SQL block 想用另一个 SQL block 的数据时
  - ❌ WRONG: `(SELECT X FROM other_block_name)` 当 PG table → `relation "other_block_name" does not exist`
  - ✅ CORRECT: WITH CTE inline 全部 logic, 同一个 SQL block 内 CTE 可引用
  - 替代方案: 不存在 (dbt PG view 失效因 date_filter 动态参数; Svelte pass data 40.x 失败 pattern)
  - 关键限制: Core 0.10.1 AOT binary 架构下, SQL block 之间**独立运行**, 只通过 `data="X"` 共享结果给 component, **不能**通过 SQL FROM 共享

- **git-tracked ≠ prod 源 (Phase 7.37)**:
  - When to use: 任何修改 `evidence/pages/.../sales-overview.md` (或类似核心 page) 之前
  - ❌ WRONG: 假设 git-tracked 是 source of truth, scp git-tracked 文件到 prod
  - ✅ CORRECT: 改前先 `ls /opt/sales-bi/evidence-core/pages/.../` 对比 prod 实际状态; 改后 sync 两路径 (git-tracked + untracked deploy/)
  - 实际架构: `evidence/pages/` (git-tracked, Legacy Evidence 40.x build-and-deploy.sh 用) + `deploy/evidence-core-migration/pages/` (untracked, Core AOT bind-mount 直接读) + `/opt/sales-bi/evidence-core/pages/` (prod, 与 untracked 同步) — **3 路径必须 100% 一致**
  - 历史教训: 86ad9fe + f176603 用 git-tracked 简化版 scp 覆盖 prod, 5b702cf 真瀑布图退化为 bar_chart 5 天才被发现

- **Phase 7.x 累积 CTE/过滤不能简化 (Phase 7.37)**:
  - When to use: 改 Phase 7.15+ 的 SQL 时
  - ❌ WRONG: "简化" SQL, 删 CTE, 删 WHERE 过滤, 合并聚合
  - ✅ CORRECT: 保留所有累积的 CTE (daily_store_all), 双引号包裹中文字段 ("店铺"), WHERE "类型" = '真实' 过滤, 等
  - 简化 1 行可能破坏 5 commit 累积的数据完整性 (Phase 7.37 f176603 pnl_kpis 漏 CTE 导致 column 店铺 does not exist)

- **真验证 = 直连 PG + 多 pattern grep + Playwright 真渲染 + prod 实际状态**:
  - When to use: 任何"修完宣布成功"前
  - How: 详见 ch09,5 pattern 同时 grep = 0 + psycopg2 真测 query + Playwright DOM verify (大 chart/table 是否真 render) + **prod `/opt/sales-bi/evidence-core/` 实际状态对比**

## Key Concepts

- **`column "ds" does not exist`**:典型 Chinese schema SQL error,英文假列名
- **`relation "X" does not exist`**:典型**跨 SQL block 引用错**(Phase 7.36), X 是另一个 SQL block 名字, 不是 PG table
- **`"店铺"` `"订单日期"` `"销售金额"`**:real PG column names(双引号是 PG identifier quote)
- **`"订单日期"::text`**:PG cast syntax,把 date 强转 text 让 SUBSTR 可用
- **`TO_CHAR("订单日期", 'YYYY-MM')`**:标准 SQL 格式化 date → string
- **`html-tags-not-allowed`**:Core 编译错,触发后 error 替换整个父级块
- **`{% row %}...{% /row %}`**:Evidence 原生 Row 组件,等价于 flex row layout
- **`.hall-page` class 不存在**:Core HTML wrap 是 `<article>`,不是 `<div class="hall-page">`
- **WITH CTE inline 跨块共享**:Core 0.10.1 DuckDB WASM 唯一 work pattern

## Mental Models

- **列名 = 数据契约**:spec 写的是逻辑列名,真 PG 列名经常不同(尤其是中文)
- **silent compile error 是反模式**:`html-tags-not-allowed` 不阻塞 render,但悄悄替换 component
- **`date` vs `text` 类型差异**:PG 严格类型系统,SQL 函数不自动 cast
- **3 层错误**:`connection.yaml`(配置层) → `column does not exist`(SQL 层) → `html-tags-not-allowed`(组件层),每层独立 grep

## Anti-patterns

- **❌ 写 English 假名**:`ds`、`shop_name`、`stat_date` → PG 报 `column does not exist`
- **❌ 中文列名无双引号**:`SUM(销售金额)` → PG 把"销售金额"当 string literal,聚合函数拿到字符串
- **❌ `SUBSTR(date_col, 1, 7)`**:`function substr(date, integer, integer) does not exist`
- **❌ raw `<div>` 包裹 component**:`<div class="hall-kpi">{% big_value ... %}</div>` → `html-tags-not-allowed` → 替换整块
- **❌ 跨 SQL block 引用 (Phase 7.36)**: `(SELECT X FROM other_block)` 当 PG table → `relation "other_block" does not exist`, 持续 5 天才被发现
- **❌ git-tracked 当 prod 源 (Phase 7.37)**: 假设 `evidence/pages/.../sales-overview.md` git-tracked 是真源, scp 覆盖了 prod `/opt/sales-bi/evidence-core/` 真实 Phase 7.15 真瀑布图, 退化为 bar_chart 5 天才被发现
- **❌ 简化 Phase 7.x 累积 SQL (Phase 7.37)**: 删 daily_store_all CTE 复用 + 删 WHERE "类型" = '真实' 过滤 → 数据完整性破坏, column 店铺 does not exist
- **❌ 单独 grep 1 个 error pattern**:漏报 90% error(详见 ch09)
- **❌ commit 后只验 big_value count, 不验 bar_chart / line_chart / table 实际 render**: Phase 7.36 sales-overview 9 KPI render OK 但 waterfall 仍 broken 5 天就是这个原因
- **❌ 只用 git-tracked 做 controlled experiment**: Phase 7.36 误判 "waterfall pre-existing" 因没对比 prod 真实状态; 必用 prod bind-mount 实际 scp 状态做 pre vs post 对比

## Code Examples

**跨 SQL block 引用 — WITH CTE inline 唯一 work pattern (Phase 7.36)**:

```sql
-- ❌ WRONG: 跨块引用 (25ee79a 错误尝试, 持续 5 天 broken)
```sql pnl_waterfall
SELECT * FROM (VALUES
  ('实际销售额', (SELECT actual_sales FROM pnl_kpis)),  -- pnl_kpis 不是 PG table
  ...
) AS t(step_name, amount)
```
**报错**: `relation "pnl_kpis" does not exist`

```sql
-- ✅ CORRECT: WITH CTE inline 复制 logic
```sql pnl_waterfall
WITH kpi AS (
  SELECT
    ROUND(SUM("销售金额"), 0) AS actual_sales,
    ROUND(SUM("销售金额") - SUM("退货金额"), 0) AS net_sold,
    ...
  FROM (
    SELECT * FROM erp_result.daily_store_dd_2025
    UNION ALL SELECT * FROM erp_result.daily_store_dd_2026
    ...
  ) daily_store_all
  WHERE {{date_filter.filter}} AND {{shop_filter.filter}}
)
SELECT * FROM (VALUES
  ('实际销售额', (SELECT actual_sales FROM kpi)),
  ...
) AS t(step_name, amount)
```
**核心**: 同一个 SQL block 内 CTE 可引用, 跨 block 引用不行
**代价**: pnl_kpis 改动必须同步改 pnl_waterfall 的 CTE (注释标 sync 关系)
**替代方案评估** (Phase 7.36 commit 详):
1. dbt PG view: date_filter 动态参数失效
2. Svelte pass data: 40.x 失败 pattern (`pnl_kpis[0]?.X ?? 0` 不 work)
3. 删 waterfall: 退 Phase 7.30
**结论**: CTE inline 是 Core 0.10.1 唯一 work pattern

**中文列双引号**:
```sql
-- CORRECT
SELECT SUM("销售金额") FROM erp.erp_result.daily_store_dd_2026
WHERE SUBSTR("订单日期"::text, 1, 7) = '2026-01'

-- WRONG (中文无引号,PG 当 string literal)
SELECT SUM(销售金额) FROM erp.erp_result.daily_store_dd_2026
```

**date 列按月聚合**:
```sql
-- CORRECT 1: TO_CHAR
SELECT TO_CHAR("订单日期", 'YYYY-MM') AS month, SUM("销售金额")
FROM erp.erp_result.daily_store_dd_2026
WHERE TO_CHAR("订单日期", 'YYYY-MM') BETWEEN '2025-01' AND '2026-09'
GROUP BY 1

-- CORRECT 2: ::text cast
SELECT SUBSTR("订单日期"::text, 1, 7) AS month, SUM("销售金额")
FROM ...
GROUP BY 1

-- WRONG (date 类型不支持 SUBSTR)
SELECT SUBSTR("订单日期", 1, 7) AS month, SUM("销售金额")
FROM ...
```

**KPI 数字块**:
```markdown
-- CORRECT (Evidence Row)
{% row %}
  {% big_value data="pnl_kpis" value="actual_sales" title="实际销售额" text_size="xl" /%}
  {% big_value data="pnl_kpis" value="net_sold" title="净销售" text_size="xl" /%}
  {% big_value data="pnl_kpis" value="refund" title="退款" text_size="xl" /%}
{% /row %}

-- WRONG (触发 html-tags-not-allowed)
<div class="hall-kpi">
  {% big_value data="pnl_kpis" value="actual_sales" title="实际销售额" text_size="xl" /%}
</div>
```

## y_fmt="pct1" expects raw 0-1 fraction (Phase 7.38 教训)

**陷阱**: SQL 算 percent 经常习惯 `* 100`,但 y_fmt="pct1" 已经自动 × 100, 多乘一次 → 显示 100x 错值 (Phase 7.38 commit 4b85979 销售额显示 10000% 而非 100%)

**Why**: Evidence 0.10.1 y_fmt 设计是 "raw fraction → display percent" (符合 Excel/Google Sheets 习惯: `0.5 * 100% = 50%`), 不是 "raw percent → display percent"

**How to apply**:
- ✅ SQL: `ROUND(val / total, 4) AS amount_pct` (output 0-1)
- ❌ SQL: `ROUND(val / total * 100, 2) AS amount_pct` (output 0-100, y_fmt 再 × 100 → 10000)
- y_fmt="pct1" + amount=0.5 → "50.0%"
- y_fmt="pct1" + amount=1.0 → "100.0%"
- y_fmt="pct1" + amount=100 → "10,000.0%" (错)

**绝对值 fmt (cny0 / usd0k) 不 × 100**:
- y_fmt="cny0" + amount=1234 → "1,234" (千分位, 整数)
- y_fmt="usd0k" + amount=12345 → "$ 12.3K" (自动 K/M)

**Playwright 验证 percent 模式必抓 axis label 文字**, 不能再只查 canvas dataURL (Phase 7.36 baseline 不足):
```python
# 检查 y 轴 labels
y_labels = wc.locator('text').all_text_contents()
pct_labels = [t for t in y_labels if '%' in t]
# 100% 不能变 10000%
assert any('100' in t for t in pct_labels), f"销售额 100% 缺失: {pct_labels}"
```

## 多 pattern grep 验证**:
```bash
# Server-side SSR HTML grep — 必须 0 matches for SQL errors
curl -s 'http://127.0.0.1:8080/<page>' | grep -oE 'does not exist|relation.{0,20}|column.{0,30}|undefined.{0,40}' | grep -v 'undefinedpx' | head -3
# 0 行 = 该 page SQL 干净

# 直连 PG 测 query
PYTHONIOENCODING=utf-8 python3 -c "
import psycopg2
conn = psycopg2.connect(host='124.223.115.201', dbname='erp', user='kizemo', password='...')
cur = conn.cursor()
cur.execute('SELECT SUM(\"销售金额\") FROM erp.erp_result.daily_store_dd_2026 WHERE TO_CHAR(\"订单日期\", \\'YYYY-MM\\') = \\'2026-01\\'')
print(cur.fetchone())
"
```

## Reference Tables

### 假名 → 真名映射

| 假名(代码里写的) | 真名(PG 列) | 出现位置 |
|---|---|---|
| `ds` | `"订单日期"` | sales-overview L49/135/143/156/164/197/209 |
| `shop_name` | `"店铺"` | 13 个 page 出现(mart + raw 都中招) |
| `stat_date` | `"订单日期"` 或 `source_year_month` | shop_* page 系列 |

### raw 表 vs mart 表列名差异

| 表 | 店名列 | 时间列 | 备注 |
|---|---|---|---|
| `erp.erp_result.daily_store_*` | `"店铺"` | `"订单日期"` (date) | raw 日表 |
| `erp.erp_result.daily_sku_*` | `"店铺"` | `"订单日期"` (date) | raw SKU 日表 |
| `bi.mart_sales_monthly` | `"店铺"` | `source_year_month` (str 'YYYY-MM') | 月聚合 |
| `bi.mart_after_sales_monthly` | `"店铺"` | `source_year_month` | 售后月 |
| `bi.mart_shop_kpi_daily` | `"店铺"` | `stat_date` (date) | 店铺日 KPI |
| `bi.mart_customer_asset_daily` | `"店铺"` | `stat_date` (date) | 客户资产日 |

### 错误模式 grep 5 pattern

| Pattern | 检测 | 漏报风险 |
|---|---|---|
| `unsupported type` | connection.yaml 错 | ⚠️ 只匹配 connection,**不**匹配 SQL column |
| `does not exist` | SQL column / relation / **跨块引用** | ✓ |
| `html-tags-not-allowed` | component 包裹错 | ✓ |
| `Error[^A-Za-z]` | 红色 Error badge | ✓ |
| `<ErrorBadge` | ErrorBadge 占位 | ✓ |

### 跨块引用 Playwright 真渲染 baseline (Phase 7.36 防 regression)

每次 commit 修改 `evidence/pages/<group>/<page>.md` 含 SQL block 必跑:
```bash
# Playwright 真浏览器 verify 每个 component 实际 render, 不只数 count
# Phase 7.36 教训: 9 big_value render OK, bar_chart #2 仍 broken
venv/bin/python3 -c "
from playwright.sync_api import sync_playwright
import time
with sync_playwright() as p:
    b = p.chromium.launch(headless=True, args=['--no-sandbox'])
    page = b.new_page()
    page.goto('<page url>', wait_until='domcontentloaded', timeout=60_000)
    time.sleep(25)  # DuckDB WASM cold start
    # 1. chart 是否有 data (canvas dataURL length > 1000 = real data)
    canvas_len = page.locator('canvas').first.evaluate('el => el.toDataURL().length')
    assert canvas_len > 1000, f'canvas empty: {canvas_len}'
    # 2. table row count > 0
    rows = page.locator('table tbody tr').count()
    assert rows > 0, f'table empty'
    # 3. no 'no-data' / 'empty' / 'error' markers
    assert page.locator('text=No data available').count() == 0
    assert page.locator('[class*=\"empty\"]').count() == 0
    b.close()
"
```

**判据**: 所有 chart canvas dataURL > 1000 + 所有 table row > 0 + 0 no-data/empty/error marker + 0 console error

## Key Takeaways

1. **假名 → 真名映射**:`ds`→`"订单日期"`、`shop_name`→`"店铺"`、`stat_date`→`"订单日期"` 或 `source_year_month`
2. **中文列必须双引号** —— PG identifier quote
3. **`date` 不能 SUBSTR** —— 用 `TO_CHAR` 或 `::text` cast
4. **`<div class="hall-kpi">` 是定时炸弹** —— 用 `{% row %}` 替代
5. **跨 SQL block 引用红线 (Phase 7.36)** —— WITH CTE inline 唯一 work, `(SELECT X FROM other_block)` 必报 `relation does not exist`
6. **git-tracked ≠ prod 源 (Phase 7.37)** —— `evidence/pages/` git-tracked 是 outdated, prod `/opt/sales-bi/evidence-core/` 来自 untracked `deploy/evidence-core-migration/`; 必 3 路径 100% 一致
7. **Phase 7.x 累积 SQL 不能简化** —— 保留所有 CTE / 双引号 / WHERE 过滤, 简化 1 行可能破坏 5 commit 累积
8. **真验证不止 grep** —— 必须直连 PG + 5 pattern 同时 grep + **Playwright 真渲染 baseline (每个 chart canvas dataURL > 1000) + prod 实际状态对比**

## Connects To

- **ch04**:Bug A(SUBSTR date)+ Bug B(html-tags-not-allowed)+ **Bug C(跨块引用 25ee79a 引入, Phase 7.36 修)** 出自切 prod 真根因
- **ch07**:shop_* page 用 `taobao.mart_shop_kpi_daily`,列名差异
- **ch09**:3 次 fake success 根因 = 只 grep 1 pattern,**Phase 7.36 加 1 案例 = 只验 big_value count 不验 chart canvas dataURL**
- **ch10**:13+ page 待修列名 fix,下个会话必做
