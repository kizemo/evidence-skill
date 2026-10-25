---
type: external-doc
title: Fence
source: https://docs.evidence.dev/components/fence
source_hash: 6d3325d493
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Fence

> 原文: <https://docs.evidence.dev/components/fence>

```language query_name
      content
    ```
    

## 

​

Examples

### 

​

Basic Usage
    
    
    ```language query_name
      content
    ```
    

### 

​

Define a SQL query
    
    
    ```sql electronics_orders
      SELECT * FROM demo.daily_orders WHERE category = 'Electronics'
    ```
    
    <!-- Use the query in other components -->
    {% table data="electronics_orders" /%}
    

## 

​

Attributes

​

string

The language of the code block

​

string

The content of the code block

​

string

Optional name for the code block. If provided and language is “sql”, this will register the query as an inline query that can be used by other components. Inline queries must be defined at the top level of the page, not inside other components (e.g. tabs, accordion, details).

​

number

Set the width of this component (in percent) relative to the page width

Was this page helpful?

⌘I
