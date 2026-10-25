---
type: external-doc
title: Dimension Grid
source: https://docs.evidence.dev/components/dimension_grid
source_hash: 711b631513
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Dimension Grid

> 原文: <https://docs.evidence.dev/components/dimension_grid>

![Basic Usage](https://mintcdn.com/evidence/2tF3d2fUavrUXFJO/images/components/dimension_grid/dimension_grid.png?fit=max&auto=format&n=2tF3d2fUavrUXFJO&q=85&s=f546e4c1c79405972d35e2ae6422d285)
    
    
    {% dimension_grid
        id="product_filter"
        data="demo.daily_orders"
    /%}
    

## 

​

Examples

### 

​

Basic Usage

![Basic Usage](https://mintcdn.com/evidence/2tF3d2fUavrUXFJO/images/components/dimension_grid/dimension_grid.png?fit=max&auto=format&n=2tF3d2fUavrUXFJO&q=85&s=f546e4c1c79405972d35e2ae6422d285)
    
    
    {% dimension_grid
        id="product_filter"
        data="demo.daily_orders"
    /%}
    

### 

​

With Explicit Dimensions and Metric
    
    
    {% dimension_grid
        id="sales_filter"
        data="orders"
        dimensions=["category", "region"]
        metric="sum(sales)"
        fmt="usd"
        limit=5
    /%}
    

### 

​

Using Filter in Query

![Using Filter in Query](https://mintcdn.com/evidence/Z2CIIsynbphd2HX3/images/components/dimension_grid/example-2.png?fit=max&auto=format&n=Z2CIIsynbphd2HX3&q=85&s=de4f6384997b03f763903473c7a7f192)
    
    
    {% dimension_grid id="order_filter" data="orders" /%}
    
    ```sql filtered_orders
    select * from orders
    where {{order_filter.filter}}
    ```
    

## 

​

Attributes

​

string

required

The id of the dimension grid to use in a `filters` prop

​

string

required

Name of the table to query

​

array

Array of column names to display as dimensions. If not provided, auto-detects string columns.

​

string

default:"count(*)"

SQL aggregation expression (default: count(*))

​

string

Label displayed above the metric values

​

string

Format code for the metric values (e.g., “num0”, “usd”, “pct1”). See [Value Formatting](</core-concepts/value-formatting>) for available formats.

​

number

default:"10"

Maximum number of values to show per dimension (default: 10)

​

boolean

default:"true"

Allow multiple selections per dimension (default: true)

​

array

Array of filter IDs to apply when querying dimension values

​

string

SQL WHERE clause to filter data

​

string

Title displayed above the dimension grid

​

string

Subtitle displayed below the title

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

## 

​

Using the Filter Variable

Reference this filter using `{{filter_id}}`. The value returned depends on where you use it.

Context| Default Property| No Selection| Result  
---|---|---|---  
  
### 

​

Available Properties

You can also access specific properties using `{{filter_id.property}}`:

#### 

​

.filter

Returns a complete SQL filter expression combining all dimension selections. Returns `true` when no values are selected.
    
    
    {% dimension_grid id="dim_filter" data="orders" /%}
    
    ```sql filtered_orders
    select * from orders
    where {{dim_filter.filter}}
    ```
    

#### 

​

.selected

Returns an object with dimension names as keys and selected values as arrays. Useful for accessing individual dimension selections.
    
    
    {% dimension_grid id="dim_filter" data="orders" dimensions=["category", "region"] /%}
    
    Selected categories: {{dim_filter.category}}
    Selected regions: {{dim_filter.region}}
    

#### 

​

.literal

Returns a human-readable summary of all selections.
    
    
    {% dimension_grid id="dim_filter" data="orders" /%}
    
    Active filters: {{dim_filter.literal}}
    

Was this page helpful?

⌘I
