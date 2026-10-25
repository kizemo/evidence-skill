---
type: external-doc
title: Area
source: https://docs.evidence.dev/components/area
source_hash: 0f2abaca43
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Area

> 原文: <https://docs.evidence.dev/components/area>

{% combo_chart data="demo.daily_orders" x="date" date_grain="month" %}
        {% area y="sum(total_sales)" /%}
    {% /combo_chart %}
    

## 

​

Examples

### 

​

Basic Area Series
    
    
    {% combo_chart data="demo.daily_orders" x="date" date_grain="month" %}
        {% area y="sum(total_sales)" /%}
    {% /combo_chart %}
    

### 

​

Unstacked Areas
    
    
    {% combo_chart data="demo.daily_orders" x="date" date_grain="month" %}
        {% area y="sum(total_sales)" series="category" stacked=false /%}
    {% /combo_chart %}
    

### 

​

Area with Gradient Fill
    
    
    {% combo_chart data="demo.daily_orders" x="date" date_grain="month" %}
        {% area y="sum(total_sales)" stacked=false options={
            gradient=true
        } /%}
    {% /combo_chart %}
    

## 

​

Attributes

​

string

Column name for y-axis. Aggregated at the parent combo_chart’s `x=`. Omit when using `metric`.

​

string

Semantic metric name to display, from metrics/*.yaml. Use instead of the raw data/value/x/y attributes.

​

string

Column name for series grouping

​

string

default:"y1"

The axis to render the series on**Allowed values:**

  * `y1`
  * `y2`

​

string

Format for this series’ values. See [Value Formatting](</core-concepts/value-formatting>) for available formats.

​

array

Extra columns to include in the tooltip on hover. Each entry is `{ value, label?, fmt?, color_by_sign?, down_is_good? }`. See the [tooltip fields guide](</components/tooltip-fields>) for examples.

​

options group

Label each point in the series with its value**Example:**
    
    
     data_labels={
      position = "above"
      fmt = "date"
      size = 0
      distance = 0
      rotate = 0
      color = "string"
      border_color = "string"
      show_overlap = true
    }
    

**Attributes:**

  * position: `string` \- Position the label relative to its data point
    * **Allowed values:**
      * `above`
      * `below`
      * `left`
      * `right`
      * `middle`
  * fmt: `string` \- Format the label value. Defaults to series or axis fmt.
    * **Allowed values:** See [Value Formatting](</core-concepts/value-formatting>) for all available formats.
  * size: `number` \- Font size in px
  * distance: `number` \- How far the label is from the data point
  * rotate: `number` \- Rotate each label (degrees)
  * color: `string` \- Change the text color of the labels
  * border_color: `string` \- Change the border color surrounding text labels, defaults to chart background
  * show_overlap: `boolean` \- Show labels for every point even when they overlap

​

options group

**Example:**
    
    
     options={
      step = "start"
      smooth = true
      gradient = true
    }
    

**Attributes:**

  * step: `string` \- Show a stepped line rather than a smooth line between points and control where the step happens (start, middle, or end)
    * **Allowed values:**
      * `start`
      * `middle`
      * `end`
  * smooth: `boolean`
  * gradient: `boolean` \- Apply a color-to-transparent gradient fill to the area, fading from the series color at the top to transparent at the bottom

​

map

Raw [ECharts series options](<https://echarts.apache.org/en/option.html#series>) deep-merged into this series only. For overrides that apply to every series on the chart, set `echarts_series_options` on the parent chart instead.**Example:**
    
    
     echarts_options={
        endLabel={ show=true }
        markPoint={ data=[{ type="max" }, { type="min" }] }
    }
    

​

boolean | string

default:"true"

Whether to stack areas with the same series value. Set to “100%” for percentage stacking.

​

string

Stack identifier - areas with the same stack_id value will be stacked together. Overrides the stacked prop.

## 

​

Allowed Parents

  * [combo_chart](</components/combo_chart>)

Was this page helpful?

⌘I
