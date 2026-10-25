---
type: external-doc
title: Reference Line
source: https://docs.evidence.dev/components/reference_line
source_hash: af95512265
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Reference Line

> 原文: <https://docs.evidence.dev/components/reference_line>

![Horizontal Target Line](https://mintcdn.com/evidence/oSDKudj1ZJ3RGuev/images/components/reference_line/reference_line.png?fit=max&auto=format&n=oSDKudj1ZJ3RGuev&q=85&s=79f4f2da74679374240239ddaf0014de)
    
    
    {% line_chart
        data="demo.daily_orders"
        x="date"
        y="sum(total_sales)"
        date_grain="month"
    %}
        {% reference_line 
            y=3500000 
            label="Target" 
            color="red" 
        /%}
    {% /line_chart %}
    

## 

​

Examples

### 

​

Horizontal Target Line

![Horizontal Target Line](https://mintcdn.com/evidence/oSDKudj1ZJ3RGuev/images/components/reference_line/reference_line.png?fit=max&auto=format&n=oSDKudj1ZJ3RGuev&q=85&s=79f4f2da74679374240239ddaf0014de)
    
    
    {% line_chart
        data="demo.daily_orders"
        x="date"
        y="sum(total_sales)"
        date_grain="month"
    %}
        {% reference_line 
            y=3500000 
            label="Target" 
            color="red" 
        /%}
    {% /line_chart %}
    

### 

​

Vertical Line at Date

![Vertical Line at Date](https://mintcdn.com/evidence/oSDKudj1ZJ3RGuev/images/components/reference_line/example-1.png?fit=max&auto=format&n=oSDKudj1ZJ3RGuev&q=85&s=c1e1b0035b6ce99eb00cb9781268d9f6)
    
    
    {% line_chart
        data="demo.daily_orders"
        x="date"
        y="sum(total_sales)"
        date_grain="month"
    %}
        {% reference_line 
            x="2021-06-01" 
            label="Product Launch" 
            color="green" 
        /%}
    {% /line_chart %}
    

### 

​

Reference Lines from Data

![Reference Lines from Data](https://mintcdn.com/evidence/oSDKudj1ZJ3RGuev/images/components/reference_line/example-2.png?fit=max&auto=format&n=oSDKudj1ZJ3RGuev&q=85&s=cd4c125f21c273a97434fa199f6d1eeb)
    
    
    ```sql ref_lines
    select 2500000 as ref, 'Min Target' as ref_label
    union all
    select 4000000, 'Stretch Goal'
    ```
    
    {% bar_chart
        data="demo.daily_orders"
        x="date"
        y="sum(total_sales)"
        date_grain="month"
    %}
        {% reference_line 
            data="ref_lines" 
            y="ref" 
            label="ref_label" 
        /%}
    {% /bar_chart %}
    

### 

​

Custom Label Styling

![Custom Label Styling](https://mintcdn.com/evidence/oSDKudj1ZJ3RGuev/images/components/reference_line/example-3.png?fit=max&auto=format&n=oSDKudj1ZJ3RGuev&q=85&s=0417e2b1b63e98f7693d08cfc62f968e)
    
    
    {% line_chart
        data="demo.daily_orders"
        x="date"
        y="sum(total_sales)"
        date_grain="month"
    %}
        {% reference_line 
            y=3000000 
            label="Threshold" 
            color="orange"
            label_options={
                position="above_start"
                background_color="orange"
                color="white"
                padding=4
            }
            line_options={
                type="dashed"
                width=2
            }
        /%}
    {% /line_chart %}
    

### 

​

Multiple Reference Lines

![Multiple Reference Lines](https://mintcdn.com/evidence/oSDKudj1ZJ3RGuev/images/components/reference_line/example-4.png?fit=max&auto=format&n=oSDKudj1ZJ3RGuev&q=85&s=f00451efb7e6aed0f4f212de23366dfd)
    
    
    {% line_chart
        data="demo.daily_orders"
        x="date"
        y="sum(total_sales)"
        date_grain="month"
    %}
        {% reference_line y=2500000 label="Min Target" color="red" /%}
        {% reference_line y=4500000 label="Stretch Goal" color="green" /%}
    {% /line_chart %}
    

## 

​

Attributes

​

string

Query name to use for calculating dynamic reference values

​

array

IDs of filters to apply to the query (requires `data`)

​

string

Custom SQL WHERE condition to apply to the query. For date filters, use date_range instead.

​

string

Text label to display on the reference line

​

string | number

X-axis value for a vertical reference line (e.g., a date or category)

​

string | number

Y-axis value for a horizontal reference line (e.g., a target or threshold)

​

string | number

Starting x-coordinate for a sloped line

​

string | number

Starting y-coordinate for a sloped line

​

string | number

Ending x-coordinate for a sloped line

​

string | number

Ending y-coordinate for a sloped line

​

string

Color of the reference line

​

options group

Styling options for the label**Example:**
    
    
     label_options={
      position = "above_end"
      align = "left"
      color = "string"
      background_color = "string"
      padding = 0
      fmt = "date"
      hide_value = true
      border = {
        width = 0
        type = "solid"
        color = "string"
        radius = 0
      }
      text = {
        size = 0
        bold = true
        italic = true
      }
    }
    

**Attributes:**

  * position: `string`
    * **Allowed values:**
      * `above_end`
      * `above_start`
      * `above_center`
      * `below_end`
      * `below_start`
      * `below_center`
  * align: `string`
    * **Allowed values:**
      * `left`
      * `center`
      * `right`
  * color: `string`
  * background_color: `string`
  * padding: `number`
  * fmt: `string` \- Format the label value. Defaults to series or axis fmt.
    * **Allowed values:** See [Value Formatting](</core-concepts/value-formatting>) for all available formats.
  * hide_value: `boolean`
  * border: `options group`
    * **Options:**
      * width: `number`
      * type: `string`
        * **Allowed values:**
          * `solid`
          * `dashed`
          * `dotted`
      * color: `string`
      * radius: `number`
  * text: `options group`
    * **Options:**
      * size: `number`
      * bold: `boolean`
      * italic: `boolean`

​

options group

Styling options for the line itself**Example:**
    
    
     line_options={
      color = "string"
      width = 0
      type = "solid"
      opacity = 0
    }
    

**Attributes:**

  * color: `string`
  * width: `number` \- Width of the line
  * type: `string`
    * **Allowed values:**
      * `solid`
      * `dashed`
      * `dotted`
  * opacity: `number` \- Between 0 and 1

​

options group

Symbol shapes to display at line endpoints**Example:**
    
    
     symbols={
      start = {
        shape = "circle"
        size = 0
      }
      end = {
        shape = "circle"
        size = 0
      }
    }
    

**Attributes:**

  * start: `options group`
    * **Options:**
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
  * end: `options group`
    * **Options:**
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
