---
type: external-doc
title: If
source: https://docs.evidence.dev/components/if
source_hash: f02b74dc8b
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# If

> 原文: <https://docs.evidence.dev/components/if>

{% if data="demo.daily_orders" %}
    	Content
    {% /if %}
    

## 

​

Examples

### 

​

Basic Usage
    
    
    {% if data="demo.daily_orders" %}
    	Content
    {% /if %}
    

### 

​

With an else branch
    
    
    {% if data="demo.daily_orders" condition="has_rows" %}
    	There are orders.
    {% /if %}
    {% else %}
    	No orders found.
    {% /else %}
    

## 

​

Attributes

​

string

required

Table or view to query

​

array

IDs of filters to apply to the query

​

string

default:"has_rows"

Set to “no_rows” to render children only if the query returns no rows. Set to “has_rows” (or omit) to render if query returns rows.**Allowed values:**

  * `no_rows`
  * `has_rows`

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

Was this page helpful?

⌘I
