# Chapter 5: SQL Anti-patterns — Chinese Columns, Fake Names, date SUBSTR, html-tags

## Core Idea

Core 28 page migration 完成后,所有 SQL 用的列名是**真 PG 列**(`"店铺"` `"订单日期"` `"销售金额"` 中文),但很多 page 写的是 **English 假名**(`shop_name`, `ds`, `stat_date`)。假列在 PG 不存在 → `column "ds" does not exist` 报错。还有 2 个真根因:**`date` 类型不能 `SUBSTR`**(要用 `TO_CHAR`)、**raw `<div>` 包裹 component 触发 `html-tags-not-allowed` 替换整块**(要用 `{% row %}`)。

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

- **真验证 = 直连 PG + 多 pattern grep**:
  - When to use: 任何"修完宣布成功"前
  - How: 详见 ch09,5 pattern 同时 grep = 0 + psycopg2 真测 query

## Key Concepts

- **`column "ds" does not exist`**:典型 Chinese schema SQL error,英文假列名
- **`"店铺"` `"订单日期"` `"销售金额"`**:real PG column names(双引号是 PG identifier quote)
- **`"订单日期"::text`**:PG cast syntax,把 date 强转 text 让 SUBSTR 可用
- **`TO_CHAR("订单日期", 'YYYY-MM')`**:标准 SQL 格式化 date → string
- **`html-tags-not-allowed`**:Core 编译错,触发后 error 替换整个父级块
- **`{% row %}...{% /row %}`**:Evidence 原生 Row 组件,等价于 flex row layout
- **`.hall-page` class 不存在**:Core HTML wrap 是 `<article>`,不是 `<div class="hall-page">`

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
- **❌ 单独 grep 1 个 error pattern**:漏报 90% error(详见 ch09)

## Code Examples

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

**多 pattern grep 验证**:
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
| `does not exist` | SQL column / relation | ✓ |
| `html-tags-not-allowed` | component 包裹错 | ✓ |
| `Error[^A-Za-z]` | 红色 Error badge | ✓ |
| `<ErrorBadge` | ErrorBadge 占位 | ✓ |

## Key Takeaways

1. **假名 → 真名映射**:`ds`→`"订单日期"`、`shop_name`→`"店铺"`、`stat_date`→`"订单日期"` 或 `source_year_month`
2. **中文列必须双引号** —— PG identifier quote
3. **`date` 不能 SUBSTR** —— 用 `TO_CHAR` 或 `::text` cast
4. **`<div class="hall-kpi">` 是定时炸弹** —— 用 `{% row %}` 替代
5. **真验证不止 grep** —— 必须直连 PG + 5 pattern 同时 grep

## Connects To

- **ch04**:Bug A(SUBSTR date)+ Bug B(html-tags-not-allowed)出自切 prod 真根因
- **ch07**:shop_* page 用 `taobao.mart_shop_kpi_daily`,列名差异
- **ch09**:3 次 fake success 根因 = 只 grep 1 pattern
- **ch10**:13+ page 待修列名 fix,下个会话必做
