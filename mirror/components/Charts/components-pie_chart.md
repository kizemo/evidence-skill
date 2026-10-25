---
type: external-doc
title: Pie Chart
source: https://docs.evidence.dev/components/pie_chart
source_hash: 310bdeea0d
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Pie Chart

> 原文: <https://docs.evidence.dev/components/pie_chart>

![Basic Usage](https://mintcdn.com/evidence/oSDKudj1ZJ3RGuev/images/components/pie_chart/pie_chart.png?fit=max&auto=format&n=oSDKudj1ZJ3RGuev&q=85&s=64dda124a60fd354c2c338a1c09aac55)
    
    
    {% pie_chart
        data="demo.daily_orders"
        category="category"
        value="sum(total_sales)"
    /%}
    

## 

​

Examples

### 

​

Basic Usage

![Basic Usage](https://mintcdn.com/evidence/oSDKudj1ZJ3RGuev/images/components/pie_chart/pie_chart.png?fit=max&auto=format&n=oSDKudj1ZJ3RGuev&q=85&s=64dda124a60fd354c2c338a1c09aac55)
    
    
    {% pie_chart
        data="demo.daily_orders"
        category="category"
        value="sum(total_sales)"
    /%}
    

### 

​

Pie Chart with Title

![Pie Chart with Title](https://mintcdn.com/evidence/oSDKudj1ZJ3RGuev/images/components/pie_chart/example-1.png?fit=max&auto=format&n=oSDKudj1ZJ3RGuev&q=85&s=9f428060936ff0b6916fa6c96ccad6dd)
    
    
    {% pie_chart
        data="demo.daily_orders"
        value="sum(total_sales)"
        category="category"
        title="Sales by Category"
    /%}
    

### 

​

With Custom Colors

![With Custom Colors](https://mintcdn.com/evidence/oSDKudj1ZJ3RGuev/images/components/pie_chart/example-2.png?fit=max&auto=format&n=oSDKudj1ZJ3RGuev&q=85&s=ea3acf300f80800db06afc4ec03ca222)
    
    
    {% pie_chart
        data="demo.daily_orders"
        category="category"
        value="sum(total_sales)"
        title="Sales by Category"
        chart_options={
            color_palette = ["#0d0887", "#6300a7", "#a62098", "#d5546e", "#f68d45", "#fcd225", "#f0f921"]
        }
    /%}
    

### 

​

With Series Colors

![With Series Colors](https://mintcdn.com/evidence/oSDKudj1ZJ3RGuev/images/components/pie_chart/example-3.png?fit=max&auto=format&n=oSDKudj1ZJ3RGuev&q=85&s=62789fad6d0c15e1bdabb844c5250185)
    
    
    {% pie_chart
        data="demo.daily_orders"
        category="category"
        value="sum(total_sales)"
        title="Sales by Category"
        chart_options={
            series_colors={
                "Electronics"="#3b82f6"
                "Clothing"="#22c55e"
                "Home"="#f59e0b"
                "Sports"="#ef4444"
            }
        }
    /%}
    

### 

​

With Percentage Labels

![With Percentage Labels](https://mintcdn.com/evidence/oSDKudj1ZJ3RGuev/images/components/pie_chart/example-4.png?fit=max&auto=format&n=oSDKudj1ZJ3RGuev&q=85&s=1b7ab6a4bc5b82610c699588e23c34d7)
    
    
    {% pie_chart
        data="demo.daily_orders"
        category="category"
        value="sum(total_sales)"
        title="Sales by Category"
        pct=true
    /%}
    

### 

​

With Custom Percentage Format

![With Custom Percentage Format](https://mintcdn.com/evidence/oSDKudj1ZJ3RGuev/images/components/pie_chart/example-5.png?fit=max&auto=format&n=oSDKudj1ZJ3RGuev&q=85&s=0edf582640733588991e59adefa19509)
    
    
    {% pie_chart
        data="demo.daily_orders"
        category="category"
        value="sum(total_sales)"
        title="Sales by Category"
        pct=true
        pct_fmt="pct0"
    /%}
    

## 

​

Attributes

​

string

Name of the table or view to query. Omit when using `metric`.

​

string

Semantic metric name to display, from metrics/*.yaml. Use instead of the raw data/value/x/y attributes.

​

array

IDs of filters to apply to the query

​

options group

Filter data to a time period. `date_range` is an OBJECT with `range` (the period) and optionally `date` (which column to filter on when the table has more than one). Shape: `date_range={ range="last 12 months" date="order_date" }`. `range` accepts predefined values (`last 7 days`, `month to date`), dynamic patterns (`Last 90 days`), custom windows (`2020-01-01 to 2023-03-01`), or partial ranges (`from 2020-01-01`, `until 2023-03-01`). Pass a plain string for `range` — the whole object is NOT a string.**Example:**
    
    
     date_range={
      range = "today"
      date = "string"
    }
    

**Attributes:**

  * range: `string` \- Time period to filter. Use presets like ‘last 7 days’, dynamic patterns like ‘Last 90 days’, custom ranges like ‘2020-01-01 to 2023-03-01’, or partial ranges like ‘from 2020-01-01’.
    * **Allowed values:**
      * `today`
      * `yesterday`
      * `last 7 days`
      * `last 30 days`
      * `last 3 months`
      * `last 6 months`
      * `last 12 months`
      * `previous week`
      * `previous month`
      * `previous quarter`
      * `previous year`
      * `this week`
      * `this month`
      * `this quarter`
      * `this year`
      * `next week`
      * `next month`
      * `next quarter`
      * `next year`
      * `week to date`
      * `month to date`
      * `quarter to date`
      * `year to date`
      * `all time`
  * date: `string` \- Date column to filter on. Required when the data has multiple date columns.

​

string

required

Column name for categories (pie slices)

​

string

Column name for values (slice sizes). Omit when using `metric`.

​

string

Title to display above the component

​

string

Subtitle to display below the title

​

string

Information tooltip text (can only be used with title). Displays an info icon next to the title.

​

string

URL to link the info text to (can only be used with info)

​

string

Create a custom link title for the info link, placed after the info text (can only be used with info_link)

​

string

default:"40%"

Inner radius of the pie (use 0% for full pie, >0% for donut style)

​

string

default:"num"

Format for values. See [Value Formatting](</core-concepts/value-formatting>) for available formats.

​

boolean

default:"false"

Show legend instead of slice labels

​

boolean

default:"false"

Show percentage values on pie slice labels

​

string

default:"pct1"

Format for percentage values (e.g., pct0, pct1, pct2). See [Value Formatting](</core-concepts/value-formatting>) for available formats.

​

string

default:"top"

Position of the legend (top or bottom)**Allowed values:**

  * `top`
  * `bottom`

​

options group

Chart configuration options**Example:**
    
    
     chart_options={
      color_palette = ["value1", "value2"]
      series_colors = {
        "key1" = "value1"
        "key2" = "value2"
      }
    }
    

**Attributes:**

  * color_palette: `array of strings`
  * series_colors: `map of key-value pairs`

​

number

Time in seconds between automatic data refreshes (minimum 30). Overrides the page-level auto-refresh setting for this component.

​

string

Custom SQL WHERE condition to apply to the query. For date filters, use date_range instead.

​

string

Custom SQL HAVING condition to apply to the query after GROUP BY

​

number

Maximum number of rows to return from the query. Note: When used with tables, limit will disable subtotals to prevent incomplete subtotal rows.

​

string

Column name(s) with optional direction (e.g. “column_name”, “column_name desc”)

​

string

Custom SQL QUALIFY condition to filter windowed results

​

number

Set the width of this component (in percent) relative to the page width

​

number

Set a fixed height for the chart in pixels

​

string

Link this chart to others sharing the same id, syncing their tooltips, axis-pointer, and zoom

​

array

Extra columns to include in the tooltip on hover. Each entry is `{ value, label?, fmt?, color_by_sign?, down_is_good? }`. See the [tooltip fields guide](</components/tooltip-fields>) for examples.

​

map

Raw [ECharts options](<https://echarts.apache.org/en/option.html>) deep-merged over the chart’s final configuration. Use for anything the structured props do not expose — `graphic`, `visualMap`, tooltip styling, and so on. Partial overrides win key-by-key without clobbering Studio’s computed siblings. For overrides scoped to the data series, use `echarts_series_options`.**Example:**
    
    
     echarts_options={
        tooltip={ position="top" }
        graphic=[{ type="text" }]
    }
    

​

map

Raw [ECharts series options](<https://echarts.apache.org/en/option.html#series>) deep-merged into the chart series. Use for series-level styling the structured props do not expose.**Example:**
    
    
     echarts_series_options={
        itemStyle={ borderRadius=8 }
    }
    

Was this page helpful?

⌘I
