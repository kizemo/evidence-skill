---
type: external-doc
title: Repeat
source: https://docs.evidence.dev/components/repeat
source_hash: 586dd1d603
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Repeat

> 原文: <https://docs.evidence.dev/components/repeat>

{% repeat 
    	id="category_repeat" 
    	data="demo.daily_orders" 
    	column="category" 
    %}
       {% line_chart
            data="demo.daily_orders"
            x="date"
            date_grain="month"
            y="sum(total_sales)"
            filters=["category_repeat"]
       /%}
    {% /repeat %}
    

## 

​

Examples

### 

​

Basic Usage
    
    
    {% repeat 
    	id="category_repeat" 
    	data="demo.daily_orders" 
    	column="category" 
    %}
       {% line_chart
            data="demo.daily_orders"
            x="date"
            date_grain="month"
            y="sum(total_sales)"
            filters=["category_repeat"]
       /%}
    {% /repeat %}
    

### 

​

With WHERE Clause
    
    
    {% repeat 
    	id="top_categories" 
    	data="demo.daily_orders" 
    	column="category"
    	where="total_sales > 1000"
    %}
       ### {{top_categories}}
       {% line_chart
            data="demo.daily_orders"
            x="date"
            y="sum(total_sales)"
            filters=["top_categories"]
       /%}
    {% /repeat %}
    

## 

​

Attributes

​

string

required

The id of this repeat component to be used in the `filters` prop of its children

​

string

required

The name of the table to query

​

string

required

The name of the column within the `data` table to get distinct values

​

string

SQL WHERE clause to filter which values to repeat over

​

array

Array of filter IDs to apply when querying for distinct values

Was this page helpful?

⌘I
