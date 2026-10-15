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

- **ch05** — `html-tags-not-allowed` 在 custom_echart 内部仍触发,`{% html %}` 块逃逸
- **ch08** — Pentagram 主题定制 component className 锚点
- **ch18 (BI 工具对比)** — Evidence 组件 vs Metabase drag-drop 的关键差异
