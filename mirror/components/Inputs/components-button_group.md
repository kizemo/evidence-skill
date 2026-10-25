---
type: external-doc
title: Button Group
source: https://docs.evidence.dev/components/button_group
source_hash: 213bcc000d
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Button Group

> 原文: <https://docs.evidence.dev/components/button_group>

![Using filters](https://mintcdn.com/evidence/Z2CIIsynbphd2HX3/images/components/button_group/button_group.png?fit=max&auto=format&n=Z2CIIsynbphd2HX3&q=85&s=0985e8a35f7764ebf24dddb375f1c9b1)
    
    
    {% button_group
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

![Using filters](https://mintcdn.com/evidence/Z2CIIsynbphd2HX3/images/components/button_group/button_group.png?fit=max&auto=format&n=Z2CIIsynbphd2HX3&q=85&s=0985e8a35f7764ebf24dddb375f1c9b1)
    
    
    {% button_group
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

![Using where](https://mintcdn.com/evidence/qxp1vq5RbQ6sDlcg/images/components/button_group/example-1.png?fit=max&auto=format&n=qxp1vq5RbQ6sDlcg&q=85&s=19f6f028d7b44a8811e841845a662fca)
    
    
    {% button_group
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
    
    
    {% button_group
        id="category_filter"
        data="demo.daily_orders"
        value_column="category"
    /%}
    
    ```sql filtered_orders
    select * from demo.daily_orders
    where category = {{category_filter}}
    ```
    
    {% table data="filtered_orders" /%}
    

### 

​

Vertical orientation inside a row

![Vertical orientation inside a row](https://mintcdn.com/evidence/Z2CIIsynbphd2HX3/images/components/button_group/example-3.png?fit=max&auto=format&n=Z2CIIsynbphd2HX3&q=85&s=dad206fabeb2436805e5e8d912e06cd9)
    
    
    {% row %}
    
    {% button_group
        id="category_filter"
        data="demo.daily_orders"
        value_column="category"
        orientation="vertical"
    /%}
    
    {% bar_chart
        data="demo.daily_orders"
        x="date"
        y="sum(total_sales)"
        filters=["category_filter"]
        date_grain="month"
    /%}
    
    {% /row %}
    

## 

​

Attributes

​

string

required

The id of the button group to be used in a `filters` prop

​

string

Name of the table to query

​

array

Array of filter IDs to apply when querying for options

​

string

Column name to use as the value for each option, and the column to filter by when this button group’s `id` is used in the `filters` prop of a chart

​

string

Column name to use as the label for each option

​

string

Text displayed above the button group

​

string

Information tooltip text

​

string | number | array

Initial selected value(s)

​

boolean

default:"false"

Allows multiple selections

​

boolean

default:"false"

Automatically select the first option when the component loads

​

string

default:"horizontal"

Layout direction of the button group. Use “vertical” to stack buttons in a column. Best paired with a sibling element inside a `row` so the stack sits alongside its target content.**Allowed values:**

  * `horizontal`
  * `vertical`

​

number

Maximum height in pixels for the button stack when `orientation="vertical"`. Buttons scroll if the list overflows. Ignored when horizontal.

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

## 

​

Using the Filter Variable

Reference this filter using `{{filter_id}}`. The value returned depends on where you use it. The examples below show values for three scenarios:

  * **No selection**
  * **Single select:** “Electronics” selected
  * **Multi select:** “Sports” and “Home” selected (when `multiple=true`)

Context| Default Property| No Selection| Single Select| Multi Select  
---|---|---|---|---  
Inline SQL query| `.selected`| `''`| `'Electronics'`| `('Sports', 'Home')`  
`where` attribute| `.selected`| `''`| `'Electronics'`| `('Sports', 'Home')`  
Text / Markdown| `.literal`| | `Electronics`| `Sports, Home`  
  
### 

​

Available Properties

You can also access specific properties using `{{filter_id.property}}`:

#### 

​

.filter

Returns a complete SQL filter expression ready to use in WHERE clauses. Returns `true` when no value is selected.
    
    
    {% button_group id="category_filter" data="products" value_column="category" /%}
    
    ```sql filtered_products
    select * from products
    where {{category_filter.filter}}
    ```
    

No Selection| Single Select| Multi Select  
---|---|---  
`true`| `category = 'Electronics'`| `category IN ('Sports', 'Home')`  
  
#### 

​

.selected

Returns the selected value(s) wrapped in quotes, suitable for SQL comparisons. Returns an empty string when no value is selected.
    
    
    {% button_group id="category_filter" data="products" value_column="category" /%}
    
    ```sql products_by_category
    select * from products
    where category = {{category_filter.selected}}
    ```
    

No Selection| Single Select| Multi Select  
---|---|---  
`''`| `'Electronics'`| `('Sports', 'Home')`  
  
#### 

​

.literal

Returns the raw unescaped selected value(s), useful for display in text or dynamic column selection.
    
    
    {% button_group id="sort_column" data="products" value_column="column_name" /%}
    
    ```sql dynamic_sort
    select * from products
    order by {{sort_column.literal}}
    ```
    

No Selection| Single Select| Multi Select  
---|---|---  
“| `Electronics`| `Sports, Home`  
  
#### 

​

.label

Returns the display label for the selected option(s). Falls back to the value if no label is defined.
    
    
    {% button_group id="category_filter" %}
        {% option value="Electronics" label="Electronics" /%}
        {% option value="Sports" label="Sports" /%}
        {% option value="Home" label="Home" /%}
    {% /button_group %}
    
    Selected: {{category_filter.label}}
    

No Selection| Single Select| Multi Select  
---|---|---  
“| `Electronics`| `Sports, Home`  
  
#### 

​

.fmt

Returns the format string associated with the selected option. For multiple selections, returns the first format.
    
    
    {% button_group id="metric_selector" %}
        {% option value="revenue" label="Revenue" fmt="usd" /%}
        {% option value="growth_rate" label="Growth Rate" fmt="pct1" /%}
    {% /button_group %}
    
    {% big_value data={metrics} value=value fmt={{metric_selector.fmt}} /%}
    

No Selection| Single Select| Multi Select  
---|---|---  
“| `usd`| `usd`  
  
## 

​

Allowed Children

  * [option](</components/option>)

Was this page helpful?

⌘I
