---
type: external-doc
title: Area Layer
source: https://docs.evidence.dev/components/area_layer
source_hash: 38707ef78f
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Area Layer

> 原文: <https://docs.evidence.dev/components/area_layer>

{% map %}
        {% area_layer
            geography="us_states"
            match_by="name"
            data="state_sales"
            area_id="state"
            value="sum(sales)"
        /%}
    {% /map %}
    

## 

​

Examples

### 

​

US States by Name
    
    
    {% map %}
        {% area_layer
            geography="us_states"
            match_by="name"
            data="state_sales"
            area_id="state"
            value="sum(sales)"
        /%}
    {% /map %}
    

### 

​

US States by Abbreviation
    
    
    {% map %}
        {% area_layer
            geography="us_states"
            match_by="abbr"
            data="state_sales"
            area_id="state_abbr"
            value="sum(sales)"
        /%}
    {% /map %}
    

### 

​

US Counties by State + County
    
    
    {% map %}
        {% area_layer
            geography="us_counties"
            match_by="state-county"
            data="county_sales"
            area_id="state || '-' || county"
            value="sum(sales)"
        /%}
    {% /map %}
    

### 

​

US Counties by FIPS
    
    
    {% map %}
        {% area_layer
            geography="us_counties"
            match_by="fips"
            data="county_sales"
            area_id="state_fips || county_fips"
            value="sum(sales)"
        /%}
    {% /map %}
    

### 

​

Custom GeoJSON
    
    
    {% area_layer
        geojson_url="https://example.com/custom.geojson"
        geojson_id="id"
        data="my_data"
        area_id="region_id"
        value="sum(sales)"
    /%}
    

### 

​

Diverging Scale Centered at 0
    
    
    {% map %}
        {% area_layer
            geography="us_states"
            match_by="abbr"
            data="state_growth"
            area_id="state_abbr"
            value="growth_pct"
            color_scale=["#d73027", "#ffffbf", "#1a9850"]
            min=-100
            max=100
            midpoint=0
        /%}
    {% /map %}
    

### 

​

Custom Breakpoints
    
    
    {% map %}
        {% area_layer
            geography="us_states"
            match_by="abbr"
            data="state_sales"
            area_id="state_abbr"
            value="sum(sales)"
            color_stops=[
                { value=0 color="#ffffff" },
                { value=10000 color="#f59e0b" },
                { value=50000 color="#1a9850" }
            ]
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

Pre-provided geography (use this OR geojson_url + geojson_id)**Allowed values:**

  * `us_states`
  * `us_counties`

​

string

How to match areas. For us_states: “name”, “abbr”, or “fips”. For us_counties: “state-county” or “fips”.

​

string

URL to custom GeoJSON file (use with geojson_id for custom maps)

​

string

GeoJSON property to join on. Use a string for single property (e.g., “NAME”) or array for composite key (e.g., [“STATE”, “COUNTY”]).

​

string

required

Column name in data that matches geo_id (e.g., “state_id”)

​

string

required

Column or expression for coloring the choropleth (e.g., “sum(sales)”)

​

array

Array of colors for the choropleth gradient. A single-color array auto-expands to [background, color].

​

array

Deprecated. Use `color_scale`.

​

number

Lower bound for the color scale. Values below this clamp to the first color in the scale. Defaults to the minimum value in the data.

​

number

Upper bound for the color scale. Values above this clamp to the last color in the scale. Defaults to the maximum value in the data.

​

number

Anchor a specific value (typically 0) at the middle of a diverging color scale. Requires a color_scale with 3 or more colors.

​

array

Pin specific data values to specific colors, interpolating between them. Provide an array of value and color pairs. Values beyond the first and last breakpoint are clamped to the end colors. Takes precedence over color_scale, min, max, and midpoint.

​

boolean

default:"true"

Whether to show areas that do not have matching data

​

boolean

default:"true"

Show tooltips on hover

​

array

Array of SQL expressions for additional fields to show in tooltip (e.g., [“category”, “emissions”])

​

string

GeoJSON property to use for area name in tooltip (defaults to “NAME”)

​

string

default:"num"

Format for values in tooltip. See [Value Formatting](</core-concepts/value-formatting>) for available formats.

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
