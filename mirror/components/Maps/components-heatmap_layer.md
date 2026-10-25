---
type: external-doc
title: Heatmap Layer
source: https://docs.evidence.dev/components/heatmap_layer
source_hash: 135f5847b3
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Heatmap Layer

> 原文: <https://docs.evidence.dev/components/heatmap_layer>

{% map %}
        {% heatmap_layer
            data="events"
            lat="latitude"
            lng="longitude"
        /%}
    {% /map %}
    

## 

​

Examples

### 

​

Basic Heatmap
    
    
    {% map %}
        {% heatmap_layer
            data="events"
            lat="latitude"
            lng="longitude"
        /%}
    {% /map %}
    

### 

​

Weighted Heatmap
    
    
    {% map %}
        {% heatmap_layer
            data="sales"
            lat="latitude"
            lng="longitude"
            weight="sum(revenue)"
        /%}
    {% /map %}
    

### 

​

Custom Color Palette
    
    
    {% map %}
        {% heatmap_layer
            data="incidents"
            lat="latitude"
            lng="longitude"
            color_scale=["#ffffb2", "#fecc5c", "#fd8d3c", "#f03b20", "#bd0026"]
        /%}
    {% /map %}
    

### 

​

Adjusted Radius and Intensity
    
    
    {% map %}
        {% heatmap_layer
            data="visits"
            lat="latitude"
            lng="longitude"
            weight="visit_count"
            radius=50
            intensity=2
        /%}
    {% /map %}
    

## 

​

Attributes

​

string

required

Name of the table to query

​

array

Array of filter IDs to apply

​

string

required

Column name for latitude values

​

string

required

Column name for longitude values

​

string

Column or expression for weighting points in the heatmap (e.g., “sum(sales)”). Higher values create more intense heat

​

number

default:"30"

Base radius of influence for each point in pixels (adjusts with zoom)

​

number

default:"1"

Intensity multiplier for the heatmap. Higher values create more pronounced heat

​

number

default:"0.8"

Opacity of the heatmap layer (0-1)

​

array

Array of colors for the heatmap gradient, from lowest to highest density. A single-color array auto-expands to [background, color]. Defaults to the theme color scale.

​

array

Deprecated. Use `color_scale`.

​

array

Zoom range [min, max] where this layer is visible (e.g., [0, 8] shows layer from zoom 0 to 8)

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

string

Custom SQL WHERE condition to apply to the query. For date filters, use date_range instead.

​

string

Custom SQL HAVING condition to apply to the query after GROUP BY

​

number

Maximum number of rows to return from the query. Note: When used with tables, limit will disable subtotals to prevent incomplete subtotal rows.

​

string

Column name(s) with optional direction (e.g. “column_name”, “column_name desc”)

​

string

Custom SQL QUALIFY condition to filter windowed results

## 

​

Allowed Parents

  * [map](</components/map>)

Was this page helpful?

⌘I
