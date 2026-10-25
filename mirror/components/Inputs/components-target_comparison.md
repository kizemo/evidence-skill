---
type: external-doc
title: Target Comparison
source: https://docs.evidence.dev/components/target_comparison
source_hash: 145a8c7e56
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Target Comparison

> 原文: <https://docs.evidence.dev/components/target_comparison>

{% target_comparison 
        name="vs Budget" 
        target="budget_amount"
    /%}
    

## 

​

Examples

### 

​

Compare vs Budget
    
    
    {% target_comparison 
        name="vs Budget" 
        target="budget_amount"
    /%}
    

### 

​

Compare vs Fixed Goal
    
    
    {% target_comparison 
        name="vs 100K Goal" 
        target="100000"
    /%}
    

### 

​

Compare vs Aggregated Target
    
    
    {% target_comparison 
        name="vs Target" 
        target="sum(target_sales)"
    /%}
    

## 

​

Attributes

​

string

required

Display name shown in the dropdown (e.g., “vs Budget”)

​

string

required

Target value for comparison. Can be a column name, aggregation (e.g., “sum(budget)”), or literal value.

​

string

Default display type for this comparison. Options: pct (percentage change), abs (absolute change), compared_value (target value)**Allowed values:**

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
