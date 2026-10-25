---
type: external-doc
title: Slider
source: https://docs.evidence.dev/components/slider
source_hash: 6a935431c6
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Slider

> 原文: <https://docs.evidence.dev/components/slider>

![Using filters](https://mintcdn.com/evidence/ImoR9rIJqT9N0Esa/images/components/slider/slider.png?fit=max&auto=format&n=ImoR9rIJqT9N0Esa&q=85&s=524baf92d184a457edcb8178292e8574)
    
    
    {% slider
        id="sales_filter"
        data="demo.daily_orders"
        value_column="total_sales"
        title="Sales Amount"
    /%}
    
    {% bar_chart
        data="demo.daily_orders"
        x="date"
        y="sum(total_sales)"
        filters=["sales_filter"]
        date_grain="month"
    /%}
    

## 

​

Examples

### 

​

Using `filters`

![Using filters](https://mintcdn.com/evidence/ImoR9rIJqT9N0Esa/images/components/slider/slider.png?fit=max&auto=format&n=ImoR9rIJqT9N0Esa&q=85&s=524baf92d184a457edcb8178292e8574)
    
    
    {% slider
        id="sales_filter"
        data="demo.daily_orders"
        value_column="total_sales"
        title="Sales Amount"
    /%}
    
    {% bar_chart
        data="demo.daily_orders"
        x="date"
        y="sum(total_sales)"
        filters=["sales_filter"]
        date_grain="month"
    /%}
    

### 

​

Using `where`

![Using where](https://mintcdn.com/evidence/ImoR9rIJqT9N0Esa/images/components/slider/example-1.png?fit=max&auto=format&n=ImoR9rIJqT9N0Esa&q=85&s=070bd1a6691f693a2451f0d5418e4b2e)
    
    
    {% slider
        id="sales_filter"
        data="demo.daily_orders"
        value_column="total_sales"
        range=true
    /%}
    
    {% table
        data="demo.daily_orders"
        where="total_sales {{sales_filter.between}}"
    /%}
    

### 

​

Using Inline SQL

![Using Inline SQL](https://mintcdn.com/evidence/ImoR9rIJqT9N0Esa/images/components/slider/example-2.png?fit=max&auto=format&n=ImoR9rIJqT9N0Esa&q=85&s=c15098be77e0cb137315b6a4146f8dd7)
    
    
    {% slider
        id="sales_filter"
        data="demo.daily_orders"
        value_column="total_sales"
        range=true
    /%}
    
    ```sql filtered_orders
    select * from demo.daily_orders
    where total_sales {{sales_filter.between}}
    ```
    
    {% table data="filtered_orders" /%}
    

## 

​

Attributes

​

string

required

The id of the slider to be used in a `filters` prop

​

string

Text displayed above the slider

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

string

Name of the table to query for min/max values

​

string

SQL expression to get min/max values from. When provided with data, queries MIN(value_column) and MAX(value_column) to set the slider range.

​

number

Minimum value for the slider

​

number

Maximum value for the slider

​

number

default:"1"

Step size for the slider (must be greater than 0)

​

boolean

default:"true"

If true, automatically adjusts min/max to align with step boundaries for cleaner numbers (e.g., range 15-103818 with step=10000 becomes 0-110000)

​

string

default:"num"

Format code for the slider values (e.g., “num”, “usd”, “pct”). See formatValue documentation for available formats. See [Value Formatting](</core-concepts/value-formatting>) for available formats.

​

boolean

default:"false"

If true, enables range mode with two handles for selecting a min/max range

​

number | array

Initial selected value (number for single value, [min, max] array for range mode)

​

boolean

default:"false"

If true, shows a number input next to the slider value for direct editing

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

Reference this filter using `{{filter_id}}`. The value returned depends on where you use it.

Context| Default Property| No Selection| Result  
---|---|---|---  
Inline SQL query| `.value`| | `500`  
`where` attribute| `.value`| | `500`  
Text / Markdown| `.value`| | `500`  
  
### 

​

Available Properties

You can also access specific properties using `{{filter_id.property}}`:

#### 

​

.value

Returns the numeric slider value, or the selected range when in range mode.
    
    
    {% slider id="age_filter" data="users" value_column="age" /%}
    
    ```sql filtered_users
    select * from users
    where age >= {{age_filter}}
    ```
    

**Example value:** `500`

#### 

​

.filter

Returns a complete SQL filter expression. Returns `true` when no value is selected. Only available when `value_column` is provided.
    
    
    {% slider id="age_filter" data="users" value_column="age" /%}
    
    ```sql filtered_users
    select * from users
    where {{age_filter.filter}}
    ```
    

**Example value:** `age >= 25`

#### 

​

.literal

Returns the raw numeric value. Only available in single value mode.
    
    
    {% slider id="price_filter" min=0 max=1000 /%}
    
    ```sql filtered_products
    select * from products
    where price >= {{price_filter.literal}}
    ```
    

**Example value:** `500`

#### 

​

.min

Returns the minimum value of the selected range. Only available when range mode is enabled.
    
    
    {% slider id="age_filter" data="users" value_column="age" range=true /%}
    
    ```sql filtered_users
    select * from users
    where age >= {{age_filter.min}}
    ```
    

**Example value:** `25`

#### 

​

.max

Returns the maximum value of the selected range. Only available when range mode is enabled.
    
    
    {% slider id="age_filter" data="users" value_column="age" range=true /%}
    
    ```sql filtered_users
    select * from users
    where age <= {{age_filter.max}}
    ```
    

**Example value:** `65`

#### 

​

.between

Returns a SQL BETWEEN clause fragment. Only available when range mode is enabled.
    
    
    {% slider id="price_filter" min=0 max=1000 range=true /%}
    
    ```sql filtered_products
    select * from products
    where sale_price {{price_filter.between}}
    ```
    

**Example value:** `BETWEEN 25 AND 65`

Was this page helpful?

⌘I
