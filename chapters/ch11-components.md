# Chapter 11: Components Reference (R3 — 7 大类 80+ 组件)

## Core Idea

Evidence 用 `{% name attr=val / %}` 模板语法调用组件,7 大类 (Values / Tables / Charts / Maps / Inputs / Custom UI / Logic) 覆盖 80+ 组件;命名 snake_case,属性类型化,IDE 实时校验。

## Frameworks

- **Markup 模板语法**: `{% name attr=val / %}` 自闭合 vs `{% name %}content{% /name %}` 含内容两种;`/ `触发自动补全
- **变量**: `{{ var }}` / `{{ var.attr }}` / `{{ var.method(...) }}`;`{{ $user.x }}` Studio 内置
- **7 大组件类**: Values(5+) / Tables(3) / Charts(20+) / Maps(3) / Inputs(6) / Custom UI(5+) / Logic(3)
- **属性类型化**: String / Number / Boolean / Object (`x_axis_options={title='Date'}`);内部双引号用 `\"` 适配 Snowflake 含空格列

## Key Components (最常用)

- **big_value** — 大号 KPI 数字;属性 `title`, `value`, `fmt`, `comparison`
- **value** — 通用文本/数值;`fmt='usd'` / `'pct'` / `'#,##0'`
- **delta** — 同比/环比;`value`, `comparison`, `down_is_good=true`
- **table / data_table / grouped_table** — 交互式表格;`DataTable` 是 v0.10+ 推荐 API
- **bar / line / area / scatter / pie / funnel** — 6 基础图表;data + x + y 三必填
- **heatmap / box_plot / waterfall / sankey / radar / candlestick / chord / combo / treemap** — 9 高级图表
- **dropdown / checkbox / range_calendar / text_input / button / slider** — 6 输入
- **`{% if %} / {% repeat %} / {% let %}`** — 逻辑控制

## Anti-patterns (Components 雷区)

- ❌ URL 路径用 kebab-case `/components/scatter-chart` → 全 404,**用 snake_case** `/components/scatter_chart`
- ❌ `/components/checkbox` → **是 `{% toggle %}`**(Mintlify URL vs 标签错位)
- ❌ `/components/date_picker` → 404,**用 `{% range_calendar %}`**(唯一 date picker)
- ❌ `/components/box_plot` / `/components/waterfall_chart` → 404,**用 `{% custom_echart %}`**
- ❌ `format_titles=true`(默认) + Chinese 表头 → **Title Case 破坏中文别名**,**改 `format_titles=false`**
- ❌ 同一 page 引用 `data=` 同名 query → validators 误报(2026-10-08 修)
- ❌ `combo_chart` 内 `echarts_options` 多 key 用 `,` 分隔 → Markdoc parser 报 `Expected "}", identifier, string, or whitespace but "," found`; 必 newline 分隔 (Phase 7.43 sales-overview)
- ❌ `combo_chart` echarts_options 用 `:` (ECharts 风格) → Markdoc parser 把 `:` 当 tag 解析, 报 `Node 'tag' is missing opening`; 用 `=` (Evidence 风格, Phase 7.42)

## `{% combo_chart %}` 模式 (Phase 7.40-7.45 sales-overview 沉淀)

**用途**: 1 个 chart 含 2+ series 不同 type (bar + line / 2 bar + 1 line / 1 bar + 2 line 等), 双 Y 轴单位

### 基本结构 (Evidence Core 0.10.1 native)

```markdown
{% combo_chart
    data="my_sql"
    x="date_col"
    y_fmt="cny0"           {# 主 Y 轴 fmt, 金额/整数/百分比 #}
    y2_fmt="pct1"          {# 副 Y 轴 fmt, 百分比 #}
    title="..."
%}
    {% bar y="amount_col" /%}                {# 主 Y 轴 #}
    {% line y="pct_col" axis="y2" /%}        {# 副 Y 轴, axis="y2" 必加 #}
    {% line y="pct_col2" axis="y2" /%}      {# 副 Y 轴可多条 #}
{% /combo_chart %}
```

**ECharts 标准** (ch11 文档):
- `axis="y1"` (主) / `axis="y2"` (副)
- `stacked=true` 默认 → 2 bar 会堆叠; **`stacked=false` 必加** 让 bar 独立起算
- `y_fmt` / `y2_fmt` 控制各 axis 数字格式 (cny0 / pct1 / usd0k)

### 4 大必避 (Phase 7.41-7.43 实证)

1. **内 `{% bar title="..." %}` 不支持** → "Invalid attribute: 'title'"; 改用 `echarts_options={ name="..." }` 覆盖 legend
2. **`echarts_options={ barGap: '-50%' }` (ECharts `:` 风格)** → "Node 'tag' is missing opening"; 改 `barGap="-50%"` (`=` Evidence 风格)
3. **`echarts_options={ name="X", barWidth="70%" }` (` ,` 分隔多 key)** → "Expected '}', identifier, string, or whitespace but ',' found"; 多 key 必 **newline 分隔**
4. **不写 `stacked=false`** → 2 bar 默认堆叠, 净实销在 1000-1800 (从实际销售额顶部起)

### 2 bar 重叠模式 (Phase 7.43 完全重叠)

```markdown
{% combo_chart
    echarts_options={
        barGap="-100%"           {# 完全重叠 = 100% 负 gap #}
        barCategoryGap="20%"      {# 月份间 20% gap #}
    }
%}
    {% bar y="big" stacked=false echarts_options={
        name="大值背景"
        barWidth="70%"            {# 大值 70% 宽, 背景 #}
    } /%}
    {% bar y="small" stacked=false echarts_options={
        name="小值前景"
        barWidth="50%"            {# 小值 50% 宽, 在前 (覆盖上层) #}
    } /%}
{% /combo_chart %}
```

**重叠原理**:
- `stacked=false`: 2 series 独立起算 (不堆叠, 各自从 0 起)
- `barGap="-100%"`: 同 x category 内 2 bar 完全重叠
- `barWidth="70%"` / `"50%"`: 大小控制视觉差
- **声明顺序**: 后面声明的 series 默认后绘制 = 在前 (覆盖上层)
- **效果**: 大值宽色底 + 小值窄色顶, 同一 x 位置, 视觉层级清晰

### 中文 legend 自动 (Phase 7.41)

`{% bar y="actual_sales_real" %}` → legend 显示 `"actual_sales_real"` (y 列名)

用 `echarts_options={ name="实际销售额" }` 覆盖 → legend 显示 `"实际销售额 - actual_sales_real"` (Chinese + y 列名)

⚠️ 已知 limitation: Evidence wrapper 自动拼接 echarts_options.name 跟 y 列名 (Chinese + dash + English), **去不掉**(除非该图只有 1 个 series)。
命名规则的源码与影响面(多 series 时必带后缀 / 改 y 列名会改 series 名 / 写死完整系列名的地方要同步改)→ **ch21 §1.3**。
要图例真正显示「线 + 标记形状」而不是只显示形状 → **ch21 §1.4**(`legend.data` + `icon="path://"`)。

### 避坑对比 (4 次尝试)

| 尝试 | 错 | 修 |
|---|---|---|
| `barGap: '-50%'` (Phase 7.42) | "Node 'tag' is missing opening" | `barGap="-50%"` (`=` 不是 `:`) |
| `barGap="-50%"` (Phase 7.42 v1) | still side-by-side (重叠不足) | `barGap="-100%"` 完全重叠 |
| `barWidth="70%"` 内联 `{ name="X", barWidth="Y" }` (Phase 7.43) | "Expected '}', identifier, string, or whitespace but ',' found" | 多 key **newline 分隔** |
| `axis="y2"` 漏写 | line 默认主 Y 轴, 退款率混入金额 | `axis="y2"` 必加 |

### line 节点标记 (showSymbol + itemStyle.opacity) (Phase 7.58.4 实证)

**场景**: 用户要求 combo_chart 折线在数据点处显示节点标记 (symbol / 圆点)。

**❌ 错误写法 (只加 showSymbol + symbolSize, 不生效)**:

```markdown
{% line y="refund_rate_real" axis="y2" echarts_options={
    name="退款率"
    showSymbol=true
    symbolSize=7
} /%}
```

**✅ 正确写法 (必须加 itemStyle.opacity=1)**:

```markdown
{% line y="refund_rate_real" axis="y2" echarts_options={
    name="退款率"
    showSymbol=true
    symbolSize=7
    itemStyle={
        opacity=1
    }
} /%}
```

**根因** (查 Evidence 源码定位, 不要自己猜 CSS):

`core/src/user-components/tags/series_charts/combo_chart/series/seriesConfig.ts` 的 `applyLineMarkerVisibility` 函数:

```js
if (s.itemStyle?.opacity !== undefined || s.showSymbol === false || s.symbol === 'none') return;
// 上面不 return 就继续:
s.itemStyle = { ...s.itemStyle, opacity: labels ? 1 : 0 };  // line 无 label → opacity=0 隐藏 marker
```

- combo_chart 的 line series **默认无 label** → `labels=false` → `itemStyle.opacity=0` → marker 被 opacity=0 隐藏
- 只有 author **显式设 `itemStyle.opacity`** 才让函数提前 return, marker 保持显示
- 该函数在 `ComboChart.svelte` 每个 data series 后调用 (注释 "Runs after every override so any author marker setting wins")

**关键源码位置** (后续查证用):
- `seriesConfig.ts` — `applyLineMarkerVisibility` (~348-378 行) + `DEFAULT_SYMBOL_SIZE = 9`
- `ComboChart.svelte` — 调用点 (~661 行): `for (const s of dataSeries) { merge(s, seriesOverrides); applyLineMarkerVisibility(s); }`

### 2 bar 同宽模式 (Phase 7.46-7.48 实证)

**场景**: 实际销售额 + 净实销 等"总量 vs 净量"对比 (用户要求同 x 同宽, 净实销在前面)

```markdown
{% combo_chart
    echarts_options={
        barGap="-100%"           {# 完全重叠 #}
        barCategoryGap="20%"      {# 月份间 gap #}
    }
%}
    {% bar y="total" stacked=false echarts_options={
        name="总量"
        barWidth="60%"
        itemStyle={
            color="#1E40AF"   {# Pentagram navy #}
        }
    } /%}
    {% bar y="net" stacked=false echarts_options={
        name="净量"
        barWidth="60%"
        itemStyle={
            color="#0F766E"   {# Pentagram dark teal #}
    }
} /%}
{% /combo_chart %}
```

**关键 3 点**:
1. **2 bar `barWidth` 完全相同** (60% / 60%) — 不同宽度导致视觉错位 (Phase 7.46 教训)
2. **显式 `itemStyle.color`** (不依赖 ECharts 默认调色板) — Pentagram brand 一致
3. **净量 bar 在后声明** (默认后绘制 = 在前) — 视觉层级清晰

### 透明度 trade-off (Phase 7.46 vs 7.48)

| 方案 | 视觉 | 用户反馈 |
|---|---|---|
| `opacity=0.6` 实际销售额 (Phase 7.46) | 实际销售额 navy 60% 透明 → 浅粉/淡蓝 | ❌ "颜色与示例不符" (legend navy 实色, bar 浅色) |
| **完全不透明 (Phase 7.48)** | 实际销售额 navy 实色, 净实销 teal 实色 | ✅ 与 legend 颜色方块完全一致 |

**结论**: 用户期望 `legend color square == bar visible color` (实色一致),`opacity<1` 会让 Pentagram 深色视觉淡化 → 弃用。

### Pentagram 颜色 palette 映射 (Phase 7.47 sales-overview)

| Theme yaml 字段 | Hex | 用途 | 销售 BI 例 |
|---|---|---|---|
| `total_color` | `#1E40AF` (navy brand) | 总量背景层 | 实际销售额 (柱背景) |
| `increase_color` | `#0F766E` (dark teal) | 净量前景层 | 净实销 (柱前景) |
| `decrease_color` | `#B91C1C` (dark red) | 减项 | 退款率 (Phase 7.39 锁定) |
| `connector_color` | `#94A3B8` (gray) | connector lines | waterfall (Phase 7.39 锁定) |

**为何必显式 color**: ECharts 默认 series 调色板 (dataView 0=#5470C6) 与 Pentagram theme 不同步, 必 `itemStyle.color="#hex"` 显式指定。

### 月度趋势场景完整模板 (Phase 7.40 + 7.41 + 7.43 + 7.47)

**SQL (单查询 3 列)**: 详见 ch05 — y_fmt="pct1" expects raw 0-1 fraction 段 (Phase 7.38 教训)

**chart (双 Y 轴 + 中文 legend + 2 bar 同 x 重叠 + Pentagram 配色)**:

```markdown
```sql monthly_trend
SELECT
  TO_CHAR("订单日期", 'YYYY-MM') AS source_year_month,
  ROUND(SUM(CASE WHEN "类型" = '真实' THEN "销售金额" ELSE 0 END), 0) AS actual_sales_real,
  ROUND(SUM(CASE WHEN "类型" = '真实' THEN "销售金额" - "退货金额" ELSE 0 END), 0) AS net_sold_real,
  ROUND(1.0 * SUM(CASE WHEN "类型" = '真实' THEN "退货金额" ELSE 0 END)
    / NULLIF(SUM(CASE WHEN "类型" = '真实' THEN "销售金额" ELSE 0 END), 0), 4) AS refund_rate_real
FROM (...) daily_store_all
WHERE {{date_filter.between}} AND ({{ shop_filter.filter }})
GROUP BY 1
```

{% combo_chart
    data="monthly_trend"
    x="source_year_month"
    y_fmt="cny0"
    y2_fmt="pct1"
    title="月度销售趋势 (实际销售额 / 净实销 / 退款率)"
    echarts_options={
        barGap="-100%"
        barCategoryGap="20%"
    }
%}
    {% bar y="actual_sales_real" stacked=false echarts_options={
        name="实际销售额"
        barWidth="60%"
        itemStyle={
            color="#1E40AF"
        }
    } /%}
    {% bar y="net_sold_real" stacked=false echarts_options={
        name="净实销"
        barWidth="60%"
        itemStyle={
            color="#0F766E"
        }
    } /%}
    {% line y="refund_rate_real" axis="y2" echarts_options={
        name="退款率"
    } /%}
{% /combo_chart %}
```

## Reference Tables

| 任务 | 推荐组件 | 注意 |
|---|---|---|
| KPI 大数字 | `{% big_value %}` | + `{% delta %}` 配套 |
| 趋势图 | `{% line_chart x=date y=value %}` | x 列 date type,format 自动 |
| 柱状对比 | `{% bar_chart x=category y=value %}` | categorical x |
| 大表展示 | `<DataTable data={x} />` | 别用 `rows=N` 旧 API |
| 月度筛选 | `{% range_calendar %}` | 唯一 date picker |
| 单选下拉 | `{% dropdown %}` | multi-select by `multiple=true` |

## 关键事实

- **custom_echart 套 box_plot / waterfall** — 当 80+ 内置组件不满足时,逃逸到 ECharts 全功能(2026-09-21 上线)
- **mintlify 文档 8 个 URL 404** — checkbox/date_picker/box_plot/waterfall/button/data_table/text/value_fmt,URL 与标签错位是常见踩坑

## Connects To

- **ch21** — 图表组件 14 条契约(静默毁图 4 条 / 选项合并 / 图例 / 取 canvas / 表格 / 页面内容规则 / 3-piece 验收 / A/B 实验法)
- **ch05** — `html-tags-not-allowed` 在 custom_echart 内部仍触发,`{% html %}` 块逃逸
- **ch08** — Pentagram 主题定制 component className 锚点
- **ch18 (BI 工具对比)** — Evidence 组件 vs Metabase drag-drop 的关键差异
