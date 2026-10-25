---
type: external-doc
title: Period Selector
source: https://docs.evidence.dev/components/period_selector
source_hash: ef10a741b6
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Period Selector

> 原文: <https://docs.evidence.dev/components/period_selector>

{% period_selector id="sales_period" grain="quarter" /%}
    
    {% big_value
        data="demo.daily_orders"
        value="sum(total_sales)"
        where="date {{sales_period.between}}"
    /%}
    

## 

​

Examples

### 

​

Basic Usage
    
    
    {% period_selector id="sales_period" grain="quarter" /%}
    
    {% big_value
        data="demo.daily_orders"
        value="sum(total_sales)"
        where="date {{sales_period.between}}"
    /%}
    

### 

​

Using `filters`
    
    
    {% period_selector id="sales_period" grain="quarter" value_column="date" /%}
    
    {% line_chart
        data="demo.daily_orders"
        x="date"
        y="sum(total_sales)"
        filters=["sales_period"]
    /%}
    

### 

​

Placing the `workflow.period` selector
    
    
    ---
    workflow:
      period:
        grain: quarter
    ---
    
    {% row align="center" justify="between" %}
    # {{ period.label }} Review
    {% period_selector id="period" /%}
    {% /row %}
    

## 

​

Attributes

​

string

required

The id of the period, used in SQL query templates (e.g. `{{my_period.between}}`). On a page with `workflow.period`, `id="period"` places that selector here instead of at the top of the page.

​

string

The calendar grain each period covers. Defaults to `month`.**Allowed values:**

  * `day`
  * `week`
  * `month`
  * `quarter`
  * `year`

​

number

How many complete periods to offer. Defaults to 12.

​

string

The date column to filter. When set, the selector can be referenced with `{{id.filter}}` or listed in a chart’s `filters` prop, e.g. `filters=["sales_period"]`.

## 

​

Using the Filter Variable

Reference this filter using `{{filter_id}}`. The value returned depends on where you use it.

Context| Default Property| No Selection| Result  
---|---|---|---  
Inline SQL query| `.between`| | `BETWEEN toDate('2026-07-01') AND toDate('2026-07-31')`  
`where` attribute| `.between`| | `BETWEEN toDate('2026-07-01') AND toDate('2026-07-31')`  
Text / Markdown| `.label`| | `Jul 2026`  
  
### 

​

Available Properties

You can also access specific properties using `{{filter_id.property}}`:

#### 

​

.between

A WHERE-clause fragment covering the period in the active SQL dialect: `BETWEEN <start> AND <end>`. **Example value:** `BETWEEN toDate('2026-07-01') AND toDate('2026-07-31')`

#### 

​

.label

The period’s display label, e.g. `Jul 2026`. **Example value:** `Jul 2026`

#### 

​

.start

The first date of the period, as an expression for the active SQL dialect. Use `start_label` in prose. **Example value:** `toDate('2026-07-01')`

#### 

​

.end

The last date of the period (inclusive), for the active SQL dialect. Use `end_label` in prose. **Example value:** `toDate('2026-07-31')`

#### 

​

.start_label

The first date of the period, formatted for display. **Example value:** `Jul 1, 2026`

#### 

​

.end_label

The last date of the period (inclusive), formatted for display. **Example value:** `Jul 31, 2026`

#### 

​

.filter

A complete SQL filter expression for the period. Requires `value_column`; resolves to `true` without it. **Example value:** `order_date >= toDate('2026-07-01') AND order_date <= toDate('2026-07-31')`

#### 

​

.key

The period’s stable identifier, e.g. `2026-07`. Use it to key per-period records. **Example value:** `2026-07`

#### 

​

.grain

The period grain, e.g. `month`. **Example value:** `month`

Was this page helpful?

⌘I
