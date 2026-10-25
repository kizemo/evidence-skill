---
type: external-doc
title: Progress Bars
source: https://docs.evidence.dev/components/progress_bars
source_hash: 994fcf428d
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Progress Bars

> 原文: <https://docs.evidence.dev/components/progress_bars>

{% progress_bars
      data="staff"
      dimension="provider_type"
      numerator="count(*) FILTER (WHERE on_shift = true)"
      denominator="count(*)"
      title="Provider Staffing"
    /%}
    

## 

​

Examples

### 

​

Basic Usage
    
    
    {% progress_bars
      data="staff"
      dimension="provider_type"
      numerator="count(*) FILTER (WHERE on_shift = true)"
      denominator="count(*)"
      title="Provider Staffing"
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

Column for row labels (e.g., provider_type)

​

string

required

SQL aggregation for the filled portion (e.g., count(*) FILTER (WHERE active))

​

string

SQL aggregation for the total (e.g., count(*)). Omitted in metric mode — the metric supplies the default denominator (its own aggregate).

​

string

Format code for numerator and denominator values. See [Value Formatting](</core-concepts/value-formatting>) for available formats.

​

string

Component title

​

array

Two-color array defining the gradient for bars and badges, e.g. [“#ef4444”, “#22c55e”]. First color maps to 0%, second to 100%.

​

array

Two percentage thresholds (0-1) creating 3 color zones: red below first, amber between, green above second. e.g. [0.5, 0.8].

​

boolean

If true, low percentages are green and high percentages are red. Default (false) means high percentages are green. Only applies when thresholds are set.

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
