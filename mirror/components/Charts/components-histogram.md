---
type: external-doc
title: Histogram
source: https://docs.evidence.dev/components/histogram
source_hash: cd1a3c74f1
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Histogram

> 原文: <https://docs.evidence.dev/components/histogram>

![Basic Usage](https://mintcdn.com/evidence/oSDKudj1ZJ3RGuev/images/components/histogram/histogram.png?fit=max&auto=format&n=oSDKudj1ZJ3RGuev&q=85&s=454a91cad5e5ce865def1268d0181025)
    
    
    {% histogram
        data="demo.daily_orders"
        value="transactions"
        bin_count=30
    /%}
    

## 

​

Examples

### 

​

Basic Usage

![Basic Usage](https://mintcdn.com/evidence/oSDKudj1ZJ3RGuev/images/components/histogram/histogram.png?fit=max&auto=format&n=oSDKudj1ZJ3RGuev&q=85&s=454a91cad5e5ce865def1268d0181025)
    
    
    {% histogram
        data="demo.daily_orders"
        value="transactions"
        bin_count=30
    /%}
    

## 

​

Attributes

​

string

required

Name of the table to query

​

string

required

SQL expression for the values to create histogram for

​

string

Column name to group data by series

​

options group

Chart configuration options**Example:**
    
    
     chart_options={
      color_palette = ["value1", "value2"]
    }
    

**Attributes:**

  * color_palette: `array of strings`

​

number

default:"20"

Number of bins for the histogram (takes precedence over bin_width)

​

number

Width of each bin (ignored if bin_count is specified)

​

string

Format for the column values displayed in bin ranges. See [Value Formatting](</core-concepts/value-formatting>) for available formats.

​

array

IDs of filters to apply to the query

​

string

Title to display above the histogram

​

string

Subtitle to display below the title

​

string

Info text to display in a tooltip next to the title. Can only be used with the title prop.

​

string

URL to link the info text to (can only be used with info)

​

string

Create a custom link title for the info link, placed after the info text (can only be used with info_link)

​

boolean

default:"true"

Show legend

​

string

default:"top"

Position of the legend (top or bottom)**Allowed values:**

  * `top`
  * `bottom`

​

number

Time in seconds between automatic data refreshes (minimum 30). Overrides the page-level auto-refresh setting for this component.

​

string

Custom SQL WHERE condition to apply to the query. For date filters, use date_range instead.

​

number

Maximum number of rows to return from the query. Note: When used with tables, limit will disable subtotals to prevent incomplete subtotal rows.

​

number

Set the width of this component (in percent) relative to the page width

​

number

Set a fixed height for the chart in pixels

​

string

Link this chart to others sharing the same id, syncing their tooltips, axis-pointer, and zoom

​

map

Raw [ECharts options](<https://echarts.apache.org/en/option.html>) deep-merged over the chart’s final configuration. Use for anything the structured props do not expose — `graphic`, `visualMap`, tooltip styling, and so on. Partial overrides win key-by-key without clobbering Studio’s computed siblings. For overrides scoped to the data series, use `echarts_series_options`.**Example:**
    
    
     echarts_options={
        tooltip={ position="top" }
        graphic=[{ type="text" }]
    }
    

​

map

Raw [ECharts series options](<https://echarts.apache.org/en/option.html#series>) deep-merged into the chart series. Use for series-level styling the structured props do not expose.**Example:**
    
    
     echarts_series_options={
        itemStyle={ borderRadius=8 }
    }
    

Was this page helpful?

⌘I
