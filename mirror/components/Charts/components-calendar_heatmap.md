---
type: external-doc
title: Calendar Heatmap
source: https://docs.evidence.dev/components/calendar_heatmap
source_hash: b589d49e1b
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Calendar Heatmap

> 原文: <https://docs.evidence.dev/components/calendar_heatmap>

![Basic Usage](https://mintcdn.com/evidence/oSDKudj1ZJ3RGuev/images/components/calendar_heatmap/calendar_heatmap.png?fit=max&auto=format&n=oSDKudj1ZJ3RGuev&q=85&s=cef902b63817951ff551dc2a0e18a005)
    
    
    {% calendar_heatmap
        data="demo.daily_orders"
        date="date"
        value="sum(total_sales)"
    /%}
    

## 

​

Examples

### 

​

Basic Usage

![Basic Usage](https://mintcdn.com/evidence/oSDKudj1ZJ3RGuev/images/components/calendar_heatmap/calendar_heatmap.png?fit=max&auto=format&n=oSDKudj1ZJ3RGuev&q=85&s=cef902b63817951ff551dc2a0e18a005)
    
    
    {% calendar_heatmap
        data="demo.daily_orders"
        date="date"
        value="sum(total_sales)"
    /%}
    

### 

​

Calendar Heatmap with Custom Colors

![Calendar Heatmap with Custom Colors](https://mintcdn.com/evidence/oSDKudj1ZJ3RGuev/images/components/calendar_heatmap/example-1.png?fit=max&auto=format&n=oSDKudj1ZJ3RGuev&q=85&s=74449f964b0851b668b617451e3b2150)
    
    
    {% calendar_heatmap
        data="demo.daily_orders"
        date="date"
        value="sum(total_sales)"
        title="Daily Sales Heatmap"
        chart_options={
            color_scale = ["#0d0887", "#6300a7", "#a62098", "#d5546e", "#f68d45", "#fcd225", "#f0f921"]
        }
    /%}
    

### 

​

Calendar Heatmap with Conditional Colors

![Calendar Heatmap with Conditional Colors](https://mintcdn.com/evidence/oSDKudj1ZJ3RGuev/images/components/calendar_heatmap/example-2.png?fit=max&auto=format&n=oSDKudj1ZJ3RGuev&q=85&s=aea9ea9318834566204899e6c4951874)
    
    
    {% calendar_heatmap
        data="demo.daily_orders"
        date="date"
        value="sum(total_sales)"
        title="Sales Performance Heatmap"
        chart_options={
            conditional_colors = "case when sum(total_sales) > 65000 then '#22c55e' when sum(total_sales) > 40000 then '#f59e0b' else '#ef4444' end"
        }
    /%}
    

### 

​

Calendar Heatmap with Conditional Colors and Legend

![Calendar Heatmap with Conditional Colors and Legend](https://mintcdn.com/evidence/oSDKudj1ZJ3RGuev/images/components/calendar_heatmap/example-3.png?fit=max&auto=format&n=oSDKudj1ZJ3RGuev&q=85&s=ede6d64272e7c774c4f29cc9e0859e30)
    
    
    {% calendar_heatmap
        data="demo.daily_orders"
        date="date"
        value="sum(total_sales)"
        title="Sales Performance Heatmap"
        chart_options={
            conditional_colors="case when sum(total_sales) > 65000 then '#22c55e' when sum(total_sales) > 40000 then '#f59e0b' else '#ef4444' end"
            color_map={
                "#22c55e"="High Sales"
                "#f59e0b"="Medium Sales"
                "#ef4444"="Low Sales"
            }
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

required

Column name for dates

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

Information tooltip text (can only be used with title)

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

boolean

default:"true"

Show color scale legend

​

boolean

default:"true"

Show borders between calendar cells

​

options group

Color and styling options for the heatmap**Example:**
    
    
     chart_options={
      color_scale = ["value1", "value2"]
      color_palette = ["value1", "value2"]
      conditional_colors = "string"
      color_map = {
        "key1" = "value1"
        "key2" = "value2"
      }
    }
    

**Attributes:**

  * color_scale: `array of strings` \- Array of hex colors for the heatmap gradient. A single-color array auto-expands to [background, color].
  * color_palette: `array of strings` \- Deprecated. Use `color_scale`.
  * conditional_colors: `string` \- SQL CASE expression returning hex colors based on value
  * color_map: `map of key-value pairs` \- Maps hex colors to legend labels when using conditional_colors

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
