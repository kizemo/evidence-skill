---
type: external-doc
title: Point Layer
source: https://docs.evidence.dev/components/point_layer
source_hash: a99dfc48cc
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Point Layer

> 原文: <https://docs.evidence.dev/components/point_layer>

{% map %}
        {% point_layer
            data="store_locations"
            lat="latitude"
            lng="longitude"
            point_title="store_name"
        /%}
    {% /map %}
    

## 

​

Examples

### 

​

Basic Point Map
    
    
    {% map %}
        {% point_layer
            data="store_locations"
            lat="latitude"
            lng="longitude"
            point_title="store_name"
        /%}
    {% /map %}
    

### 

​

Colored by Value
    
    
    {% map %}
        {% point_layer
            data="store_sales"
            lat="latitude"
            lng="longitude"
            point_title="store_name"
            color_value="sum(sales)"
            color_palette=["#feedde", "#fdd0a2", "#fdae6b", "#fd8d3c", "#e6550d", "#a63603"]
            color_value_fmt="usd"
        /%}
    {% /map %}
    

### 

​

Sized by Value
    
    
    {% map %}
        {% point_layer
            data="store_locations"
            lat="latitude"
            lng="longitude"
            point_title="store_name"
            size_value="sum(customers)"
        /%}
    {% /map %}
    

### 

​

Color and Size by Different Values
    
    
    {% map %}
        {% point_layer
            data="store_metrics"
            lat="latitude"
            lng="longitude"
            color_value="sum(sales)"
            size_value="sum(customers)"
            color_palette=["#feedde", "#fdd0a2", "#fdae6b", "#fd8d3c", "#e6550d", "#a63603"]
        /%}
    {% /map %}
    

### 

​

Categorical Coloring
    
    
    {% point_layer
        data="locations"
        lat="latitude"
        lng="longitude"
        point_title="name"
        color_value="category"
        color_palette=["#154886", "#45a1bf", "#a5cdee", "#8dacbf", "#85c7c6"]
    /%}
    

### 

​

Custom Tooltip Fields
    
    
    {% map %}
        {% point_layer
            data="power_plants"
            lat="latitude"
            lng="longitude"
            point_title="name"
            point_subtitle="region"
            tooltip_fields=["fuel_type", "capacity_mw", "emissions"]
        /%}
    {% /map %}
    

### 

​

Clustered (large datasets)
    
    
    {% map %}
        {% point_layer
            data="stations"
            lat="latitude"
            lng="longitude"
            cluster=true
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

Column or expression for coloring points. Numeric values create a gradient scale, categorical values (strings) assign discrete colors from the palette. Examples: “sum(sales)” (numeric gradient), “category” (categorical colors)

​

string

Column or expression for sizing points (e.g., “sum(customers)”). If not provided, all points will be the same size

​

string

Column name to use as the point title in tooltips (e.g., “city_name”). If not provided, lat/lng will be shown

​

string

Column name to use as the point subtitle in tooltips (e.g., “region”). Displays as a second line with muted text

​

string

default:"circle"

Shape of the point marker**Allowed values:**

  * `circle`
  * `pin`
  * `square`
  * `triangle`
  * `star`
  * `diamond`

​

boolean

default:"false"

Group nearby points into aggregated bubbles that split apart as you zoom in. Recommended for large datasets (thousands of points); raises the point fetch limit so there is enough data to cluster.

​

string

Single color for all points (hex, rgb/rgba, or CSS color name)

​

array

Array of colors for coloring. For numeric color_value: creates gradient. For categorical color_value: assigns discrete colors to categories (cycles if more categories than colors)

​

number

Lower bound for the numeric color scale. Values below this clamp to the first color in the palette. Ignored for categorical color_value. Defaults to the minimum value in the data.

​

number

Upper bound for the numeric color scale. Values above this clamp to the last color in the palette. Ignored for categorical color_value. Defaults to the maximum value in the data.

​

number

Anchor a specific value (typically 0) at the middle of a diverging color palette. Requires a numeric color_value and a color_palette with 3 or more colors.

​

array

Pin specific data values to specific colors for a numeric color_value, interpolating between them. Provide an array of value and color pairs. Values beyond the first and last breakpoint are clamped to the end colors. Takes precedence over color_palette, min, max, and midpoint. Ignored for categorical color_value.

​

number

default:"6"

Base size of points in pixels

​

number

default:"1"

Scale multiplier for value-based sizing. Higher values create larger size differences

​

boolean

default:"true"

Show tooltips on hover

​

array

Array of SQL expressions for additional fields to show in tooltip (e.g., [“category”, “emissions”])

​

string

default:"num"

Format for color values in tooltip. See [Value Formatting](</core-concepts/value-formatting>) for available formats.

​

string

default:"num"

Format for size values in tooltip. See [Value Formatting](</core-concepts/value-formatting>) for available formats.

​

array

Zoom range [min, max] where this layer is visible (e.g., [0, 8] shows layer from zoom 0 to 8)

​

boolean

default:"true"

Show legend for this layer

​

string

Custom label for the legend (defaults to table name)

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
