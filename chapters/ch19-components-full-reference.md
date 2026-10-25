# Chapter 19: Components Full Reference (94 个组件全量索引)

> **R3 升级版 · 2026-10-15 抓取全量**
> 源数据: `mirror/components/{Values,Tables,Charts,Maps,Inputs,Custom,UI,Logic}/` — 随 skill 分发的公开文档镜像,sitemap 94/94 全覆盖
> 索引入口: `mirror/README.md`(8 大类 + 抓取时间 + 上游出处)

## Core Idea

docs.evidence.dev/components/ 共 **94 个文档页**,sidebar 导航分 8 大类(V/T/C/M/I/C-UI/L)。每页都含 Attributes + Examples + 可用日期变量,随 skill 分发的 mirror 保留原文以供 IDE 实时校验 / 中文 schema 适配。

## 8 大类速览

| 类 | 页数 | 用途 | 本地路径 |
|---|---|---|---|
| **Values** | 5 | KPI 大数字、变化量、热力格 | `mirror/components/Values/` |
| **Tables** | 4 | 表格、维度、指标、透视 | `mirror/components/Tables/` |
| **Charts** | 29 | 20 完整图表 + 6 series/layer + 3 reference marker | `mirror/components/Charts/` |
| **Maps** | 5 | 自定义地图 + 3 layer | `mirror/components/Maps/` |
| **Inputs** | 16 | 下拉、滑块、日期、过滤等 | `mirror/components/Inputs/` |
| **Custom** | 1 | Html 原生块 | `mirror/components/Custom/` |
| **UI** | 30 | Accordion、Callout、Image、Link、Modal、Note 等 | `mirror/components/UI/` |
| **Logic** | 4 | If / Else If / Else / Repeat | `mirror/components/Logic/` |
| **总计** | **94** | sitemap 全覆盖 | — |

## Full Component List (94)

### Values (5)

| Component | Purpose | Key attrs |
|---|---|---|
| `value` | 通用数值/文本 | `data`, `value`, `fmt` |
| `big_value` | 大号 KPI | `title`, `value`, `comparison`, `fmt` |
| `delta` | 同比/环比 | `value`, `comparison`, `down_is_good` |
| `heat_grid` | 网格热力图 | `data`, `x`, `y`, `color` |
| `progress_bars` | 进度条列表 | `data`, `value`, `max` |

### Tables (4)

| Component | Purpose | Key attrs |
|---|---|---|
| `table` | 通用表格 (DataTable v0.10+) | `data`, `rows`, `columns`, `format_titles` |
| `dimension` | 单维度分组 | `data`, `dim`, `metric` |
| `measure` | 单一指标 | `data`, `metric`, `fmt` |
| `pivot` | 透视表 | `data`, `rows`, `cols`, `values` |

### Charts (29)

**完整图表 (20)**: `area_chart`, `bar_chart`, `bubble_chart`, `calendar_heatmap`, `candlestick`, `chord_chart`, `combo_chart`, `custom_echart`, `funnel_chart`, `heatmap`, `histogram`, `horizontal_bar_chart`, `line_chart`, `pie_chart`, `polar_chart`, `radar_chart`, `sankey_chart`, `scatter_chart`, `treemap`, `waterfall_chart`

**Series/Layer (6)**: `area`, `bar`, `bubble`, `line`, `scatter`, `sparkline` — 用于 `combo_chart` 内层

**Reference marker (3)**: `reference_area`, `reference_line`, `reference_point` — 标注阈值/目标

### Maps (5)

`map`, `custom_map`, `area_layer`, `heatmap_layer`, `point_layer`

### Inputs (16)

`button_group`, `comparison_selector`, `benchmark_comparison`, `target_comparison`, `date_grain_selector`, `dropdown`, `filter_bar`, `input_tabs`, `option`, `range_calendar`, `slider`, `table_filter`, `text_input`, `toggle`, `dimension_grid`, `period_selector`

### Custom (1)

`html` — 原生 HTML 块 (注意 html-tags-not-allowed 规则,见 ch15)

### UI (30)

`accordion`, `accordion_item`, `accordion_title`, `audio`, `callout`, `clock`, `commentary`, `details`, `download`, `fence`, `file`, `hidden_in_print`, `icon`, `iframe`, `image`, `info`, `line_break`, `link`, `link_button`, `logo`, `modal`, `note`, `only_in_print`, `page_break`, `partial`, `print_group`, `row`, `stack`, `tab`, `tabs`

### Logic (4)

`if`, `else_if`, `else`, `repeat` — 控制流

## Quick Decision (按任务选组件)

| 任务 | 推荐组件 | 详见本地 |
|---|---|---|
| KPI 大数字 + 变化 | `big_value` + `delta` | `mirror/components/Values/` |
| 趋势曲线 (date x) | `line_chart` | `mirror/components/Charts/components-line_chart.md` |
| 类别对比 (cat x) | `bar_chart` | `mirror/components/Charts/components-bar_chart.md` |
| 多系列叠加 | `combo_chart` + `area`/`bar`/`line` | `mirror/components/Charts/components-combo_chart.md` |
| 占比 | `pie_chart` 或 `treemap` | `mirror/components/Charts/` |
| 地理分布 | `map` + `point_layer` | `mirror/components/Maps/` |
| 中文列表格 | `table` + `format_titles=false` | `mirror/components/Tables/components-table.md` + ch05 |
| 用户交互过滤 | `dropdown` / `period_selector` | `mirror/components/Inputs/` |
| 条件显示 | `if` / `else` | `mirror/components/Logic/` |
| 隐藏 markdown 残留 / 自定义 | `html` + `partial` | `mirror/components/Custom/components-html.md` + ch15 |

## Anti-patterns (新发现的,基于 94 页全量)

- ❌ URL 写 `box_plot` / `data_table` / `checkbox` / `date_picker` → 全部 404,**官方只支持 snake_case + 上述 94 个组件**
- ❌ 把 `value` 当 `big_value` 用 → 大屏不显
- ❌ `format_titles=true` (默认) + 中文列名 → Title Case 破坏中文别名,**改 `false`** (见 ch05)
- ❌ `combo_chart` 引用未在 `area`/`bar`/`line` 之内的 series → 报 "unsupported type undefined"
- ❌ `partial` 写在 `pages/` 而非 `partials/` 目录 → Core 报 `html-tags-not-allowed`(误把 `{{> partial }}` 当 HTML)
- ❌ `hidden_in_print` / `only_in_print` 仅在 print_group 内生效,单独使用无效

## 与 ch11 (R3) 关系

- ch11 = R3 摘要(80+ 组件概览 + 7 大类 framework)
- **ch19 = 全量索引**(94 个逐项列出 + 本地 wikilink 跳转)
- 用法: 先看 ch11 决策 → 跳 ch19 查具体 attrs → 跳本地 docs mirror 看原文示例

## Reference Links

- mirror 索引: `mirror/README.md`(8 大类 + 抓取时间 + 上游出处)
- 原文 sitemap: <https://docs.evidence.dev/sitemap.xml> (153 URL, 94 在 `/components/`)
- 框架语义: ch11 (Components 7 大类 Framework)
- 中文列陷阱: ch05 (SQL Anti-patterns)
- theme / html block: ch15 (Theme & Pentagram P2)
