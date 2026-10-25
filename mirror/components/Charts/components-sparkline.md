---
type: external-doc
title: Sparkline
source: https://docs.evidence.dev/components/sparkline
source_hash: d70189f219
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Sparkline

> 原文: <https://docs.evidence.dev/components/sparkline>

{% sparkline
    	data="demo.daily_orders"
    	y="total_sales"
    	x="date"
    	date_grain="month"
    	date_range={
    		date="date"
    		range="last 12 months"
    	}
    /%}
    

## 

​

Examples

### 

​

Basic Usage
    
    
    {% sparkline
    	data="demo.daily_orders"
    	y="total_sales"
    	x="date"
    	date_grain="month"
    	date_range={
    		date="date"
    		range="last 12 months"
    	}
    /%}
    

### 

​

Semantic metric
    
    
    {% sparkline metric="revenue" /%}
    

### 

​

Sparkline with Color
    
    
    {% sparkline
        data="demo.daily_orders"
        y="total_sales"
        x="date"
        color="blue"
    /%}
    

### 

​

Sparkline with Type
    
    
    {% sparkline
        data="demo.daily_orders"
        y="total_sales"
        x="date"
        type="area"
    /%}
    

## 

​

Attributes

​

string

Table or view to query. Omit when using `metric`.

​

string

Semantic metric name to display, from metrics/*.yaml. Use instead of the raw data/value/x/y attributes.

​

string

Column for x-axis (often date/time). Omit to use the metric view time column.

​

string

Column for y-axis (values to plot). Omit when using `metric`.

​

string

default:"line"

Type of sparkline to display**Allowed values:**

  * `line`
  * `area`
  * `bar`

​

string

Color of the sparkline (CSS color)

​

string

Format for y-axis tooltip (e.g., “num”, “usd”, “pct”). See [Value Formatting](</core-concepts/value-formatting>) for available formats.

​

string

Format for x-axis tooltip (e.g., “shortdate”, “longdate”). See [Value Formatting](</core-concepts/value-formatting>) for available formats.

​

boolean

default:"false"

Whether to scale the y-axis to the data range

​

boolean

default:"true"

Whether the sparkline should be interactive

​

array

IDs of filters to apply to the query

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

Time in seconds between automatic data refreshes (minimum 30). Overrides the page-level auto-refresh setting for this component.

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

number

Set the width of this component (in percent) relative to the page width

​

string

Link this chart to others sharing the same id, syncing their tooltips, axis-pointer, and zoom

Was this page helpful?

⌘I
