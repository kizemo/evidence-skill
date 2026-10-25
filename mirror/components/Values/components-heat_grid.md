---
type: external-doc
title: Heat Grid
source: https://docs.evidence.dev/components/heat_grid
source_hash: a1166aa8b1
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Heat Grid

> 原文: <https://docs.evidence.dev/components/heat_grid>

{% heat_grid
      data="appointments"
      dimension="province"
      value="avg(wait_time)"
      thresholds=[60, 90]
      units="MIN"
      lower_is_better=true
    /%}
    

## 

​

Examples

### 

​

Basic Usage
    
    
    {% heat_grid
      data="appointments"
      dimension="province"
      value="avg(wait_time)"
      thresholds=[60, 90]
      units="MIN"
      lower_is_better=true
    /%}
    

## 

​

Attributes

​

string

Table or view to query. Omit when using `metric`.

​

string

Semantic metric name to display, from metrics/*.yaml. Use instead of the raw data/value/x/y attributes.

​

string

required

Column for cell labels (e.g., province)

​

string

SQL aggregation expression for the metric (e.g., avg(wait_time)). Omit when using `metric`.

​

array

required

Two numeric thresholds creating 3 color zones: [low, high]. Values below low are zone 1, between low and high are zone 2, at or above high are zone 3.

​

string

Units label shown below the value in each cell (e.g., “MIN”)

​

string

Format code for the metric value. See [Value Formatting](</core-concepts/value-formatting>) for available formats.

​

string

Component title

​

boolean

If true, low values are green and high values are red. Default (false) means high values are green.

​

boolean

If true, removes gaps and borders between cells and applies a single border radius to the outer grid.

​

array

IDs of filters to apply to the query

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

​

number

Set the width of this component (in percent) relative to the page width

Was this page helpful?

⌘I
