---
type: external-doc
title: Reference Point
source: https://docs.evidence.dev/components/reference_point
source_hash: eafcef1a93
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Reference Point

> 原文: <https://docs.evidence.dev/components/reference_point>

![Hardcoded Point](https://mintcdn.com/evidence/oSDKudj1ZJ3RGuev/images/components/reference_point/reference_point.png?fit=max&auto=format&n=oSDKudj1ZJ3RGuev&q=85&s=b4863f8d05ff90cef5cbfb2c84c6ec4d)
    
    
    {% line_chart
        data="demo.daily_orders"
        x="date"
        y="sum(total_sales)"
        date_grain="month"
    %}
        {% reference_point
            x="2024-12-01"
            y=4441307
            label="Peak Month"
            color="green"
            label_options={
                position="left"
            }
        /%}
    {% /line_chart %}
    

## 

​

Examples

### 

​

Hardcoded Point

![Hardcoded Point](https://mintcdn.com/evidence/oSDKudj1ZJ3RGuev/images/components/reference_point/reference_point.png?fit=max&auto=format&n=oSDKudj1ZJ3RGuev&q=85&s=b4863f8d05ff90cef5cbfb2c84c6ec4d)
    
    
    {% line_chart
        data="demo.daily_orders"
        x="date"
        y="sum(total_sales)"
        date_grain="month"
    %}
        {% reference_point
            x="2024-12-01"
            y=4441307
            label="Peak Month"
            color="green"
            label_options={
                position="left"
            }
        /%}
    {% /line_chart %}
    

### 

​

Point from Data

![Point from Data](https://mintcdn.com/evidence/oSDKudj1ZJ3RGuev/images/components/reference_point/example-1.png?fit=max&auto=format&n=oSDKudj1ZJ3RGuev&q=85&s=6e5a97544caf4d8fd208693087beeccd)
    
    
    ```sql top_months
    select 
        toStartOfMonth(date) as month,
        sum(total_sales) as sales,
        'Top ' || row_number() over (order by sum(total_sales) desc) as label
    from demo.daily_orders
    group by month
    order by sales desc
    limit 3
    ```
    
    {% line_chart
        data="demo.daily_orders"
        x="date"
        y="sum(total_sales)"
        date_grain="month"
    %}
        {% reference_point 
            data="top_months"
            x="month" 
            y="sales" 
            label="label"
            color="purple"
        /%}
    {% /line_chart %}
    

### 

​

Callout Style

![Callout Style](https://mintcdn.com/evidence/oSDKudj1ZJ3RGuev/images/components/reference_point/example-2.png?fit=max&auto=format&n=oSDKudj1ZJ3RGuev&q=85&s=43a8ed4f89dcb4dd18b547bf45b6c4dc)
    
    
    {% line_chart
        data="demo.daily_orders"
        x="date"
        y="sum(total_sales)"
        date_grain="month"
    %}
        {% reference_point 
            x="2021-02-01" 
            y=1754436 
            label="Lowest Month"
            color="red"
            label_options={
                variant="callout"
                position="top"
            }
        /%}
    {% /line_chart %}
    

### 

​

Custom Symbol

![Custom Symbol](https://mintcdn.com/evidence/oSDKudj1ZJ3RGuev/images/components/reference_point/example-3.png?fit=max&auto=format&n=oSDKudj1ZJ3RGuev&q=85&s=855769c6e5c21de2de7e398d682a85b3)
    
    
    {% line_chart
        data="demo.daily_orders"
        x="date"
        y="sum(total_sales)"
        date_grain="month"
    %}
        {% reference_point 
            x="2023-12-01" 
            y=3720260 
            label="Record High"
            symbol_options={
                shape="diamond"
                size=12
                color="gold"
            }
            label_options={
                position="right"
                color="orange"
            }
        /%}
    {% /line_chart %}
    

## 

​

Attributes

​

string

Query name to use for placing multiple points from data

​

array

IDs of filters to apply to the query (requires `data`)

​

string

Custom SQL WHERE condition to apply to the query. For date filters, use date_range instead.

​

string

Text label to display at the reference point

​

string

Color of the point marker

​

string | number

required

X-axis position of the point (e.g., a date or category)

​

string | number

required

Y-axis position of the point (e.g., a value)

​

options group

Styling options for the label

​

options group

Styling options for the point marker**Example:**
    
    
     symbol_options={
      shape = "circle"
      size = 0
      color = "string"
    }
    

**Attributes:**

  * shape: `string`
    * **Allowed values:**
      * `circle`
      * `emptyCircle`
      * `rect`
      * `roundRect`
      * `triangle`
      * `diamond`
      * `pin`
      * `arrow`
      * `none`
      * `image://`
      * `path://`
  * size: `number`
  * color: `string`

## 

​

Allowed Parents

  * [area_chart](</components/area_chart>)
  * [bar_chart](</components/bar_chart>)
  * [bubble_chart](</components/bubble_chart>)
  * [combo_chart](</components/combo_chart>)
  * [horizontal_bar_chart](</components/horizontal_bar_chart>)
  * [line_chart](</components/line_chart>)
  * [scatter_chart](</components/scatter_chart>)

Was this page helpful?

⌘I
