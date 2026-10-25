---
type: external-doc
title: Big Value
source: https://docs.evidence.dev/components/big_value
source_hash: b3568cf008
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Big Value

> 原文: <https://docs.evidence.dev/components/big_value>

![Basic Usage](https://mintcdn.com/evidence/Z2CIIsynbphd2HX3/images/components/big_value/big_value.png?fit=max&auto=format&n=Z2CIIsynbphd2HX3&q=85&s=07a67a4d9c58570d4d3db0638fd47999)
    
    
    {% big_value
      data="demo.daily_orders"
      value="sum(total_sales)"
      fmt="usd1m"
    /%}
    

## 

​

Examples

### 

​

Basic Usage

![Basic Usage](https://mintcdn.com/evidence/Z2CIIsynbphd2HX3/images/components/big_value/big_value.png?fit=max&auto=format&n=Z2CIIsynbphd2HX3&q=85&s=07a67a4d9c58570d4d3db0638fd47999)
    
    
    {% big_value
      data="demo.daily_orders"
      value="sum(total_sales)"
      fmt="usd1m"
    /%}
    

### 

​

Comparison

![Comparison](https://mintcdn.com/evidence/qxp1vq5RbQ6sDlcg/images/components/big_value/example-1.png?fit=max&auto=format&n=qxp1vq5RbQ6sDlcg&q=85&s=61552571ba248ab0fb44c3f9dec439b3)
    
    
    {% big_value
    	data="demo.daily_orders"
    	value="sum(total_sales)"
    	fmt="usd1m"
    	date_range={
    		date="date"
    		range="last 12 months"
    	}
    	comparison={
    		compare_vs="prior year"
    	}
    /%}
    

### 

​

Sparkline

![Sparkline](https://mintcdn.com/evidence/Z2CIIsynbphd2HX3/images/components/big_value/example-2.png?fit=max&auto=format&n=Z2CIIsynbphd2HX3&q=85&s=7d24595d05d954cba9f3381a11655d12)
    
    
    {% big_value
    	data="demo.daily_orders"
    	value="sum(total_sales)"
    	fmt="usd1m"
    	sparkline={
    		type="line"
    		x="date"
    	}
    /%}
    

### 

​

Text Size

![Text Size](https://mintcdn.com/evidence/Z2CIIsynbphd2HX3/images/components/big_value/example-3.png?fit=max&auto=format&n=Z2CIIsynbphd2HX3&q=85&s=4d920c0ab114633cc478a65e7c6f43c8)
    
    
    {% big_value
    	data="demo.daily_orders"
    	value="sum(total_sales)"
    	fmt="usd1m"
    	text_size="4xl"
    /%}
    

## 

​

Attributes

​

string

Table or view to query. Omit when using `metric`.

​

string

Semantic metric to display (the whole reference — a metric is a single number). Use instead of `data` \+ `value`.

​

string

The SQL aggregation to display (with `data`). Omit when using `metric`.

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

options group

Comparison configuration object**Example:**
    
    
     comparison={
      compare_vs = "prior year"
      display_type = "compared_value"
      target = "string"
      benchmark = {
        agg = "avg"
        subject = "store_name"
        within = ["region"]
      }
      hide_pct = true
      pct_fmt = "string"
      abs_fmt = "string"
      text = "string"
      delta = true
      down_is_good = true
      neutral_range = []
    }
    

**Attributes:**

  * compare_vs: `string` \- Type of comparison to perform. Options: prior year (same period last year), prior period (previous period of same duration), target (compare against a target value), benchmark (compare against group average/aggregate)
    * **Allowed values:**
      * `prior year`
      * `prior period`
      * `target`
      * `benchmark`
  * display_type: `string` \- What to display for comparison. Options: compared_value (comparison period value), abs (absolute change), pct (percentage change). Default: pct
    * **Allowed values:**
      * `compared_value`
      * `abs`
      * `pct`
  * target: `string` \- Target value for target comparison. Can be a column name, aggregation (e.g., “sum(target_sales)”), or literal value.
  * benchmark: `options group`
    * **Options:**
      * agg: `string` \- Aggregation function to apply across benchmark group. Options: avg (average), median, min, max, sum, count, count_distinct
        * **Allowed values:**
          * `avg`
          * `median`
          * `min`
          * `max`
          * `sum`
          * `count`
          * `count_distinct`
      * subject: `string` \- Column or expression that defines individual entities in the benchmark (e.g., “store_name”, “customer_id”). Required for single-value components.
      * value: `string` \- Optional column or expression to use for benchmark calculation. If not specified, uses the main value column. Useful if you have a pre-aggregated benchmark table for RLS reasons.
      * within: `array of strings` \- Dimension columns to group the benchmark by (e.g., [“region”]). Leave empty for dataset-wide benchmark.
      * where: `string` \- SQL WHERE clause to filter which entities are included in the benchmark
      * exclude_self: `boolean` \- Exclude the current row from its own benchmark calculation (table context only). Default: false
  * hide_pct: `boolean` \- Hide the percentage change line in comparison tooltips
  * pct_fmt: `string` \- Format code for percentage values in comparison tooltips
  * abs_fmt: `string` \- Format code for absolute values in comparison tooltips
  * text: `string` \- Text displayed after the comparison value
  * delta: `boolean` \- Whether to display the comparison as a delta
  * down_is_good: `boolean` \- Whether a decrease is considered positive. Defaults to the page’s `down_is_good` setting.
  * neutral_range: `array` \- Range [min, max] for neutral values. Use null for infinity (e.g., [null, 0] means anything ≤ 0 is neutral)

​

options group

Sparkline configuration object**Example:**
    
    
     sparkline={
      type = "line"
      color = "string"
      x = "string"
      y_fmt = "string"
      x_fmt = "string"
      fit_to_data = true
      connect_group = "string"
      date_grain = "year"
      date_range = {
        range = "today"
        date = "string"
      }
    }
    

**Attributes:**

  * type: `string` \- The type of sparkline to display
    * **Allowed values:**
      * `line`
      * `area`
      * `bar`
  * color: `string` \- Color for the sparkline
  * x: `string` \- X column for the sparkline
  * y_fmt: `string` \- Value format for the sparkline tooltips
  * x_fmt: `string` \- Date format for the sparkline tooltips
  * fit_to_data: `boolean` \- Whether to fit the Y axis scale to the data range
  * connect_group: `string` \- Connect group for the sparkline
  * date_grain: `string` \- Time grain for the sparkline data points
    * **Allowed values:**
      * `year`
      * `quarter`
      * `month`
      * `week`
      * `day`
      * `hour`
      * `minute`
  * date_range: `options group`
    * **Options:**
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

Format for the main value. See [Value Formatting](</core-concepts/value-formatting>) for available formats.

​

string

Title for the main value

​

string

default:"fit-content"

Maximum width of the component

​

string

default:"auto"

Minimum width of the component

​

string

Text size for the value. Options: sm, base, lg, xl (default), 2xl, 3xl, 4xl, 5xl**Allowed values:**

  * `sm`
  * `base`
  * `lg`
  * `xl`
  * `2xl`
  * `3xl`
  * `4xl`
  * `5xl`

​

string

Additional CSS classes for the title

​

string

Additional CSS classes for the value

​

string

URL to link the title to

​

string

Information tooltip text

​

string

URL to link the info text to (can only be used with info)

​

string

Create a custom link title for the info link, placed after the info text (can only be used with info_link)

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

string

Additional CSS classes for the component

​

number

Time in seconds between automatic data refreshes (minimum 30). Overrides the page-level auto-refresh setting for this component.

​

number

Set the width of this component (in percent) relative to the page width

Was this page helpful?

⌘I
