---
type: external-doc
title: Else If
source: https://docs.evidence.dev/components/else_if
source_hash: 8048bfaf1b
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Else If

> 原文: <https://docs.evidence.dev/components/else_if>

{% if data="demo.daily_orders" condition="no_rows" %}
    	If Content
    {% /if %}
    {% else_if data="demo.daily_orders" %}
    	Else If Content
    {% /else_if %}
    

## 

​

Examples

### 

​

Basic Usage
    
    
    {% if data="demo.daily_orders" condition="no_rows" %}
    	If Content
    {% /if %}
    {% else_if data="demo.daily_orders" %}
    	Else If Content
    {% /else_if %}
    

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
