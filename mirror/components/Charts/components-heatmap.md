---
type: external-doc
title: Heatmap
source: https://docs.evidence.dev/components/heatmap
source_hash: 23d0eabbaa
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Heatmap

> 原文: <https://docs.evidence.dev/components/heatmap>

![Basic Usage](https://mintcdn.com/evidence/oSDKudj1ZJ3RGuev/images/components/heatmap/heatmap.png?fit=max&auto=format&n=oSDKudj1ZJ3RGuev&q=85&s=fbce07804847bec81292a7480fbe2419)
    
    
    {% heatmap
        data="demo.daily_orders"
        x="date"
        x_date_grain="year"
        y="category"
        value="sum(total_sales)"
    /%}
    

## 

​

Examples

### 

​

Basic Usage

![Basic Usage](https://mintcdn.com/evidence/oSDKudj1ZJ3RGuev/images/components/heatmap/heatmap.png?fit=max&auto=format&n=oSDKudj1ZJ3RGuev&q=85&s=fbce07804847bec81292a7480fbe2419)
    
    
    {% heatmap
        data="demo.daily_orders"
        x="date"
        x_date_grain="year"
        y="category"
        value="sum(total_sales)"
    /%}
    

### 

​

With Custom Colors

![With Custom Colors](https://mintcdn.com/evidence/oSDKudj1ZJ3RGuev/images/components/heatmap/example-1.png?fit=max&auto=format&n=oSDKudj1ZJ3RGuev&q=85&s=96bcfcbb8274ac60cf9476888f27ca95)
    
    
    {% heatmap
        data="demo.daily_orders"
        x="date"
        x_date_grain="year"
        y="category"
        value="sum(total_sales)"
        title="Sales Intensity by Year and Category"
        chart_options={
            color_scale = ["#f7fafc", "#e2e8f0", "#cbd5e0", "#a0aec0", "#718096", "#4a5568", "#2d3748"]
        }
    /%}
    

## 

​

Attributes

​

string

Name of the table to query. Omit when using `metric`.

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

Bucket dates into a grain. Pass the raw date column as `x` and the chart truncates and groups for you. Temporal grains (`day`, `week`, `month`, `quarter`, `year`, `hour`) preserve the year — use for time-series. Seasonality grains (`day of week`, `day of month`, `day of year`, `week of year`, `month of year`, `quarter of year`) collapse across years — use for cyclical patterns like “which month sells most regardless of year”.**Allowed values:**

  * `day`
  * `week`
  * `month`
  * `quarter`
  * `year`
  * `hour`
  * `day of week`
  * `day of month`
  * `day of year`
  * `week of year`
  * `month of year`
  * `quarter of year`

​

string

Bucket dates into a grain. Pass the raw date column as `x` and the chart truncates and groups for you. Temporal grains (`day`, `week`, `month`, `quarter`, `year`, `hour`) preserve the year — use for time-series. Seasonality grains (`day of week`, `day of month`, `day of year`, `week of year`, `month of year`, `quarter of year`) collapse across years — use for cyclical patterns like “which month sells most regardless of year”.**Allowed values:**

  * `day`
  * `week`
  * `month`
  * `quarter`
  * `year`
  * `hour`
  * `day of week`
  * `day of month`
  * `day of year`
  * `week of year`
  * `month of year`
  * `quarter of year`

​

string

required

Column name for x-axis categories

​

string

required

Column name for y-axis categories

​

string

Title to display for the x-axis

​

string

Title to display for the y-axis

​

string

Column name for cell values. Omit when using `metric`.

​

string

Title to display above the chart

​

string

Subtitle to display below the title

​

string

Info text to display in a tooltip next to the title. Can only be used with the title prop.

​

string

URL to link the info text to (can only be used with info)

​

string

Create a custom link title for the info link, placed after the info text (can only be used with info_link)

​

string

Format for values (defaults to `num`, or the metric format in metric mode). See [Value Formatting](</core-concepts/value-formatting>) for available formats.

​

string

Format for x axis labels. See [Value Formatting](</core-concepts/value-formatting>) for available formats.

​

string

Format for y axis labels. See [Value Formatting](</core-concepts/value-formatting>) for available formats.

​

string

Sort order for x axis categories (asc or desc)**Allowed values:**

  * `asc`
  * `desc`

​

string

Sort order for y axis categories (asc or desc)**Allowed values:**

  * `asc`
  * `desc`

​

string

Sort categories by total value (asc or desc). Applies to the first axis without explicit sort.**Allowed values:**

  * `asc`
  * `desc`

​

boolean

default:"true"

Show color scale legend

​

boolean

default:"true"

Show borders around heatmap cells

​

options group

Chart configuration options**Example:**
    
    
     chart_options={
      color_scale = ["value1", "value2"]
      color_palette = ["value1", "value2"]
    }
    

**Attributes:**

  * color_scale: `array of strings` \- Array of hex colors for the heatmap gradient. A single-color array auto-expands to [background, color].
  * color_palette: `array of strings` \- Deprecated. Use `color_scale`.

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
