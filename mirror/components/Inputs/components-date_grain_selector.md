---
type: external-doc
title: Date Grain Selector
source: https://docs.evidence.dev/components/date_grain_selector
source_hash: be77488fe9
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Date Grain Selector

> 原文: <https://docs.evidence.dev/components/date_grain_selector>

![Using date_grain](https://mintcdn.com/evidence/Z2CIIsynbphd2HX3/images/components/date_grain_selector/date_grain_selector.png?fit=max&auto=format&n=Z2CIIsynbphd2HX3&q=85&s=c7ad2e83d3447067c683a5ae1f474da0)
    
    
    {% date_grain_selector
        id="time_grain"
        default_value="month"
    /%}
    
    {% line_chart
        data="demo.daily_orders"
        x="date"
        y="sum(total_sales)"
        date_grain={{time_grain}}
    /%}
    

## 

​

Examples

### 

​

Using `date_grain`

![Using date_grain](https://mintcdn.com/evidence/Z2CIIsynbphd2HX3/images/components/date_grain_selector/date_grain_selector.png?fit=max&auto=format&n=Z2CIIsynbphd2HX3&q=85&s=c7ad2e83d3447067c683a5ae1f474da0)
    
    
    {% date_grain_selector
        id="time_grain"
        default_value="month"
    /%}
    
    {% line_chart
        data="demo.daily_orders"
        x="date"
        y="sum(total_sales)"
        date_grain={{time_grain}}
    /%}
    

### 

​

Using Inline SQL

![Using Inline SQL](https://mintcdn.com/evidence/Z2CIIsynbphd2HX3/images/components/date_grain_selector/example-1.png?fit=max&auto=format&n=Z2CIIsynbphd2HX3&q=85&s=28ac9734b1df60a70b6633f6b9e796df)
    
    
    {% date_grain_selector
        id="time_grain"
        default_value="month"
    /%}
    
    ```sql sales_by_period
    select 
        date_trunc({{time_grain}}, date) as period,
        sum(total_sales) as total_sales
    from demo.daily_orders
    group by 1
    order by 1
    ```
    
    {% line_chart
        data="sales_by_period"
        x="period"
        y="total_sales"
    /%}
    

## 

​

Attributes

​

string

required

The id of the date grain selector to be used in SQL query templates

​

array

Optional array of preset date grain values to show. If not provided, all date grain options will be available.

​

string

Default date grain to select on load**Allowed values:**

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

string

default:"Date Grain"

Text displayed above the selector

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

Placeholder text displayed when no value is selected

​

string

default:"clock"

Icon to display

​

number

Set the width of this component (in percent) relative to the page width

## 

​

Using the Filter Variable

Reference this filter using `{{filter_id}}`. The value returned depends on where you use it.

Context| Default Property| No Selection| Result  
---|---|---|---  
Inline SQL query| `.selected`| `''`| `'month'`  
`where` attribute| `.selected`| `''`| `'month'`  
Text / Markdown| `.literal`| | `month`  
  
### 

​

Available Properties

You can also access specific properties using `{{filter_id.property}}`:

#### 

​

.selected

Returns the selected date grain value with quotes. Returns an empty string when no value is selected.
    
    
    {% date_grain_selector id="grain" preset_values=["day", "week", "month"] /%}
    
    ```sql sales_by_grain
    select 
      toStartOf{{grain.selected}}(date) as period,
      sum(sales) as total_sales
    from orders
    group by period
    ```
    

**Example value:** `'month'`

#### 

​

.literal

Returns the raw unescaped selected value. Use this with the `date_grain` attribute.
    
    
    {% date_grain_selector id="grain" preset_values=["day", "week", "month"] /%}
    
    ```sql sales_by_grain
    select 
      {{grain.literal}} as period,
      sum(sales) as total_sales
    from orders
    group by period
    ```
    

**Example value:** `month`

Was this page helpful?

⌘I
