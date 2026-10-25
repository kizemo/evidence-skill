---
type: external-doc
title: Delta
source: https://docs.evidence.dev/components/delta
source_hash: d2cbefeaa7
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Delta

> 原文: <https://docs.evidence.dev/components/delta>

![Basic Usage](https://mintcdn.com/evidence/Z2CIIsynbphd2HX3/images/components/delta/delta.png?fit=max&auto=format&n=Z2CIIsynbphd2HX3&q=85&s=fe51a0cd84cde3e514892dfd54c869ff)
    
    
    {% delta
    	data="demo.daily_orders"
    	value="sum(total_sales)"
    	comparison={
    		compare_vs="target"
    		target="120000000"
    	}
    /%}
    

## 

​

Examples

### 

​

Basic Usage

![Basic Usage](https://mintcdn.com/evidence/Z2CIIsynbphd2HX3/images/components/delta/delta.png?fit=max&auto=format&n=Z2CIIsynbphd2HX3&q=85&s=fe51a0cd84cde3e514892dfd54c869ff)
    
    
    {% delta
    	data="demo.daily_orders"
    	value="sum(total_sales)"
    	comparison={
    		compare_vs="target"
    		target="120000000"
    	}
    /%}
    

### 

​

Semantic metric
    
    
    {% delta metric="revenue" comparison={ compare_vs="prior year" } /%}
    

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

SQL expression to insert into the SELECT part of the query (e.g., “COUNT(*)”, “SUM(sales)”). Omit when using `metric`.

​

string

Format code for the value (e.g., “num”, “usd”, “pct”). See formatValue documentation for available formats. See [Value Formatting](</core-concepts/value-formatting>) for available formats.

​

string

Text appearing after the delta (e.g., vs. prev month)

​

boolean

default:"false"

Whether to display as a chip

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
      down_is_good = true
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
  * down_is_good: `boolean` \- Whether a downward trend is considered positive. Defaults to the page’s `down_is_good` setting.

​

boolean

default:"true"

Whether to show the value

​

boolean

default:"true"

Whether to show the delta symbol

​

string

default:"right"

Position of the delta symbol relative to the value**Allowed values:**

  * `left`
  * `right`

​

array

default:"[0,0]"

Range [min, max] for neutral values. Use null for infinity (e.g., [null, 0] means anything ≤ 0 is neutral)

​

array

IDs of filters to apply to the query

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

number

Set the width of this component (in percent) relative to the page width

Was this page helpful?

⌘I
