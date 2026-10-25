---
type: external-doc
title: Table Filter
source: https://docs.evidence.dev/components/table_filter
source_hash: fc8c632684
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Table Filter

> 原文: <https://docs.evidence.dev/components/table_filter>

![Using filters](https://mintcdn.com/evidence/3VqmJP-3dtFa-PrP/images/components/table_filter/table_filter.png?fit=max&auto=format&n=3VqmJP-3dtFa-PrP&q=85&s=bcf23945899bee094b96181b1d1e0992)
    
    
    {% table_filter
        id="my_filter"
        data="demo.daily_orders"
    /%}
    
    {% bar_chart
        data="demo.daily_orders"
        x="date"
        y="sum(total_sales)"
        filters=["my_filter"]
        date_grain="month"
    /%}
    

## 

​

Examples

### 

​

Using `filters`

![Using filters](https://mintcdn.com/evidence/3VqmJP-3dtFa-PrP/images/components/table_filter/table_filter.png?fit=max&auto=format&n=3VqmJP-3dtFa-PrP&q=85&s=bcf23945899bee094b96181b1d1e0992)
    
    
    {% table_filter
        id="my_filter"
        data="demo.daily_orders"
    /%}
    
    {% bar_chart
        data="demo.daily_orders"
        x="date"
        y="sum(total_sales)"
        filters=["my_filter"]
        date_grain="month"
    /%}
    

### 

​

Using Inline SQL
    
    
    {% table_filter
        id="my_filter"
        data="demo.daily_orders"
    /%}
    
    ```sql filtered_orders
    select * from demo.daily_orders
    where {{my_filter.filter}}
    ```
    
    {% table data="filtered_orders" /%}
    

### 

​

Custom Title
    
    
    {% table_filter
        id="orders_filter"
        data="demo.daily_orders"
        title="Order Search"
    /%}
    

### 

​

Mixed Single and Multi Select
    
    
    {% table_filter
        id="mixed_filter"
        data="demo.daily_orders"
        columns=["category", "item"]
        single_select=["category"]
    /%}
    

## 

​

Attributes

​

string

required

Unique identifier for the filter component

​

string

Additional CSS classes to apply

​

string

required

ID of the table to filter

​

string

default:"Filter"

Custom title text for the filter button (defaults to “Filter”)

​

string

default:"AND"

Default conjunction between filters (AND or OR)**Allowed values:**

  * `AND`
  * `OR`

​

array

Array of column IDs to filter on. If not provided, all columns are available for filtering

​

array

Array of custom labels to display instead of column names. Must match the order of the columns array.

​

boolean

default:"true"

Whether to show a clear button to remove all filters

​

number

When set, string filters will only show values that have at least this many records, and filters will always use AND conjunction

​

boolean

default:"true"

When false, string filters will only allow selecting a single value instead of multiple values. Can be overridden per column with single_select and multi_select.

​

array

Array of column names whose string filter only allows selecting a single value, regardless of the multiple setting

​

array

Array of column names whose string filter allows selecting multiple values, regardless of the multiple setting

​

object

An object with column names as keys and this filter’s initial value for that column as the value

​

array

An array containing column names that always require a selection. Only supports text columns.

​

number

Set the width of this component (in percent) relative to the page width

## 

​

Using the Filter Variable

Reference this filter using `{{filter_id}}`. The value returned depends on where you use it.

Context| Default Property| No Selection| Result  
---|---|---|---  
Inline SQL query| `.filter`| `true`| `category = 'Electronics' AND total_sales > 1000`  
`where` attribute| `.filter`| `true`| `category = 'Electronics' AND total_sales > 1000`  
Text / Markdown| `.filter`| `true`| `category = 'Electronics' AND total_sales > 1000`  
  
### 

​

Available Properties

You can also access specific properties using `{{filter_id.property}}`:

#### 

​

.filter

Returns a complete SQL filter expression combining all active filter conditions. Returns `true` when no filters are active.
    
    
    {% table_filter id="my_filter" data="demo.daily_orders" /%}
    
    ```sql filtered_orders
    select * from demo.daily_orders
    where {{my_filter}}
    ```
    

**Example value:** `category = 'Electronics' AND total_sales > 1000`

Was this page helpful?

⌘I
