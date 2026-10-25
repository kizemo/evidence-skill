---
type: external-doc
title: Input Tabs
source: https://docs.evidence.dev/components/input_tabs
source_hash: 3992aac183
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Input Tabs

> 原文: <https://docs.evidence.dev/components/input_tabs>

![Using filters](https://mintcdn.com/evidence/Z2CIIsynbphd2HX3/images/components/input_tabs/input_tabs.png?fit=max&auto=format&n=Z2CIIsynbphd2HX3&q=85&s=2f3e58171da8c884683e658060197ec4)
    
    
    {% input_tabs
        id="category_filter"
        data="demo.daily_orders"
        value_column="category"
    /%}
    
    {% bar_chart
        data="demo.daily_orders"
        x="date"
        y="sum(total_sales)"
        filters=["category_filter"]
        date_grain="month"
    /%}
    

## 

​

Examples

### 

​

Using `filters`

![Using filters](https://mintcdn.com/evidence/Z2CIIsynbphd2HX3/images/components/input_tabs/input_tabs.png?fit=max&auto=format&n=Z2CIIsynbphd2HX3&q=85&s=2f3e58171da8c884683e658060197ec4)
    
    
    {% input_tabs
        id="category_filter"
        data="demo.daily_orders"
        value_column="category"
    /%}
    
    {% bar_chart
        data="demo.daily_orders"
        x="date"
        y="sum(total_sales)"
        filters=["category_filter"]
        date_grain="month"
    /%}
    

### 

​

Using `where`

![Using where](https://mintcdn.com/evidence/Z2CIIsynbphd2HX3/images/components/input_tabs/example-1.png?fit=max&auto=format&n=Z2CIIsynbphd2HX3&q=85&s=a7d4749aa1e4d9344a6aeb8f1041700c)
    
    
    {% input_tabs
        id="category_filter"
        data="demo.daily_orders"
        value_column="category"
    /%}
    
    {% bar_chart
        data="demo.daily_orders"
        x="date"
        y="sum(total_sales)"
        where="category = {{category_filter}}"
        date_grain="month"
    /%}
    

### 

​

Using Inline SQL

![Using Inline SQL](https://mintcdn.com/evidence/Z2CIIsynbphd2HX3/images/components/input_tabs/example-2.png?fit=max&auto=format&n=Z2CIIsynbphd2HX3&q=85&s=aac85bdd5127002c0e346c581041e2b7)
    
    
    {% input_tabs
        id="category_filter"
        data="demo.daily_orders"
        value_column="category"
    /%}
    
    ```sql filtered_orders
    select * from demo.daily_orders
    where category = {{category_filter}}
    ```
    
    {% table data="filtered_orders" /%}
    

## 

​

Attributes

​

string

required

The id of the input tabs to be used in a `filters` prop

​

string

Name of the table to query

​

array

Array of filter IDs to apply when querying for options

​

string

Column name to use as the value for each option, and the column to filter by when this input tabs’ `id` is used in the `filters` prop of a chart

​

string

Column name to use as the label for each option

​

string

Initial selected value (single selection only)

​

string

default:"default"

Visual style variant: “default” for underline style, “well” for button-style tabs**Allowed values:**

  * `default`
  * `well`

​

boolean

default:"false"

Whether the tabs should take the full width of their container

​

string

default:"left"

Horizontal alignment of tabs. Note: align right only affects the default variant.**Allowed values:**

  * `left`
  * `right`

​

boolean

default:"true"

Automatically select the first option when the component loads (defaults to true)

​

string

Column name(s) with optional direction (e.g. “column_name”, “column_name desc”)

​

string

Custom SQL WHERE condition to apply to the query. For date filters, use date_range instead.

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
Inline SQL query| `.selected`| `''`| `'Electronics'`  
`where` attribute| `.selected`| `''`| `'Electronics'`  
Text / Markdown| `.literal`| | `Electronics`  
  
### 

​

Available Properties

You can also access specific properties using `{{filter_id.property}}`:

#### 

​

.filter

Returns a complete SQL filter expression ready to use in WHERE clauses. Returns `true` when no value is selected.
    
    
    {% input_tabs id="category_filter" data="products" value_column="category" /%}
    
    ```sql filtered_products
    select * from products
    where {{category_filter.filter}}
    ```
    

**Example value:** `category = 'Electronics'`

#### 

​

.selected

Returns the selected value wrapped in quotes, suitable for SQL comparisons. Returns an empty string when no value is selected.
    
    
    {% input_tabs id="category_filter" data="products" value_column="category" /%}
    
    ```sql products_by_category
    select * from products
    where category = {{category_filter.selected}}
    ```
    

**Example value:** `'Electronics'`

#### 

​

.literal

Returns the raw unescaped selected value, useful for display in text or dynamic column selection.
    
    
    {% input_tabs id="sort_column" data="products" value_column="column_name" /%}
    
    ```sql dynamic_sort
    select * from products
    order by {{sort_column.literal}}
    ```
    

**Example value:** `Electronics`

#### 

​

.label

Returns the display label for the selected option. Falls back to the value if no label is defined.
    
    
    {% input_tabs id="category_filter" %}
        {% option value="Electronics" label="Electronics" /%}
        {% option value="Sports" label="Sports" /%}
        {% option value="Home" label="Home" /%}
    {% /input_tabs %}
    
    Selected: {{category_filter.label}}
    

**Example value:** `Electronics`

#### 

​

.fmt

Returns the format string associated with the selected option. Useful for dynamically updating chart formatting.
    
    
    {% input_tabs id="metric_selector" %}
        {% option value="revenue" label="Revenue" fmt="usd" /%}
        {% option value="growth_rate" label="Growth Rate" fmt="pct1" /%}
    {% /input_tabs %}
    
    {% big_value data={metrics} value=value fmt={{metric_selector.fmt}} /%}
    

**Example value:** `usd`

## 

​

Allowed Children

  * [option](</components/option>)

Was this page helpful?

⌘I
