# Chapter 21: 报表页 UI 契约(图表 / 表格 / 页面内容 / 验证)

> 2026-10-21 蓝红对比页 8 轮改造沉淀。**组件契约全部来自 prod 二进制实证,不是文档推测。**

## 0. 查组件契约的方法(先会查,再谈契约)

```bash
# 组件契约在 Core 二进制里(119MB 单文件)
sudo docker exec sales-bi-core sh -c "grep -ao '.\{200\}<prop>.{400\}' /usr/local/bin/evidence"
# prop 描述 + 官方 blockExample
sudo docker exec sales-bi-core sh -c "grep -ao '.\{80\}<prop>=.{0,600\}' /usr/local/bin/evidence"
# ⚠️ grep -ao 不跨行; 模式里带引号要小心 shell 转义(用 .\{n\} 代替字面引号)
```
`grep -a -A 12 '<prop>={'` 可看跨行官方示例。**不确定某个 prop 怎么用时,先查二进制,不要猜。**

## 1. 图表组件契约 14 条(Evidence Core 0.10.1)

### 1.1 会静默毁图的 5 条(整图不渲染,页面其它部分照常 → 极易漏检)

1. **`stacked` 默认是 `true`**(压缩后是 `!0`)。并列柱必须显式 `stacked=false`。
2. **combo_chart 子图的 `name` 必须写进 `echarts_options` 内部**:
   ```markdown
   {% line y="refund_rate_red" echarts_options={
       name="红队"        ← 在这里
       showSymbol=true
   } /%}
   ```
   写成子图属性 `{% line y="..." name="红队" %}` → 校验 error → **整图不渲染**。
3. **图表列名必须 ASCII**(y 列同样)。chart wrapper 会把列名再写一遍 `AS "<列名>"`,
   中文别名 → 空别名 → DuckDB `zero-length delimited identifier` → **整图 500**。
   中文只能放**数据值**里(行值 = 蓝队/红队),列名一律 ASCII。
   ⚠️ **x 列(类别轴)尤其容易踩**:`x="店铺"` → wrapper 生成 `SELECT 店铺 AS "", ...` → 整图 500,
   页面正文直接印 `zero-length delimited identifier`。修法:源 SQL 里 `"店铺" AS shop` + `x="shop"`
   (中文店名作为**数据值**,坐标轴照常显示中文)。
4. **无子图的组件必须自闭合**(`{% bar_chart ... /%}`)。漏掉 → 页面报「Node is missing closing」,
   且**后面所有组件都被截断**(整页只剩前面几张图)。有子图的才写 `{% /combo_chart %}`。
5. **SQL 里的 `ORDER BY` 传不到图表** —— chart wrapper 会把整段 SQL 包成
   `SELECT <x> AS "<x>", <y> AS "<y>" FROM ( <你的 SQL> ) "__ev_inline_<name>" GROUP BY <cols> LIMIT 2000`,
   **外层这个 GROUP BY 是 hash 聚合 → 行序被打乱** → 「Top 10 排行」类图表的柱子顺序是随机的。
   → 排序必须走图表自身的 prop(见 §1.6)。

### 1.2 选项合并规则

5. **`echarts_options` 的数组是整体替换**,不是按索引合并。传 `series=[{...}]` → 整个 series 数组被替换 → 图表数据全丢。
   深合并只对**对象**生效(key-by-key)。
6. **`echarts_series_options` = 所有 series 共用一份**,不是按名映射。
   想**按 series 单独设 symbol / 线型** → 只能用 combo_chart 子图的 `echarts_options`。
7. **`chart_options.series_colors` = 按 series 名映射**(可用,首选):
   ```markdown
   chart_options={ series_colors={ "蓝队"="#5278B2" "红队"="#BA665D" } }
   ```
   key = series 列的**数据值**,不依赖 series 顺序。`color_palette`(数组按索引上色)依赖顺序,别用。

### 1.3 图例(最常踩)

8. **Evidence built-in 图例不读 series 的 `itemStyle.color` / `symbol`** —— 它是 DOM 色块
   (`<span style="background:rgb(...)">`),颜色取主题 colorScale,标记永远是细线色块。
   → 想让图例跟随线色/标记,必须 `legend=false` + `echarts_options={ legend={ show=true } }` 切 **ECharts 原生图例**。
9. **原生图例默认只画「系列标记形状」,没有那条短横线**。根因:series 的 `getLegendIcon` 画线用
   `lineStyle.stroke`,而 legend data item 的 lineStyle 没有 stroke(且 lineWidth=0)→ 线不可见。
   **解法(见 §1.5)**。
10. **Evidence 组 series 名 = `${name} - ${y列别名}`**(多 series 时必带后缀):
    ```js
    let X = 全图 series 总数;
    let G = yProcessed.displayAlias;              // 'refund_rate_blue' → 'Refund Rate Blue'
    if (typeof w.name !== 'undefined')
      if (X > 1) G = `${w.name} - ${G}`;           // '蓝队 - Refund Rate Blue'
      else G = w.name;                            // 单 series → 只要 '蓝队'
    ```
    - 影响面:图例文字 / tooltip / 任何按 series name 匹配的地方。
    - 后缀**去不掉**(除非该图只有 1 个 series)→ 想要纯中文图例,只能放弃「点击图例切换 series」。
    - **推论:改 y 列名会改 series 名** → 所有写死完整系列名的地方(如 §1.5 的 `legend.data[].name`)必须同步改。

### 1.4 原生图例改「短线条 + 叠加标记形状」= `legend.data` + `icon="path://"`

11. 写法(自绘 SVG:25×14 盒,底线 y6..8 贯穿 x0..25,形状居中 (12.5,7)):
    ```markdown
    legend=false
    echarts_options={
        legend={
            show=true
            data=[
                {name="蓝队 - Refund Rate Blue" icon="path://M0,6 L25,6 L25,8 L0,8 Z M5.5,7 C5.5,3.134 ..." itemStyle={ color="#5278B2" }},
                {name="红队 - Refund Rate Red" icon="path://..." itemStyle={ color="#BA665D" }}
            ]
        }
    }
    ```
    - `path://` → ECharts `createSymbol` → zrender `createFromString`,会**按比例缩放居中**进 itemWidth×itemHeight 盒;
      把 path 的 bbox 设计成**恰 = 25×14** → 缩放比 1,`cover` / `center` 两种 fit 结果一致。
    - path 整体按 `itemStyle.fill` 填色 → 队色;一条 path 里画「底线矩形 + 形状」两个子路径即得目标形态。
    - ⚠️ **`legend.data` 数组元素必须逗号分隔**;改成换行 → **整图不渲染**(页面其它部分正常,极易漏检)。
    - ⚠️ `name` 必须是**完整系列名**(见第 10 条),否则图例点击不再切换 series。
    - ❌ legend **级** `lineStyle` / `itemStyle` 不会下发到 data item(实测无效)→ 逐项样式只能写进 `legend.data[i]`。

### 1.5 其它可用 prop / 取图方法

12. `series_order=["蓝队","红队"]` 固定 series 与图例顺序;`y_axis_options={ title="退款率" }` 改 Y 轴标题;
    `legend_location` top/bottom;`chart_options` / `echarts_options` / `echarts_series_options` 三层。
    ⚠️ **轴标题的键是 `title`,不是 `name`** —— `name` 不是 `y_axis_options` 的合法键,
    Core 会**静默丢弃**(页面照常渲染、只是轴名仍显示 y 列英文别名,如 "Net Sold"),
    SSR 序列化属性里也能看到 `y_axis_options":{"title":...}` 才算生效。
    (2026-10-21 实测: 本 skill 旧版写的 `name=` 是错的, 蓝红对比页也中招)
13. **取 canvas 的正确方法**:取**文档顺序上紧随标题之后的第一个 canvas**。
    ❌ 不要「从标题向上找祖先里的 canvas」—— 章节标题(h2)的祖先含整页所有图,会命中第一张图
    (本项目踩过:校验脚本误采蓝队瀑布图,误判「红色缺失」)。
14. **canvas 内元素定位靠像素聚类**,不要猜坐标:先按颜色/亮度扫描出像素 x 区间,再聚类成簇,
    取目标簇中心点击(本项目踩过:猜 x 坐标两次都点空,以为是图例失效)。

## 1.6 图表排序契约(排行类图表必读)

| prop | 类型 | 语义 | 二进制 description 原文 |
|---|---|---|---|
| **`order`** | String | 按列排序,**affectsQuery=true** → 进查询的 ORDER BY | `'Column name(s) with optional direction (e.g. "column_name", "column_name desc")'` |
| **`x_sort`** | enum(`asc`/`desc`/`data`) 或 string[] | x 轴类别排序;`data` = 保留查询顺序;数组 = 自定义顺序 | `'Sort order for x-axis categories. Options: asc (alphabetical), desc (reverse alphabetical), data (preserve query order), or an array for custom order'` |
| `y_sort` | 同上 | y 轴类别排序(横向柱图) | 同上 |

- **排行图(如 Top 10)用 `order="<y列> desc"`**:
  ```markdown
  {% bar_chart data="top10_sales" x="shop" y="total_sales" order="total_sales desc" /%}
  ```
  `order` 走查询 → 落在 wrapper 的**外层**(GROUP BY 之后)→ 顺序正确。
- `x_sort="data"` 依赖查询顺序 → **在 chart wrapper 下不可靠**(外层 GROUP BY 打乱),别用。
- `x_sort=["A","B","C"]` = 固定类别顺序(适合枚举固定的系列)。
- ⚠️ `order` 的列名要用**源 SQL 的输出别名**(不是中文原列名)。
- 官方文档镜像:`E:/obsidian/evidence/components/Charts/components-bar_chart.md`
  (含 "Sorting by Value" / "Custom Category Order" 两节)。
- **表格组件没有这个问题**(无 wrapper 的 GROUP BY)→ 明细表一直按 `ORDER BY "销售总额" DESC` 正确排序。

## 1.7 图表放大:bar/line/combo 没有内置 prop,走 ECharts toolbox(2026-10-22 实测)

- **Core 0.10.1 只有 `candlestick` 有 `chart_options.zoom`**(二进制 description 明写
  "Candlestick chart configuration options");**bar_chart / line_chart / combo_chart 没有任何放大 prop**。
- 表格的「Expand table」是**内置组件**(aria-label="Expand table"),**图表没有对应物**。
- **可行解法**:ECharts 原生 `toolbox`,通过 `echarts_options` 传(对象深合并,不触发 §1.2 的数组替换坑):
  ```markdown
  echarts_options={ toolbox={ orient="vertical" itemGap=12 right=4 top="center"
                    feature={ dataZoom={ yAxisIndex="none" } restore={} saveAsImage={} } } }
  ```
  → 右侧出现 3 个图标:**框选缩放 / 还原 / 保存为图片**。`dataZoom` 的 `yAxisIndex="none"`
  只允许横向缩放(30 个分类名的柱图,纵向缩放没意义且会拖慢渲染)。
- **验收怎么测**:toolbox 图标画在 **canvas 内**,DOM 查不到 →
  扫 canvas 右侧 6~8% 宽、全高的深色像素(`d[i+3]>40 && d[i]<200`),阈值 >30 px/图。

## 1.8 combo_chart 顶层**不能写 `y`**(2026-10-22 踩坑,整图静默不渲染)

顶层合法:`data` `x` `y_fmt` `y2_fmt` `y_axis_options` `y2_axis_options` `order`
`legend` `echarts_options` `echarts_series_options` `title` `stacked=false` …

- ❌ **顶层写 `y="xxx"` → 页面正文红条 `Invalid attribute: 'y'` → 该图完全不渲染**
  (页面 200、其它图照出、无 api 报错 → 极易误判为"数据问题")。
  **y 只能写在子图上**:`{% bar y="total_gross_profit" ... /%}`。
- 双轴图**不要手写 `yAxis=[{...} {...}]`**:
  - 数组元素漏逗号 → Markdoc 直接崩,页面印 `Node 'tag' is missing opening`,**该组件之后的所有内容全丢**;
  - Evidence 会按子图的 `axis="y2"` 自动生成双轴,**不需要手写**;
  - 轴标题用 `y_axis_options={ title="毛利额" }` / `y2_axis_options={ title="毛利率" }`(键是 `title`)。
- 组合图配色:`echarts_series_options` 对**全图所有 series 共用一份** → 双轴图必须在**各自子图**
  的 `itemStyle.color` 里分别写。

## 1.9 图表白色卡片背景 = 项目自定义 CSS 的 `data-component-type` 选择器(2026-10-22 实证)

**现象**:`combo_chart` / `waterfall_chart` 没有白色卡片背景(直接画在米色页面底上),
而 `bar_chart` / `line_chart` 有 —— **不是 Core 的 bug,是项目自定义 CSS 漏了类型**。

**真源**:`/static/sales-bi.css`(nginx `sub_filter` 注入到每个页面 `<head>`;
仓库副本 `evidence-core/static/sales-bi.css`,prod 路径 `/var/www/evidence/static/sales-bi.css`)。

**规则长这样**(Region C):
```css
.component-wrapper[data-component-type='line_chart'],
.component-wrapper[data-component-type='bar_chart'] { background:#FFFFFF !important; ... }
```
→ **只列了两种图表类型** → 漏掉的类型就没有卡片。
修法 = 把缺的类型补进同一组选择器(卡片背景 / span 字体 / 标题字体 共 3 处规则)。

**排查方法(下次直接照做,别再挖二进制)**:
1. `git log -- <css 文件>` → 确认是不是最近改坏的;
2. 三页 DOM 链对比 → 排除页面级差异;
3. `document.elementFromPoint` 拾取 + 向上遍历祖先 → 找 `.component-wrapper`(Core 的组件包装层);
4. **`page.evaluate` 读 `document.styleSheets` 找含 `data-component-type` 的规则** → 真凶就在这;
   (Core 自带 CSS 在 `_app/immutable/assets/*.css`,项目自定义的在 `/static/sales-bi.css`)

**白卡不是 canvas 背景**:实测 3 个 canvas 的背景像素全是 `0,0,0,0` —— 卡片在 DOM 容器上,不在 canvas 里。
**`page_settings: { cards: true }` 无效**(那是"整页内容包卡片",与图表卡片无关)。

## 1.10 图表 fullscreen:Core 0.10.1 **不支持单图**(2026-10-22 实证)

| 能力 | 状态 | 证据 |
|---|---|---|
| **表格**单独页打开 | ✅ 内建 | `aria-label="Expand table"` → 展开后 `"Exit fullscreen"`,`pageSizeOverride=(vh-140)/28` |
| **图表**单独页打开 | ❌ **没有** | bar/combo/waterfall 的 prop 清单无 fullscreen/expand;`Expand table` 的按钮函数只给 table 用 |
| `chart_options.zoom` | ❌ 只有 **candlestick** 有 | 二进制 description 原文:"Candlestick chart configuration options" |
| **页面级**全屏 | ✅ 有 | `?fullscreen` URL 参数 + `document.documentElement.requestFullscreen()`,控件在**页头 `...` 菜单**(鼠标移动 3 秒内显示) |
| ECharts `custom` feature(单图全屏) | ❌ 报表页做不到 | 需 `echarts.registerExtension` 注册 JS 回调;Markdoc 组件无法注入 JS |

→ 想要"单图全屏"只能:每图单独建一页 / 改 Core / 写自定义 Markdoc 组件 —— 都超出"改报表页"范围。

### 1.10-bis ✅ 已落地的替代:nginx sub_filter 注入 `chart-fullscreen.js`(2026-10-22 实测)

既然 Core 图表无内建 fullscreen,项目用**已有的 nginx sub_filter 注入机制**(与 pentagram-nav.js 同款)给图表加「全屏」按钮,交互与表格 Expand 一致:

1. **新增 `evidence-core/static/chart-fullscreen.js`**:给每个 `.component-wrapper[data-component-type$="_chart"]`
   加一个按钮(`aria-label="Expand chart"`),点击调浏览器原生 `wrapper.requestFullscreen()`;
   `fullscreenchange` 后 dispatch `window.resize` 让 ECharts 重绘。
2. **`sales-bi.css` 追加** `.chart-fullscreen-btn`(右上角 26×26)+ `.component-wrapper:fullscreen`(白底+padding)。
3. **nginx 第 308 行 sub_filter 追加** `<script src="/static/chart-fullscreen.js?v=1" defer></script>`,
   `nginx -t` + `nginx -s reload`。

**实测效果**:canvas 942×256 → 1536×1100(点击后 ECharts 自动 resize 填满)。
**验收**:`.chart-fullscreen-btn` 数量 = 图数;点击后 `document.fullscreenElement` 是 component-wrapper;
canvas 尺寸变大(截图肉眼)。

⚠️ **关键事实**:chart 组件的 prop 里 grep `expand/fullscreen/zoom/maximize` **零结果**(二进制实证 3 次);
`Expand table` 按钮(`aria-label`)只出现 1 次且在 table 组件;图表外层 `<button>` 无 onclick、`cursor-default`。
**不要再去二进制里找图表 fullscreen prop —— 它不存在。**

## 1.12 图表极值处理:轴收紧 + 超界标记点(2026-10-22 实测)

**场景**:副纵轴上少数点远偏离主体(如占比 270%、费率 -204%),把其余点的差异压平。

### Evidence/Markdoc 的硬限制(先知道再设计)
| 手段 | 可用? | 实证 |
|---|---|---|
| `y2_axis_options={ min=数字 max=数字 }` | ✅ | 轴区间收紧,刻度自动取 nice 值 |
| 子图 `echarts_options` 简单属性合并 | ✅ | `symbol` / `symbolSize=数字` / `itemStyle={color=字符串}` / `lineStyle={width=数字}` |
| **函数回调**(`symbolSize=(v)=>...`) | ❌ | Markdoc 属性解析器报 `Expected ... but "(" found` → **整个组件被丢弃**,页面印红条 |
| **数组字面量**(`markLine={data=[{...}]}` / `graphic=[...]`) | ❌ | 报 `Expected "}", identifier, string, or whitespace but "," found` → 同上 |
| `clip=false`(让点画到绘图区外) | ❌ | 图表容器是 `overflow-hidden` → 画布外被裁掉,无效果 |

### 可行方案(三件套)
1. **轴收紧**:`y2_axis_options={ min=... max=... }` 定到主体区间 → 95% 的点铺满轴高。
2. **超界标记点**:SQL 额外输出一列,**超界时取「语义边界值」**(如占比>100% → `1.0`;费率<-30% → `-0.3`),
   其余为 `NULL`(ECharts 不画 null 点);
   图上再加一个 line 子图用这列,`lineStyle={width=0}` + 红色大菱形 `symbolSize=16` → 只剩超界点可见。
   - ⚠️ **标记值必须取轴内值,不能取真值**:ECharts 默认 `clip: true` 会**整个裁掉超轴范围的 symbol**。
   - ⚠️ **轴要留一点余量**(如 max=1.05 / min=-0.35),否则贴在轴边界的点被切一半(实测红点只剩 5px)。
   - ⚠️ **tooltip 真值由主线 series 提供**:轴裁剪不影响 tooltip,它读原始 data(实测 `max=1` 时 hover 仍显示「158.8%」)。
3. **精确值在明细表**:明细表已有对应数值列,业务要精确数字看表。

**为什么不用回调/数组**:上面的限制表是 2026-10-22 在临时页 A/B 实测的(3 个变体:函数回调、对象数组、clip=false),
每个变体都被 Markdoc 解析器拒绝 → 整组件消失。别再试。

## 1.13 dropdown 过滤契约:`filter` 模板自带列名(2026-10-22 分类详情页实证)

**最易踩的一条** —— dropdown 的 `{{ id.filter }}` 模板**渲染出来已经带列名**:

| 模板 | 单选时渲染成 | 未选中时 |
|---|---|---|
| `{{ id.filter }}` | `virtual_category='收腹带'` | `true` |

→ **只能写 `WHERE ({{ id.filter }})`,不能再自己加列名**。
自己加列名 → `WHERE virtual_category virtual_category='收腹带'` → **syntax error → 整页 block 全挂**
(页面 200、其它图照出,只有这几个 block 报 `syntax error at or near "virtual_category"`)。

```sql
-- ✅ 正确(项目里 sales-overview / compare-teams / shop-overview 都是这么写)
WHERE ({{ shop_filter.filter }})
-- ❌ 错误:列名重复
WHERE shop_name {{ shop_filter.filter }}
```

- ⚠️ **加括号不是可选的**:未选中时渲染成 `true`,`WHERE true` 合法,但不加括号时
  `WHERE true AND ...` 也合法 —— 括号的作用是隔离,建议保留。
- **二进制实证**(0.10.1 dropdown):
  - `value_column` / `label_column` / `title` / `options` / `initial_value` / `select_first` / `default_top_n` / `placeholder` 都是合法 prop。
  - `select_first`:`"Automatically select the first option when the component loads"`,**默认 false** → 不显式写 `select_first=true` 就是「未选中」。
  - **dropdown 没有 `order` prop** → 选项顺序由 wrapper 固定为 `ORDER BY <value_column>`(字母序),
    **无法让下拉按业务指标排序**,`select_first` 拿到的必是字母序第一项。
  - 选项查询的 wrapper 形态:
    `SELECT virtual_category AS "value" FROM ( <你的 SQL> ) "__ev_inline_<block>" WHERE (virtual_category IS NOT NULL) GROUP BY virtual_category ORDER BY virtual_category LIMIT 10000`
    → **你在自己 SQL 里写的 `ORDER BY` 会被 wrapper 的 `ORDER BY` 覆盖**(与图表 §1.1-5 同理)。
- **排查手法**:`/api/query` 的请求体能看到模板替换后的**真实 SQL** → 一眼看出是不是列名重复。

## 1.14 明细表 grain 契约:行标签必须与行数据同粒度(2026-10-22 销售总览页实测)

**现象(2026-10-22 用户报「销售额/毛利/去费净利润重复计算、偏高」)**:
销售总览页「明细表」每行显示**全公司该月合计**,却挂着某个店的简称:

| 月份 | 页面显示 | 页面销售额 | **该店真实值** | 倍数 |
|---|---|---|---|---|
| 2026-08 | C店 - 正康 | 5,727,968 | **12,871** | 444× |
| 2026-09 | 1688 - 正康 | 5,102,239 | 7,958 | 641× |

### 根因模式(通用):`GROUP BY` 丢了维度 + `MIN(col)` 补标签

```sql
SELECT TO_CHAR(d,"YYYY-MM") AS ym,
       MIN("店铺") AS min_shop,     -- ← 标签 = 字典序最小的店,与本行数据无关
       SUM("销售金额") AS sales      -- ← 求和 = 全公司该月
FROM t GROUP BY 1                    -- ← 店铺维度被丢掉
```

→ **行数据与行标签来自不同聚合粒度** → 数值虚高,而且标签逐行跳变(本例在「C店-正康 / 1688-正康」间翻转)。
**这类 bug 页面照常渲染、API 不报错,只能靠交叉核对发现**。

### 修法

1. `GROUP BY` 必须包含**表里出现的每一个维度列**(`GROUP BY 1, 2`)。
2. 标签从**本行自己的维度**取,不要用 `MIN(col)` / `MAX(col)` 兜。
3. **所有按维度分摊的费用列(如推广费)必须同步按同粒度聚合**:
   本例 `promo_month` 原本只按月聚合,若只改 `monthly` 不改它 → 每店每月都会拿到全公司当月推广费,
   **去费净利润反而更错**。
4. **行表达式逐列 COALESCE**:
   `"销售金额" - "退货金额" - "销售成本" + "实退成本金额"` 里任一列为 NULL → **整行被 SUM 丢弃**;
   公司级聚合时这行被"吃掉"看不出问题,拆到店级就暴露(本例增值税合计与 KPI 差 50 元)。
   写成 `COALESCE(col,0)` 逐列兜,再套 `COALESCE(SUM(...),0)`。

### 自检(写完必做,30 秒)

| 检查 | 判据 |
|---|---|
| **逐行加总 == 合计行** | 明细表所有行的销售额之和 = 合计行销售额(不重不漏) |
| **同页 KPI vs 明细表合计** | 两处数字**逐项**相等(不是"接近") |
| **跨页交叉** | 同一指标在另一张用不同数据源的页面上,逐店/逐月相等(本例:店铺明细页去费净利润 vs 明细表) |
| **抽查一行** | 随机挑一个 (月,店),用 SQL 直查该店该月的真实值 = 页面该行 |

## 1.15 dropdown 的 4 个坑(2026-10-22 商品分析 4 页 + 分类详情页实测)

### ⓪ ⭐ dropdown 有 **`order` prop**,可让下拉按业务指标排序(实测 2026-10-22)

```markdown
{% dropdown id="category_filter" data="top30_categories"
   value_column="virtual_category" order="recent_sales desc" /%}
```
wrapper 实际生成:
```sql
SELECT virtual_category AS "value", MAX(recent_sales) AS "recent_sales"
FROM ( <你的 SQL> ) WHERE (virtual_category IS NOT NULL)
GROUP BY virtual_category ORDER BY recent_sales desc LIMIT 10000
```
→ **Evidence 自动用 `MAX(<order 列>)` 解决 GROUP BY 问题** ⇒ 你的 SQL 里只要输出该列即可,
   下拉就按它排序(不需要自己 ORDER BY,也不需要前缀 hack / JS 去前缀)。
→ 典型用途:「按近 30 天销售额降序」⇒ **前 20 项即 TOP20,其余分类紧随其后**(滚动可见)。
   ⭐ **Core 0.10.1 的 dropdown 没有分组/子菜单能力** ⇒「底部『查看更多』→ 弹出二级菜单」**做不到**;
   `order` prop 是等价且更省的替代(实测 0 api 错误、下拉默认选中 = 排序第一项)。

### ① `select_first=true` **不会触发数据块重查** → 初次渲染 `{{ id.value }}` 是**空串**

```
初次请求 SQL:WHERE "店铺商品编码" = ''        ← 空串 → 查不到任何行 → 页面 "No results found"
用户手动选中后:WHERE 店铺商品编码 IN ('10039049066284')   ← 只有手动选才会重查
```
→ **写 SQL 过滤条件一律用 `{{ id.filter }}`,不要用 `{{ id.value }}`**:
`filter` 在未选中时渲染 `true`(展示全量,页面不空),选中后才收窄。

### ② 下拉 wrapper 用 `ORDER BY <value_column>` **覆盖**你 SQL 里的排序

```sql
-- 你的 SQL:ORDER BY SUM(sales_amount) DESC
-- 实际发出:SELECT 商品编码 AS "value" FROM (...) WHERE (商品编码 IS NOT NULL)
--           GROUP BY 商品编码 ORDER BY 商品编码 LIMIT 10000   ← 字典序!
```
→ 下拉**永远按 value_column 字典序**展示;"Top N" 的排序只能影响**选项集合**,不影响顺序。

### ③ `filter` 模板自带列名 → `value_column` 必须是**真实列名**

```sql
-- ❌ value_column="商品编码"(别名),而表里真实列是 "店铺商品编码"
--    → filter 渲染成 商品编码 = 'x' → 该列不存在 → 报错
-- ✅ value_column="店铺商品编码" label_column="商品编码"
--    → filter 渲染成 店铺商品编码 = 'x' ✓
```

### ④ 「恒等于 100% / 恒空」的指标 = 数据模型不可得,不是 bug

`mart_sku_monthly_12m` **只物化有销售的 SKU** ⇒
- `is_active` 71,166 行**全为 True** ⇒ 「动销率 = 动销 SKU / 总 SKU」的分母不存在 ⇒ **恒 100%**
- 「哑火新品(销售额 = 0)」⇒ **恒空**

→ 这类指标不要画成图(100% 的平线 / 空表 = 误导)。
处理:图表改画**可得的绝对值**(如「动销 SKU 数」),比率列留在表里供核对,并在交接文档里标为**口径待用户拍板**。

## 1.16 图表 tooltip 格式 / 折线标记 / fullscreen 裁剪(2026-10-22 分类详情页 A/B 实测)

### ① tooltip 数值格式**由 `y_fmt` 决定**(Evidence 不用 d3 语法)

| `y_fmt` | tooltip 实测输出 |
|---|---|
| `cny0` | `¥76,391`(整数 + 千分位 + ¥)✅ **金额图用这个** |
| `cny1` | `¥76,390.6`(1 位小数) |
| `pct1` | `77.0%` ✅ **占比图用这个** |
| `,.0f` | `222.7f` ❌ **`f` 是字面量**,不是 d3 的 format spec |
| 不写 | `0k` / `2.3M` 缩写 |

- ❌ `echarts_options={ tooltip={ valueFormatter="fmt:usd0" } }` → **无效**(实测与默认一致)。
  Evidence 二进制里虽有 `valueFormatter: 'fmt:usd1m'` 的示例串,但页面里不生效。
- ⇒ **要整数/百分比,只能换 `y_fmt`**(轴标签与 tooltip 一起变)。

### ② line series 默认 `symbol:"emptyCircle"` ⇒ 只写 `showSymbol=true` **看不见**

Evidence 的 line series 默认:`symbol:"emptyCircle", symbolSize:6, showSymbol:false`。
空心细圈在 942×215 的图上几乎不可见 ⇒ 用户会说"没有加标记"。

**实测(A/B 变体 + 像素法:series 色出现 ≥6px 竖直团块才算标记)**:

| 写法 | 结果 |
|---|---|
| `echarts_series_options={ showSymbol=true symbolSize=6 }` | ❌ 无(空心细圈,像素测不出厚度) |
| **`echarts_series_options={ showSymbol=true symbolSize=8 symbol="circle" itemStyle={ opacity=1 } }`** | ✅ **有实心标记** |
| `echarts_options={ series={ showSymbol=true } }` | ❌ 无(全局覆盖无效) |
| combo_chart 子图 `echarts_options={ showSymbol=true symbol="circle" itemStyle={opacity=1} }` | ✅ 有(ch21 §9 范式) |

### ③ fullscreen 后 tooltip「看不见」= 被容器裁掉,不是 hover 失效

- `:fullscreen` 的 CSS 是 `overflow: auto` ⇒ ECharts 的 tooltip(absolute 定位在图表容器内)
  超出容器边界就被裁 ⇒ **tooltip 确实触发了、DOM 里也有,但用户看不见**。
- ✅ 修法(页面级,不动全局 CSS):`echarts_options={ tooltip={ confine=true } }`
  ⇒ tooltip 永远留在绘图区内。实测 fullscreen 下 hover 靠右 75% 位置,tooltip 未超出容器。
- 兜底(可选,全站):CSS `.component-wrapper[data-component-type$="_chart"]:fullscreen { overflow: visible }`
  (本轮未做,页面级 confine 已解决本页)

**排障手法**:别只看 tooltip 在不在 DOM,**要量它的 rect 是否在 fullscreen 容器矩形内**
(`document.fullscreenElement.getBoundingClientRect()` vs tooltip 的 rect)。

## 1.17 批量改图表的纪律(2026-10-23 全站统一实战)

**批量脚本改 Markdoc 属性对象,必须先写校验器 + 全站验收**,纯 diff 看不出解析错误。

### ① 同一对象内**重复键** ⇒ Markdoc 报 `Node 'tag' is missing open` ⇒ 整图不渲染

```markdown
echarts_options={ itemStyle={ color="#2F7D63" borderRadius=3 }
                  itemStyle={ opacity=1 } }     ← ❌ 两个 itemStyle
```
⇒ 批量合并必须**按键合并**:已有 `itemStyle` 就把 `opacity=1` **并进已有对象**,不要新增同名键。
同理 `showSymbol` / `symbol` / `symbolSize` 已有则**替换值**。

### ② 合并子键时插入的是**值片段**而非 `key=value` ⇒ 产出裸 token

```
itemStyle={ color="#2F7D63" borderRadius=3 1}   ← ❌ 多了个 "1"
```
⇒ Markdoc 报 `Expected "=" or whitespace`。

### ③ 校验器(改完必跑)—— 抓上面两类错误
| 检查 | 实现 |
|---|---|
| 同一对象内重复键 | 扫对象体顶层 key,统计重复 |
| 裸 token | 把所有 `key=value` 片段挖掉后,剩余不应有非空白字符 |
| 花括号平衡 | `count("{")==count("}")` |
| 引号键例外 | `series_colors={ "蓝队"="#5278B2" }` 是 Evidence 特殊语法,元素间允许同名键 |

### ④ 改完必须**全站 Playwright 验收**(逐图 hover + canvas 存在性)
单页验证发现不了跨页崩溃;`NO_CANVAS` 与 500 是唯一信号。

### ⑤ 批量改之前先核对 prod 与仓库是否一致
`diff --strip-trailing-cr` 逐文件比;**prod 领先仓库的页面**(手工改过)必须**在 prod 上做增量替换**,
不能整文件覆盖 —— 否则抹掉 prod 独有内容。

### ⑥ 顺带查既有缺陷:笛卡尔 JOIN
`mart_sales_monthly` JOIN `mart_after_sales_monthly` **只按月份不按店铺** ⇒ 17 店 × 17 店放大
⇒ 表格金额虚高十几倍。**JOIN 条件要覆盖全部维度**,且修完用「另一页的同口径合计」交叉核对。

## 1.18 同页新图必须复用该页 KPI 的 CTE 与过滤;占比分母必须与同页其它比率一致(2026-10-23 销售总览页实证)

**事故**:同页新增「去费净利润 + 净利润占比」模块,用户报「同页面其它部分扣费利润占比 20% 以下,这个图 40% 以上」。

### 根因 1 · 引用了**不同的数据源**
新模块自己写了一套 `monthly` CTE,**漏了 `{{ shop_filter.filter }}`**
(KPI 卡顶层是 `类型='真实'` + `{{ shop_filter.filter }}` + 日期筛选)
⇒ **选店时 KPI 卡变、新模块不变** ⇒ 两个数对不上。

⇒ **规则:同页新图不要"另起炉灶"重写 CTE** —— 直接照抄该页 KPI 块的
`WITH` 子句与顶层 `WHERE`(包括 `shop_filter` 这类下拉过滤),只把 `GROUP BY` 改成月份维度。

### 根因 2 · 占比的**分母**与同页其它比率指标不一致
- 同页「扣费利润占比」= 扣费利润 / **实际销售额** ⇒ 20.32%
- 新图用了 扣费利润 / **净实销** ⇒ 23%~28%(偏高)

⇒ **规则:同页所有"占比"类指标必须用同一个分母**。加新比率图前,
先在同页找到同类指标的分母,照抄;否则用户会看到"同一页两个占比不一样"。

### 验收自查
- Σ各月值 == KPI 卡对应指标(本例 10,383,231 vs 10,383,232,差 ¥1 舍入)
- 抽查一个月份,与 SQL 直查逐位一致
- 明确记下**分母口径**,并与同页其它比率指标交叉核对

## 1.19 Markdoc 子图标签漏自闭合斜杠 ⇒ 整页内容丢失(2026-10-24 新品追踪页实测)

**症状**:页面 200、**0 个 api 错误**、第一张图甚至画出来了,但:
- 第 2 个 `##` 标题**整个消失**
- 该标题之后的所有表格 / 图表**全部不存在**
- 页面正文出现红色错误条:`Node 'combo_chart' is missing closing`

**根因**:combo_chart 的**子图标签漏了自闭合斜杠**:
```markdown
{% bar y="x" echarts_options={ ... itemStyle={ color="#1A3A6B" opacity=1 } } %}   ← ❌ 少了 /
```
正确写法(与 `{% bar_chart %}` 等无子图组件一样,末尾必须有 `/`):
```markdown
{% bar y="x" echarts_options={ ... itemStyle={ color="#1A3A6B" opacity=1 } } /%}  ← ✅
```

**为什么难发现**:① 页面 200;② **0 个 api 错误**(SQL 根本没执行);③ 画布里甚至有像素
⇒ 只查「200 + api 无错 + canvas 非空」会**漏过**;必须额外断言
**「页面 h2 数量 == 源码 h2 数量」+「表格数量 == 源码 table 组件数量」+「正文无 `Node 'xxx' is missing`」**。

> §1.1-4 说的是「**无子图**的组件必须自闭合」;本条是它的**姊妹坑**:
> 有子图的父组件(`{% combo_chart %}…{% /combo_chart %}`)里的**子图**同样要自闭合,
> 否则 Markdoc 认为父标签未闭合 ⇒ 父标签之后的内容全丢。

### 配套:Markdoc 语法自检器(改页必跑)

`deploy/scripts/mdcheck.py`(已入库)抓 7 类静默错误:
标签开闭配对 / 子图漏 `/` / 属性对象内重复键 / 属性区裸 token / 花括号与引号不配平 /
combo_chart 顶层写 `y=` / 图表 x-y-series 列名含中文 / SQL 块用 DuckDB 函数(strftime / url_encode)。
⇒ **部署前跑一次,比部署后看页面快一个数量级**。

## 1.20 Core 0.10.1 的 `x_sort` 只接受 asc/desc(修正 §1.6 的错误)

二进制实证:
```
x_sort:{type:String,required:!1,matches:["asc","desc"],
        description:"Sort order for x axis categories (asc or desc)",affectsQuery:!1}
```
⇒ **§1.6 表格里写的 `"data"` 和 `string[]` 在 Core 0.10.1 都不存在**(那是文档镜像/旧版本的说法)。

⇒ **时间序图表的正确做法 = `order="<x列> asc"`**:
`order` 是 `affectsQuery: true` 的 prop,会落在 chart wrapper 的**外层**(GROUP BY 之后)⇒ 顺序正确。
(可同时在 SQL 里写 `ORDER BY 1` 兜底;两者不冲突。)

## 1.21 表格里的**计数**列必须 `TO_CHAR(..., 'FM999,999')` 转文本

Evidence 会把数值列按默认数字格式渲染 ⇒ `4` 变成 **"4.00"**。
⇒ 计数 / 件数 / 月数这类列一律在 SQL 里转文本:
`TO_CHAR(COUNT(DISTINCT pid),'FM999,999') AS "上新商品 ID 数"`
(金额列同理,项目里一直用 `TO_CHAR(ROUND(x,0),'FM999,999,999')`。)

## 1.22 Evidence 的 **DOM tooltip 色块用主题 colorScale**,与画布上的 series 颜色不一致

- 画布里柱子/折线是你在子图 `itemStyle.color` 里指定的颜色(像素可验);
- 但 **tooltip 每一行前面的色块** 是 Evidence 自绘的 DOM 元素,颜色取**主题 colorScale**
  (柱=方块 / 折线=短线),**既不读 `itemStyle.color`,也不读 `chart_options.series_colors`**(实测加了无效);
- 图例是画在 canvas 内的(不查 DOM),颜色与 series 一致 ⇒ **只有 tooltip 色块对不上**。
⇒ 全站既有行为(销售总览页同样如此),不是本页缺陷;别为此去改 `series_colors`。

### tooltip 探测选择器(写验收脚本用)

ECharts/Evidence 的 tooltip div **没有 class**,靠 style 定位:
```js
const els = Array.from(document.querySelectorAll('div')).filter(e => {
  const st = e.getAttribute('style') || '';
  return /position:\s*absolute/.test(st) && /z-index:\s*999/.test(st)
         && e.offsetParent !== null && (e.innerText||'').trim();
});
```
(用 `className` 匹配 /tooltip/i **找不到**。)

## 1.23 Core 的 input 组件会**自动读 URL query param**(同名 id)—— 做页面下钻的关键能力

**二进制实证**(Core 0.10.1 的 input 基类):
```js
if (!O.dontUseQueryParam) {
  let H = url?.searchParams.get(this.id);   // ← 按组件的 id 取 URL 参数
  if (H) this.value = O.deserialize(H);
  else if (O.initialValue) this.value = O.initialValue;
}
```
⇒ **只要组件的 `id` 与 URL 参数同名,访问 `?pid=xxx` 就自动选中该值**;
   反之,组件值变化时会回写 URL(`searchParams.set(this.id, ...)`)。

**已实测可用**:`{% dropdown id="pid" ... %}` 与 `{% text_input id="q" /%}` 都会读 `?pid=` / `?q=`。
⇒ **报表页之间的下钻可以纯靠 URL 参数实现,不需要自建 JS 桥**。

### ⚠️ 两个配套约束

1. **filter 模板按「列名」渲染**:`{{ pid.filter }}` 渲染成 `pid = 'xxx'`
   ⇒ **SQL 里必须有一个真的叫 `pid` 的列**,且要在 WHERE 可访问的层级。
   SELECT 别名在同一层不可用于 WHERE ⇒ 要包一层派生表:
   ```sql
   prod AS (
     SELECT * FROM (SELECT "商品ID" AS pid, ... FROM sku_code.pro_code) t
     WHERE ({{ pid.filter }})
   )
   ```
2. ⚠️ **未选中时 `.selected` 渲染成「空」(不是 `''`)**,`.filter` 渲染成 `true`
   (2026-10-24 ERP 编码页实测推翻旧版「渲染 `''`」的错误结论):
   ```sql
   -- ❌ 错误: .selected 未选中渲染成空 → () <> '' → syntax error at or near ")"
   WHERE ({{ pid.selected }}) <> ''
   -- ✅ 正确: 只用 .filter(未选中 true, 选中 col='xxx', 永远合法)
   WHERE ({{ pid.filter }})
   ```
   想做「未选则空表」:给 dropdown 加 `select_first=true` + `order=`(默认选中第一项, filter 渲染
   `col='xxx'`),或接受「未选中显示全量」。**不要再用 `{{ x.selected }}` 做守卫** ——
   本项目 new-product-detail 的 `({{ pid.selected }}) <> ''` 是同一 pre-existing bug,
   之前验收靠 URL 参数掩盖了首屏 500。
   (`{{ pid.filter }}` 未选中时是 `true`,**不能**写成 `IN ({{ pid.filter }})` —— 会变成 `IN (true)` 语法错。)

## 1.24 表格 `link` 列:整行可点,且可按 URL 标记区分「新标签页 / 同标签页」

- Core 的 `link=` 列:URL 放在 class 含 `hidden` 的 `<td>`(DOM 可见),整行点击走 Svelte `goto`。
- 项目脚本 `evidence-core/static/table-link-blank.js` 在 capture 阶段拦截,改为 `window.open(url,'_blank')`。
- **2026-10-24 扩展**:URL 含 `_self` 标记时改为 **同标签页整页跳转**(`window.location.href = url`):
  ```js
  if (url.indexOf('_self') >= 0) { e.preventDefault(); e.stopPropagation(); window.location.href = url; return; }
  ```
  ⇒ 一套 JS 同时支持两种跳转:商品详情 → 新标签页;「月份 → 展开到日期」下钻 → 同标签页。
- ⚠️ 同标签页下钻**必须用整页跳转而非 Svelte `goto`**:goto 是客户端导航,
  同路由只改 query 时组件**不会重新初始化** ⇒ dropdown 读不到新 URL 参数。
  整页跳转代价是重新加载(本页 ~15s),但参数一定生效。
- ⚠️ `link` 只能整行可点,**无法只让某一格可点**(Core 不生成 `<a>` 标签)。

## 1.25 「月份下钻到日期」在 Core 0.10.1 的可行形态

`collapsible` prop 的官方说明:*"subtotal rows become clickable to expand/collapse their child rows.
**Only works when subtotals are enabled and there are dimensions.**"* ⇒ 它绑定的是
**pivot 模式**(table 的 `dimensions`/`measures`/`pivots`),**与本项目现有的 flat SQL 写法不兼容**
(flat SQL 表的列是算好的字符串,没法再按维度聚合出小计)。

⇒ **可行替代(本项目采用)**:同一页放两张表 —— 上表「按月」(每行带 `ym` 参数的链接),
下表「按日」(`WHERE ({{ ym.filter }})` 按月份过滤)。点月份行 → 同标签页跳转 → 下表切到该月。

## 1.26 PG `TO_CHAR` 数值格式:`FM999.0` 有两个坑

| 输入 | `FM999.0` | `FM9990.0` |
|---|---|---|
| 0.0 | **`.0`** ← 丢前导零 | `0.0` ✅ |
| 14.8 | `14.8` | `14.8` |
| 100.0 | `100.0` | `100.0` |
| 1234.5 | **`###.#`** ← 溢出 | `1234.5` ✅ |
| -34.5 | `-34.5` | `-34.5` ✅ |

⇒ **百分比一律用 `'FM9990.0'`**(`FM` 去掉前导零与填充,`9990` 给 4 位整数位 + 负号)。
计数/金额沿用 `'FM999,999'` / `'FM999,999,999'`。
(注:不带 `FM` 的 `'999'` 会留前导空格 —— 销售总览页明细表的历史写法,表里能看到 `"   46%"`。)

## 1.27 报表页的时间窗口锚点:用「数据最新日」而不是 `CURRENT_DATE`

订单数据的 ETL 有延迟,`CURRENT_DATE` 会让窗口末尾留一段没有数据的空窗
(实测:订单止于 2026-10-18,而服务器当天是 2026-10-24 ⇒ 末尾 6 天空窗)。

⇒ **在 SQL 里取数据自身的最新日做锚点**:
```sql
WITH anchor AS (SELECT MAX("订单日期")::date AS d FROM "order".order_dd_2026_processed)
...
WHERE o."订单日期" > a.d - 30
```
(用户 2026-10-24 拍板:近 30 天 = 相对数据最新日。)


## 1.28 Core 0.10.1 的 `combo_chart`:**`series` 分组不生效**(2026-10-24 分类详情页实测)

`combo_chart` 的 validate 里**有** `X2("series","data","select")`(合法 prop),
但**实现上不生效** —— 实测抓 `/api/query` 请求体,combo 给每个子图发的 SQL 是:

```sql
SELECT source_year_month AS "source_year_month", link_cnt AS "link_cnt" FROM ( <你的 SQL> )
```

**wrapper 只带 x 列和该子图的 y 列,不带 `series` 列** ⇒ 子图拿不到分组维度
⇒ 实际渲染:**柱**会按 series 值拆成多条(但名字全被子图 `name` 覆盖、且 `barGap="-100%"` 让它们完全重叠),
**折线只有 1 条**(不分组)。

⇒ **多系列 combo 在 Core 0.10.1 只有一种可靠做法:宽表 + 静态子图**
(每系列一列 y,每个子图写死一个 y 列 + 手写 `legend.data`,见 §9 / compare-teams 页)。
⇒ **系列数动态时(店铺数不定),只能用两张独立图**:
```markdown
{% bar_chart  data="X" x="ym" y="cnt"  series="shop_name" ... /%}   ← bar_chart 支持 series=
{% line_chart data="X" x="ym" y="rate" series="shop_name" ... /%}   ← line_chart 支持 series=
```
(`bar_chart` / `line_chart` 的 wrapper **会**带 series 列 ⇒ 分组正常。)

### 排障手法
抓 `/api/query` 的 request body,看 Evidence 实际发出的 SQL:
- **带** series 列 ⇒ 分组应该生效
- **不带** ⇒ 该组件不支持 series 分组 ⇒ 改用宽表或拆图
(比读文档快,且能直接看出是 wrapper 层的问题。)

### ⚠️ combo_chart **不支持 `tooltip_fields`**(2026-10-24 ERP 编码页实测)

`bar_chart` / `line_chart` 有 `tooltip_fields`(数组对象语法,可在 tooltip 加额外行),
**但 `combo_chart` 顶层和子图(bar/line)都没有这个 prop** ⇒ 写在 combo 上直接报
`Invalid attribute: 'tooltip_fields'`(页面其它部分正常,只有该图不渲染)。

**替代**:combo 顶层的 `point_title="<列名>"`(单值点标签,显示在 tooltip 顶部):
```sql
-- SQL 里拼一个标签列(虚拟分类 + 系统商品名称)
MAX(virtual_category) || ' · ' || COALESCE(MAX(product_name), '') AS point_label
```
```markdown
{% combo_chart data="X" x="erp_code" point_title="point_label" ... /%}
```
实测 tooltip 顶部显示 `腿垫 · 风尚志-医用体位垫【腿垫+枕头】黑色款`。
⚠️ `point_title` 是**单值**(只能一个列/表达式),不能像 tooltip_fields 那样多行;
两个字段要同时展示就拼成一个字符串列。

### combo_chart 的 `order` 排序(2026-10-24 ERP 编码页实测)

combo 顶层 `order="<y列> desc"` **生效且不报错**(子图 bar/line 没有 order prop,
只能走顶层)。实测毛利 combo 顶层 `order="gross_profit desc"` → 柱按毛利降序,
0 个 api 错误(与 tooltip_fields 不同,order 是 combo 合法顶层属性)。


## 1.30 「点图上的柱 → 联动筛选另一个板块」怎么实现(2026-10-25 ERP 编码页实测)

**Core 0.10.1 没有原生 chart click → filter**(只有 input 组件的 `filter-set`/`filter-create`)。
⇒ 唯一可行路径 = **JS 注入**(nginx sub_filter), 借「dropdown 选中会过滤下游 SQL」这个机制中转。

### ① dropdown 选中 = 过滤下游 + 回写 URL(实测)

选中某个 option 后, Core 立刻做两件事:
- 下游 SQL 的 `{{ id.filter }}` 渲染成 `col='选中值'` ⇒ **依赖它的组件自动重查**
- URL 回写 `?id=选中值` ⇒ **刷新后状态保持**

⇒ **只要能让 dropdown 的值变成 X, 就能让别的图表按 X 过滤。**

### ② bits-ui Select 的事件契约(实测)

| 事项 | 结论 |
|---|---|
| 监听的事件 | **`pointerdown` → `pointerup` → `click`**(**没有 mousedown**) |
| 合成 `el.click()` | ❌ **无效**(不触发选中) |
| 合成 pointer 序列 | ✅ **有效**(实测 URL 与下游 SQL 都变了) |
| 真正要点的元素 | 内部 `<button data-slot="select-trigger">` —— 外层 `[data-slot="context-menu-trigger"]` **派发事件打不开 popover** |
| 最佳定位锚点 | `<label for="<input 的 id>">` —— Evidence 给每个 input 组件渲染的 label, 带 `for=id` |

```js
// JS 合成选中(dropdown 内的某个 option)
var base = {bubbles:true, cancelable:true, composed:true, view:window,
            button:0, isPrimary:true, pointerId:1, pointerType:'mouse'};
el.dispatchEvent(new PointerEvent('pointerdown', Object.assign({}, base, {buttons:1})));
el.dispatchEvent(new PointerEvent('pointerup',   Object.assign({}, base, {buttons:0})));
el.dispatchEvent(new MouseEvent('click',        Object.assign({}, base, {buttons:0})));
```

### ③ ⚠️ dropdown 的选项列表是**虚拟滚动**(只渲染可视区 ~14 项)

- 实测: 1,389 个选项 → `scrollHeight` 44,448px, DOM 里只有 ~14 个 `[role="option"]`
- ⇒ 目标 option 不在可视区时, `querySelectorAll` 找不到
- ⭐ **解法: 临时把滚动容器撑高** ⇒ 虚拟列表一次性渲染全部 option ⇒ 直接定位
  ```js
  sc.style.height = (sc.scrollHeight + 240) + 'px';
  sc.style.maxHeight = (sc.scrollHeight + 240) + 'px';
  sc.scrollTop = 0;
  ```
  实测 64 项 → 全部渲染; 1,389 项同样可行。**选中后 popover 立即关闭, 用户几乎无感。**

### ④ 从 click 事件拿到「被点的那个数据点」

ECharts 的 tooltip 在 mousemove 时就渲染了 ⇒ **click 时 tooltip 还在**, 直接读:
- tooltip 容器选择器(ECharts 无 class, 靠 style):
  ```js
  /position:\s*absolute/.test(st) && /z-index:\s*999/.test(st) && e.offsetParent !== null
  ```
- **bar_chart**: tooltip 第一行就是 x 值(= 分类/编码)
- **combo_chart**: 第一行是 `point_title`, 编码在 **`'Erp Code'` 标签的下一行**
  ⇒ 通用解析: 先找 `'Erp Code'`/`'ERP编码'` 行, 没有则取第一行
- 兜底: 快速点击(没先 hover)时 tooltip 还没渲染 ⇒ click 里补派发一个 `mousemove`

### ⑤ 单向联动的实现要点

让 A 图表驱动 B 板块: **A 的 SQL 完全不引用 B 用的那个 filter** ⇒ 反向天然不成立。
(本页: Top30 三张图的 SQL 不引用 `m_code_filter` ⇒ 点 Top30 只影响月度趋势, 反之不会。)

### ⑥ 完整落地形态(本项目实例)

- JS 文件: `deploy/evidence-core-migration/static/erp-code-link.js` ⇒ 部署到 `/var/www/evidence/static/`
- nginx 注入(`/etc/nginx/sites-enabled/sales-bi` 的 `location /` 块):
  ```
  sub_filter '</head>' '...<script src="/static/erp-code-link.js?v=1" defer></script></head>';
  ```
  ⚠️ 改 nginx 前后: 备份 → 精确 patch(用 python 改, 断言只命中 1 处)→ `nginx -t` → `nginx -s reload`
- ⚠️ **JS 全站注入 ⇒ 必须自带页面特征检测**(本项目: 三个 Top30 的 h2 标题都在才启用), 否则影响所有页面
- 验收: `deploy/scripts/verify_erp_code_link.py`(点柱 → 断言下游 SQL 变化 + 断言上游图表 **0 次重查**)


## 1.29 顶部导航(masthead)由 `pagesManifest.json` 驱动,**不是**页面 frontmatter(2026-10-24 实测)

顶部导航由 `evidence-core/static/pentagram-nav.js` 从 `/api/pagesManifest.json` 动态生成,
**两层结构各有各的驱动源**:

| 层级 | 驱动源 | 改哪里 |
|---|---|---|
| **组顺序**(大盘/商品分析/店铺运营/售后风险/数据治理) | `pentagram-nav.js` 的 `GROUP_ORDER` 常量 | 改 JS |
| **组内顺序**(组里哪页排前面) | **manifest 里 `children` 的原始顺序** | 改 manifest JSON |

### ⚠️ `sidebar_position` 对顶部导航**完全无效**

`pentagram-nav.js` 的 `collect()` **没有把 `frontMatter` 传出去** ⇒ `byPosition()`
永远拿到默认值 `999` ⇒ 所有页面并列,最终顺序退化成 manifest 的 `children` 顺序。

> 实测佐证:大盘 3 页 frontmatter 写的是 `10 / 20 / 30`,但导航实际显示仍是 manifest 原序。
> ⇒ **改 frontmatter 排导航 = 白改**(sidebar 侧边栏的排序另算,不是这套)。

### ⚠️ manifest 有**两份**,必须同步改

1. `/var/www/evidence/api/pagesManifest.json` —— 运行时 API(hydrate 后用)
2. 每个 build 产物 `index.html` 内嵌的
   `<script type="application/json" data-sveltekit-fetched data-url="/api/pagesManifest.json">`
   —— 外层结构 `{status, body}`,`body` 是 manifest 的**字符串**

只改其一 ⇒ **首屏 HTML 与 hydrate 后不一致**(用户看到"刷新一下顺序就变")。

### ⚠️ manifest 是 build 产物 ⇒ 下次 build 会覆盖

补丁脚本**必须**接进 `build-and-deploy.sh` 的 deploy 后步骤,否则一次 build 就把导航改回默认。
(Sales BI:`deploy/scripts/patch_nav_structure.py`,幂等,prod 副本在
`/opt/sales-bi/evidence-core/patch_nav_structure.py`。)

### ⚠️ 新建页面 .md 不会自动进 manifest

新页 `.md` 直接访问 URL 正常(200,Evidence dev server 直接渲染),但**不进 manifest**
⇒ **不出现在导航** ⇒ 上线新页必须同时补 manifest(或跑补丁脚本)。

### 验收

不要只看页面 200。Playwright 断言两件事:
1. **组列表**(确认要删的组没了、该有的组都在)
2. **组内子菜单顺序**(展开某组,断言子项文本序列)


## 2. 表格组件契约(Evidence Core 0.10.1)

### 2.0 `link=` 拼 URL:PG 没有 `url_encode`(2026-10-22 商品分析页实测)

`table ... link="detail_url"` 的链接列,在 SQL 里拼 URL 时:
- **`url_encode()` 是 DuckDB 函数,PostgreSQL 没有** → `function url_encode(text) does not exist` → **/api/query 500**。
  (页面照常 200,只有这一个 block 的查询挂掉;若该 block 是表格,表格就整块消失。)
- PG 侧的最小可用写法(只转义会破坏查询串的字符,中文交给浏览器自动编码):
  ```sql
  concat('/other-page?category=', replace(virtual_category, '&', '%26')) AS detail_url
  ```
- 需要完整百分号编码时:`encode(convert_to(x, 'UTF8'), 'escape')` —— 但它会把中文也转义成
  `%E5%95%86...`,可读性差且部分前端不解码,**报表页优先用上面的 `replace` 最小转义**。

### 2.0-bis 表格 link 列:无 target 控制,想 `_blank` 用 JS 注入(2026-10-22 实测)

- **Evidence table 的 `link=` 列**渲染成:① 一个 class 含 `hidden` 的 `<td>`(**文本 = URL**,DOM 可见)
  ② 最后一个 `<td>` 放 `chevron-right` SVG 箭头。**整行点击走 Svelte `goto`(当前标签页跳转)**。
- **没有 `<a>` 标签、没有 `target` prop**(二进制里 `target:"_blank"` 只出现在 BunError 组件)。
- 想「点击新标签页打开」→ 用 JS 注入(nginx sub_filter,与 chart-fullscreen.js 同款):
  ```js
  // evidence-core/static/table-link-blank.js
  document.addEventListener('click', function (e) {
    var row = e.target.closest('tbody tr');
    if (!row) return;
    var linkTd = row.querySelector('td.hidden');   // URL 在这
    if (!linkTd) return;
    var url = (linkTd.textContent || '').trim();
    if (url.charAt(0) !== '/') return;
    e.preventDefault();
    e.stopPropagation();                            // 阻止 Svelte goto
    window.open(url, '_blank');
  }, true);                                          // capture 阶段
  ```
- **验收**:`expect_popup` 捕获新标签 URL = link 值;主页面 URL 不变;`ctx.pages` 数 = 2。

- **`page_size`**:默认 10,max 200,整数。源码 `XX2("page_size",{min:1,max:200,integersOnly:!0})`。
- **内置「Expand table」按钮**(表格右上角,`aria-label="Expand table"`,展开后变 `"Exit fullscreen"`):
  - 展开逻辑:`pageSizeOverride = Math.max(1, Math.floor((window.innerHeight - 140) / 28))`
    → **每页行数按视口高度自适应**(行高 28px);
  - 同时容器从 `h-full min-h-0 overflow-auto` 改成 `overflow-visible` → **表格内部不滚,改由整页滚**;
  - `pageSizeOverride ?? page_size` → 展开态**覆盖** `page_size` prop。
  - ⚠️ 总行数 > 视口可容纳行数时,展开态**仍分页**,不是「行数不限」。
- **prop 全清单**(没有「另开详情页 / 全屏弹窗」这类 prop):
  `data` `measures` `dimensions` `pivots` `row_conditional_colors` `filters` `fmt`
  `page_size` `freeze_columns`(0..10) `search` `format_titles` `wrap_titles` `wrap`
  `row_shading` `row_lines` `link`(列名含 URL → 整行可点跳转)`show_link_column`
  `repeat_values` `collapsible`。
- **想要「点击跳转另一个页面的全量表」只能用 `link` prop + 自己建页**;想在原地放大用内置 Expand 按钮。

## 3. 报表页内容规则(用户铁律,2026-10-21)

**报表页只放业务内容。** 「怎么改报表」的说明一律不进页面:

| 内容 | 放哪 |
|---|---|
| 颜色变量 / 主题 token 真源 | `deploy/config/<页面>.palette.json`(脚本读它,页面不写) |
| 组件契约 / 踩坑 / 操作指引 | memory + handoff(本 skill 的 ch21 也是这种去处) |
| 口径与数据源说明 | 交接文档;页面里只在必要处一句话带过 |

- 颜色变量表若原本在页面里 → **移出到 json 文件**,并让「同步脚本」和「像素级回归脚本」都改读该文件
  (否则删了页面小节 = 自动化静默失效,回归脚本直接红)。
- **不要顺手补「说明性小节」**。写页面改动前先问:这段是给业务看的,还是给开发看的?给开发看的 → 不写进页面。

## 4. 改报表页的标准流程(可复用 SOP)

1. **查契约**:先 grep 二进制(§0),再动页面;不确定的 prop 别猜。
2. **备份 prod**:`sudo cp <prod 页面> /opt/sales-bi/.evidence-backup/<日期-主题>/`
   ⚠️ **备份必须对应「要回到的状态」** —— 回滚前先 `diff` 备份与当前 prod,确认备份是「要回到的那一版」。
   (本项目踩过:把「改动前的备份」当成「改动后的备份」回滚 → 白做一遍。)
3. **改本地真源** → 算 diff → 部署(prod 是 bind-mount,改 .md 即时生效,**无需 build**)。
4. **改 .md 用 Python 写**:`io.open(path, "w", encoding="utf-8", newline="\n")`。
   仓库 `core.autocrlf=true`,**改完别 `git checkout`**(会把 .md 转 CRLF,prod diff 全飘红)。
5. **验收 3-piece**(见 §5)。
6. **只 add 本次改的文件**(仓库常有 100+ 未跟踪遗留文件,`git add -A` 会误提交)。

## 5. 验收 3-piece(改完必跑)

| # | 检查 | 手段 |
|---|---|---|
| 1 | **SQL 直查 prod PG** | 把页面 SQL block 抽出来,替换 `{{ date_filter.between }}` / `{{ dropdown.filter }}` 后直连 PG 跑;核对行数 + 关键值 |
| 2 | **Playwright 真渲染** | 断言:canvas 数 ≥ N 且 dataURL 非空 / 表格行数 > 0 / 合计行在最后一页 / 无 "no data" / 无 page error / `/api/query` 无 4xx-5xx / 筛选器切换后内容或 canvas 变化 |
| 3 | **像素采样** | 柱=实心填充 → 精确 RGB 匹配;细折线有抗锯齿 → 容差(±46 且像素数 ≥12) |

补充(本轮新增,都很便宜):
- **图例可点验证**:点图例图标后「绘图区该色像素数」应归零 → 证明 legend 项名与 series 仍匹配。
- **截图肉眼**:图例形状/配色/排版这类**语义**问题,像素采样判不了 → 截 canvas 顶部区域 PNG 直接看。
- **变量漂移检测**:任何「写死字面量的同步脚本」都要做「注入漂移 → 检测 exit 1 → 修复 → 逐字节核对」往返。
  ⚠️ 正则用**具名组**(`prefix`/`suffix`/`team`),别用位置组 —— 位置组写错会产出损坏内容,
  而损坏后正则不再匹配 → **漂移检查假报通过**(本项目踩过:`color="#5278B2蓝队}`)。

## 6. A/B 变体同页实验法(改图表样式时省 2-3 轮)

一次 deploy 拿到多个数据点:
1. **同一页的不同图各放一个变体**(图 A 试变体 1 / 图 B 试变体 2 / 图 C 试变体 3),留 1 张当对照;
2. 一次部署 → 截图 → 逐张读图对比 → 挑赢的那个 → 再统一到全部图;
3. 变体互相独立:一个变体语法错只会**整图不渲染**,页面其它部分照常 → 不会连坐;
   但要能快速识别「哪张图挂了」(截图脚本按 title 定位,**title 找不到 = 该图没渲染**);
4. 变体语法要用**会改变视觉**的写法(如换 `icon`),否则「无变化」分不清是「没生效」还是「生效了但看起来一样」。

## 7. 本章常见踩坑速查

| 现象 | 根因 | 修 |
|---|---|---|
| 整张图不渲染,页面其余正常 | combo 子图 `name` 写成子图属性 / `legend.data` 用换行分隔 / 组件漏自闭合 | 见 §1.1 第 2 条、§1.4、§1.1 第 4 条 |
| 整图 500 | 列名含中文 | 列名 ASCII,中文放数据值(§1.1 第 3 条) |
| 表格/某 block 500,页面其它部分正常 | SQL 里用了 `url_encode()`(DuckDB 函数, PG 没有) | 换 `replace()` 最小转义(§2.0) |
| Playwright `innerText` 读到空串,但 DOM 里明明有文字 | 元素在 `<details>` 折叠容器里(不可见 → innerText 为空) | 改用 `textContent` |
| 图表加了 JS 回调或数组属性后整图消失,页面印 `Expected ... but "(" found` | Markdoc 属性解析器**只接受简单值/简单对象**,不支持函数与数组字面量 | 改用「多 series + SQL 输出标记列」方案(§1.12) |
| 超轴范围的点完全不画 | ECharts 默认 `clip: true` 裁掉超轴 symbol | 标记列取轴内语义边界值 + 轴留余量(§1.12) |
| combo/瀑布图没有白色卡片背景,bar/折线图有 | 项目自定义 CSS `sales-bi.css` 的 `data-component-type` 选择器漏了类型 | 补进同一组选择器(§1.9) |
| 像素法数出 32 根柱(实际 30 根) | 抗锯齿边缘产生 ±1 的颜色变体,精确匹配把一根柱切成多段 | 颜色匹配加容差 ±3(combo 图再指定主柱色) |
| 校验 toolbox「不压图」用「非白像素数」判定 | toolbox 图标本身就是深灰非白像素 → 永远失败 | 改判「右侧最外 3% 带不得出现主色/折线色」 |
| SQL 里 `SUM(x)` 后 `COUNT(SUM(x))` 报 `aggregate function calls cannot be nested` | PG 不允许嵌套聚合 | 先 `GROUP BY` 聚成子查询,再在外层聚合 |
| 图例颜色/形状与线不符 | 用了 Evidence built-in 图例 | `legend=false` + 原生图例(§1.3 第 8 条) |
| 图例只有形状没有线 | legend data item 无 lineStyle.stroke | 用 `legend.data` + `path://`(§1.4) |
| 图例点不动 | `name` 写成短名,与 series 名对不上 | 用完整系列名(§1.3 第 10 条) |
| 校验脚本误判「某色缺失」 | 用「标题向上找祖先」取 canvas | 改成「标题之后第一个 canvas」(§1.5 第 13 条) |
| 点了图例没反应 | 猜坐标点空了 | canvas 内按像素聚类定位(§1.5 第 14 条) |
| 2 bar 叠成一根 | `stacked` 默认 true | 显式 `stacked=false`(§1.1 第 1 条) |
| 表单图例点击后表格没变 | 误判筛选器只驱动图表 | 改成断言 body 文本变化(该页 team_filter 只驱动表格) |
| dropdown 过滤报 `syntax error at or near "<列名>"` | `{{ id.filter }}` **自带列名**,自己又写了一遍列名 | 改 `WHERE ({{ id.filter }})`,不加列名(§1.13) |
| 下拉默认选中的不是「第一项」(如不是销量最高) | dropdown 选项被 wrapper 强制 `ORDER BY <value_column>`(字母序),且 dropdown 无 `order` prop | 只能接受字母序第一项;要按业务排序需另设排序键(§1.13) |
| 页面正文印 `function strftime(...) does not exist` | SQL 用了 DuckDB 的 `strftime` | PG 用 `to_char(x,'YYYY-MM')`(§2.0 同类:DuckDB 函数在 PG 不存在) |
| 明细表某行数值远大于该店真实值(几百倍),且店名逐行跳变 | `GROUP BY` 丢了店铺维度 + `MIN("店铺")` 补标签 → 行数据与标签粒度不同 | `GROUP BY 1, 2` 补回维度;标签取本行自己的店;按维度分摊的费用列同步改粒度(§1.14) |
| 明细表合计行与同页 KPI 卡差几十~几百元 | 行表达式含 NULL 列 → 整行被 SUM 丢弃(公司级被吃掉、店级暴露) | 行表达式逐列 `COALESCE(col,0)`(§1.14) |
| tooltip 里数值带字面 "f"(如 `222.7f`)/ 要整数或百分比 | `y_fmt` 不是 d3 格式串,`,.0f` 的 `f` 被当字面量 | 金额图 `y_fmt="cny0"`(¥76,391)、占比图 `y_fmt="pct1"`(77.0%);`valueFormatter="fmt:..."` **无效**(§1.16 ①) |
| 折线图"没有标记" | line series 默认 `symbol="emptyCircle"`(空心细圈)→ 只写 `showSymbol=true` 看不见 | 补 `symbol="circle"` + `itemStyle={ opacity=1 }` + `symbolSize=8`(§1.16 ②) |
| fullscreen 放大后 hover 看不到数据 | tooltip 被 `:fullscreen` 容器的 `overflow:auto` 裁掉(hover 其实触发了) | `echarts_options={ tooltip={ confine=true } }`(§1.16 ③) |
| 改表格表头背景色:color/border 生效了,background 没生效(甚至变成「白字+浅底」看不见) | **Core 给表头 th 加了 `.bg-background` 类**,`main, main.bg-background, .bg-background` 的特异性(0,1,0)**高于** `table thead th`(0,0,3)⇒ 浅底赢 | 补高特异性选择器 `table thead th.bg-background` / `table thead th[class*="bg-background"]`;改样式前先 dump 匹配该元素的**所有**规则(含 @layer/@media 内)并对比 `getComputedStyle` 实际值 |
| 改 `/static/sales-bi.css` 后浏览器还是旧样式 | — | 该 location 是 `expires -1` + `no-store` ⇒ **改完即生效,不需要 cache-buster / 动 nginx**(别去跑 patch_css_cache_buster) |
| 页面 "No results found",但下拉明明选了默认值 | `select_first=true` 不触发数据块重查,初次 `{{ id.value }}` 是空串 | 改用 `{{ id.filter }}`(未选 = true)(§1.15 ①) |
| 下拉里明明按 Top N 排序,实际是字典序 | wrapper 用 `ORDER BY <value_column>` 覆盖你的排序 | 接受(顺序改不了),或换成别的选择器(§1.15 ②) |
| 图表/表格是 100% 平线,或某个板块永远空 | mart 只物化"有销售的实体"→ 分母不存在(如动销率、哑火新品) | 图表改画可得绝对值,比率留在表里 + 交接文档标口径待拍板(§1.15 ④) |
| 下拉想按销售额/业务指标排序,自己写的 ORDER BY 被覆盖 | wrapper 固定 `ORDER BY <value_column>` | 用 dropdown 的 **`order` prop**(wrapper 会生成 `MAX(<列>)` + 按它排序)(§1.15 ⓪) |
| 表格里同店同区间,两张表(按月 / 按店汇总)数值对不上 | 一张按月分摊、一张按区间一次性分摊 → 比例不同 | 汇总表复用月级分摊结果再 GROUP BY(逐行相加完全一致) |
| 页面 200、0 api 错误、图也画出来了,但第 2 个标题之后的内容全没了,正文有 `Node 'combo_chart' is missing closing` | combo_chart 的**子图标签漏自闭合斜杠**(`}} %}` 应为 `}} /%}`)⇒ Markdoc 认为父标签未闭合 ⇒ 其后内容全丢 | 补 `/`;改页前跑 `mdcheck.py`;验收要断言「h2 数量 == 源码 h2 数量」「table 数 == table 组件数」 |
| 时间序图表的月份顺序被打乱 | Core 0.10.1 的 `x_sort` **只接受 asc/desc**(§1.6 文档写错了) | 改用 `order="<x列> asc"`(affectsQuery,落 wrapper 外层) |
| 表格里的计数列显示成 "4.00" | Evidence 对数值列用默认数字格式(2 位小数) | SQL 里 `TO_CHAR(COUNT(...),'FM999,999')` 转文本 |
| JS 派发 `el.click()` 选不中 dropdown 选项 | bits-ui Select 监听 **pointerdown/pointerup/click**, 合成 click 无效 | 派发完整指针序列(见 §1.30 ②) |
| JS 给 `[data-slot="context-menu-trigger"]` 派发事件, popover 打不开 | 真正可点的是内部 `<button data-slot="select-trigger">`;外层只是容器 | 定位到 button 再派发;或用 `<label for="<input id>">` 精确定位(§1.30 ②) |
| dropdown 展开后 `querySelectorAll('[role=option]')` 只有十几个, 找不到目标项 | 选项列表是**虚拟滚动**(只渲染可视区) | 临时撑高滚动容器 ⇒ 一次性渲染全部, 再定位(§1.30 ③) |
| JS 改 URL + `history.replaceState` + 派发 popstate, 下游图表不重查 | Core 的 input 组件**只在初始化时**读 URL param, 不响应 URL 变化 | 改用「驱动 dropdown 选中」路径(§1.30 ①) |
| 图表点击联动: 找不到「被点的那个数据点」 | click 事件里没有数据信息,但 **ECharts tooltip 在 mousemove 时已渲染** | click 时读 tooltip DOM;combo 的编码在 'Erp Code' 标签的下一行(§1.30 ④) |
| 验收脚本 hover 长页面底部的图没反应 | 画布在**视口外**, 鼠标坐标是视口坐标 | 先 `scrollIntoView` 再取 rect 算坐标(§1.30 ⑥) |
| tooltip 行前面的色块颜色和柱子颜色对不上 | Evidence 的 DOM tooltip 色块取主题 colorScale,不读 `itemStyle.color` / `series_colors` | 全站既有行为,忽略;图例(canvas 内)颜色是对的 |
| 报表页之间做下钻(点 A 页某行 → B 页显示该行数据) | 以为要自建 JS 桥读 URL | Core 的 input 组件**按自己的 `id` 自动读 URL query param**(§1.23);`?pid=xxx` 即自动选中 |
| `{{ id.filter }}` 报 syntax error / 查不到数据 | filter 渲染成 `pid = 'xxx'`,但 SQL 里没有叫 `pid` 的列;或 SELECT 别名同层不可用于 WHERE | 包一层派生表让 `pid` 成为真实列(§1.23) |
| 下拉未选中时报表整块 500 | 写了 `IN ({{ id.filter }})` → 未选中时变成 `IN (true)` | 用 `WHERE ({{ id.filter }})`;要空表再加 `AND ({{ id.selected }}) <> ''` |
| 想让「月份行」点一下就地展开成日期行 | Core 的 `collapsible` 只在 pivot 模式(subtotals+dimensions)下可用,与 flat SQL 表不兼容 | 同页两张表 + URL 参数驱动(§1.25) |
| 表格行点击想「同标签页」跳转,却总是开新标签 | `table-link-blank.js` 统一 `window.open(url,'_blank')` | URL 加 `_self` 标记,JS 分流(§1.24) |
| 同标签页跳转后下拉没跟着变 | 用了 Svelte `goto` 客户端导航,同路由只改 query 时组件不重新初始化 | 改用 `window.location.href` 整页跳转(§1.24) |
| 百分比列显示 `.0%` / `###.#` | `TO_CHAR(x,'FM999.0')`:0 丢前导零,>999 溢出 | 用 `'FM9990.0'`(§1.26) |
| 「近 30 天」窗口末尾有一段没数据 | 用了 `CURRENT_DATE`,但订单 ETL 有延迟(实测数据止于 10-18,当天 10-24) | 用数据自身的最新日做锚点 `MAX("订单日期")`(§1.27) |
| combo_chart 写了 `series="shop_name"`,柱拆成了多条但折线只有 1 条 | Core 0.10.1 的 combo wrapper **不带 series 列**(实测 request body)⇒ 子图拿不到分组维度 | 拆成 `bar_chart` + `line_chart` 两张图(各自支持 series=),或改宽表+静态子图(§1.28) |
| 同一个组件系列名全一样(如都叫「在售链接数 - Link Cnt」) | series 分组后子图的 `name` 会覆盖掉自动生成的系列名 | 别依赖 combo 的自动命名;用 bar_chart/line_chart 的 series= 分组 |
| 改了页面 frontmatter 的 `sidebar_position`,顶部导航顺序没变 | 导航的组内顺序由 **manifest 的 children 顺序**决定;nav.js 的 `collect()` 不传 frontMatter ⇒ `sidebar_position` 无效 | 改 `pagesManifest.json`,并同步 API 副本 + 所有 index.html 内嵌副本(§1.29) |
| 导航改完「刷新一下顺序就变」 | 只改了 `api/pagesManifest.json`,没改 index.html 内嵌副本(首屏用内嵌、hydrate 用 API) | 两处都要改(§1.29) |
| 新建的页面 .md 不出现在导航里 | 新页不进 manifest(URL 能直接访问 ≠ 进导航) | 上线时补 manifest 节点(§1.29) |
| 下次 build 之后导航顺序被重置 | manifest 是 build 产物,build 会覆盖手工补丁 | 补丁脚本接进 `build-and-deploy.sh` 的 deploy 后步骤(§1.29) |

## 8. 日期筛选器(range_calendar)SQL 契约 —— 四态安全(2026-10-21 店铺明细页实测)

`range_calendar` 的 SQL 模板有 **4 种状态**,写错模板 → 用户每次改日期就闪 500。

| 模板 | 两端都有 | 只有起始日 | 只有结束日 | All Time |
|---|---|---|---|---|
| `{{id.between}}` | `BETWEEN A AND B` | `>= A` | `<= B` | `IS NOT NULL` |
| `{{id.filter}}` | `col >= A AND col <= B` | `col >= A` | `col <= B` | `true` |
| `{{id.start}}` / `{{id.end}}` | `DATE '...'` | 另一端 = **空串** | 另一端 = **空串** | 都是空串 |

**铁律:日期过滤只用 `{{id.between}}`(或 `{{id.filter}}` + `value_column`),不要用 `start`/`end` 拼自己的 SQL。**

- `between` 的左操作数由你自己给 → `WHERE <列/表达式> {{date_filter.between}}`。
  官方 description 原话:`where="date {{id.between}}"` matches all rows and never breaks。
- `start`/`end` 在「只选了起始日」时渲染成**空串**(不是 NULL)→ 拼进 SQL 变成
  `CAST( AS DATE)` → `syntax error at or near "AS"` → **/api/query 500**。
  ⚠️ 这不是边缘情况:用户在日历里点第 1 个格子(只定起始日)就会立刻触发。
- `all_time_range` 属性默认 `unbounded` → `between` 发 `IS NOT NULL`(全量行);
  设成 `none` 才是空串(那才是残缺 SQL 的来源)。
- postgres dialect 的 dateLiteral = `DATE '2026-01-01'`(不是 `TO_DATE(...)`)。

**月粒度列(text 'YYYY-MM')接日期筛选**:先把文本月转成该月 1 号再比日期 ——

```sql
AND TO_DATE(source_year_month || '-01', 'YYYY-MM-DD') {{ date_filter.between }}
```

- 不能写 `source_year_month BETWEEN DATE '...' AND DATE '...'`(text vs date → 类型错)。
- 语义 = **该月的 1 号落在所选区间内**(月粒度数据的自然口径)。
- `value_column` 可以不设,自己引用列即可(官方 description: "Leave it unset to instead
  reference the column yourself with `{{id.between}}`")。

**排障手法**:`/api/query` 的请求体能看到模板替换后的**真实 SQL** → 一眼看出是
空值还是类型错(比看页面报错快得多)。

## 9. 多系列折线(combo_chart)范式 —— 4 店 4 形状 + 原生图例(2026-10-21 店铺明细页)

照搬「蓝红对比」页的折线图写法,给 N 个分类各画一条线:

```markdown
{% combo_chart
    data="wide_table"          ← 每类一列的宽表(见下)
    x="source_year_month"
    y_fmt="pct1"               ← 比率列是 0~1 小数, pct1 自动 ×100
    y_axis_options={ title="退款率" }     ← 键是 title 不是 name(§1.5 第 12 条)
    legend=false               ← 切 ECharts 原生图例
    echarts_options={
        legend={
            show=true
            data=[
                {name="JD仁禾 - Refund Rate Jd" icon="path://M0,6 L25,6 L25,8 L0,8 Z M5.5,7 C5.5,3.134 ... Z" itemStyle={ color="#9A4541" }},
                ...
            ]
        }
    }
%}
    {% line y="refund_rate_jd" echarts_options={
        name="JD仁禾"            ← 子图名必须在 echarts_options 内部(§1.1 第 2 条)
        showSymbol=true
        symbol="circle"          ← 每系列独立形状, 用来在黑白/深浅色下区分
        symbolSize=8
        itemStyle={ color="#9A4541" opacity=1 }    ← opacity=1 必加(否则标记被隐藏)
    } /%}
    ...
{% /combo_chart %}
```

- **图例 name 必须是完整系列名** = `${子图 name} - ${y 列别名 humanize}`。
  y 列别名必须 ASCII(§1.1 第 3 条)→ 图例文字必然是「中文名 + 英文后缀」,**平台固有,不是 bug**
  (例:`"JD仁禾 - Refund Rate Jd"`;改 y 列名 → 图例名同步变)。
- **`path://` 图例图标**:bbox 设计成恰 25×14 → 缩放比 1;一条 path 画两个子路径 = 底线矩形 + 标记形状。
  形状 path:圆 `M5.5,7 C5.5,3.134 8.634,0 12.5,0 C16.366,0 19.5,3.134 19.5,7 C19.5,10.866 16.366,14 12.5,14 C8.634,14 5.5,10.866 5.5,7 Z` /
  方 `M5.5,0 L19.5,0 L19.5,14 L5.5,14 Z` / 菱 `M12.5,0 L19.5,7 L12.5,14 L5.5,7 Z` / 三角 `M12.5,0 L19.5,14 L5.5,14 Z`。
- **多张折线共用 1 个宽表 SQL**(每类一列 × N 指标)—— Core 不支持 block 互引,
  宽表是唯一能同时喂多张 combo_chart 的形态。
- **原生图例画在 canvas 内** → DOM 里取不到图例文字(与 Evidence built-in DOM 图例不同);
  图例顺序只能靠像素验,或读 canvas 顶部区域。
- `y_fmt="pct1"` 用于 0~1 小数比率列(自动 ×100 + 1 位小数)。

## Connects To

- **ch11** — combo_chart 4 大必避 / 中文 legend / line 节点标记 / Pentagram 配色
- **ch05** — 中文 PG 列名 / 跨 block 引用红线 / Phase 7.x 累积 SQL 不能简化
- **ch09** — 假成功分析与真验证 SOP(§5 是它的页面级细化)
- **ch15** — theme.yaml 5 层 / `{% html %}` 边界
