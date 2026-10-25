---
type: external-doc
title: Benchmark Comparison
source: https://docs.evidence.dev/components/benchmark_comparison
source_hash: 31694e3774
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Benchmark Comparison

> 原文: <https://docs.evidence.dev/components/benchmark_comparison>

{% benchmark_comparison 
        name="vs Franchisees" 
        agg="avg" 
        subject="store_id"
        where="ownership_type = 'franchise'"
    /%}
    

## 

​

Examples

### 

​

Compare vs Franchisees
    
    
    {% benchmark_comparison 
        name="vs Franchisees" 
        agg="avg" 
        subject="store_id"
        where="ownership_type = 'franchise'"
    /%}
    

### 

​

Compare vs Full Network
    
    
    {% benchmark_comparison 
        name="vs Full Network" 
        agg="avg" 
        subject="store_id"
    /%}
    

### 

​

Compare vs Region Average
    
    
    {% benchmark_comparison 
        name="vs Region Avg" 
        agg="avg" 
        subject="store_id"
        within=["region"]
    /%}
    

## 

​

Attributes

​

string

required

Display name shown in the dropdown (e.g., “vs Franchisees”)

​

string

required

Aggregation function to apply across benchmark entities. Options: avg, median, min, max, sum, count, count_distinct**Allowed values:**

  * `avg`
  * `median`
  * `min`
  * `max`
  * `sum`
  * `count`
  * `count_distinct`

​

string

required

Column that defines individual entities (e.g., “store_id”, “customer_id”). Required for benchmark calculations.

​

string

Optional column or expression to use for benchmark calculation. If not specified, uses the main value column. Useful if you have a pre-aggregated benchmark table for RLS reasons.

​

string

SQL WHERE clause to filter which entities are included in the benchmark (e.g., “ownership_type = ‘franchise’”)

​

array

Dimension columns to group the benchmark by (e.g., [“region”]). Leave empty for dataset-wide benchmark.

​

boolean

default:"false"

Exclude the current row from its own benchmark calculation (table context only). Default: false

​

string

Default display type for this comparison. Options: pct (percentage change), abs (absolute change), compared_value (benchmark value)**Allowed values:**

  * `pct`
  * `abs`
  * `compared_value`

​

string

Custom comparison label text (overrides default “vs ”)

​

string

Format code for percentage values. See [Value Formatting](</core-concepts/value-formatting>) for available formats.

​

string

Format code for absolute values. See [Value Formatting](</core-concepts/value-formatting>) for available formats.

​

boolean

If true, negative changes are shown as positive (green). Defaults to the page’s `down_is_good` setting.

## 

​

Allowed Parents

  * [comparison_selector](</components/comparison_selector>)

Was this page helpful?

⌘I
