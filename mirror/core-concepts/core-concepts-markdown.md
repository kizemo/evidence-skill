---
type: external-doc
title: Markdown
source: https://docs.evidence.dev/core-concepts/markdown
source_hash: 4fbb9e7c03
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Markdown

> 原文: <https://docs.evidence.dev/core-concepts/markdown>

Evidence supports almost all Markdown syntax.
    
    
    ---
    title: Evidence uses Markdown
    ---
    
    Markdown can be used to write expressively in text.
    
    - it supports lists,
    - **bolding**, _italics_ and `inline code`,
    - links to [external sites](https://google.com) and other [Evidence pages](/another/page)
    

## 

​

Components

Evidence has a built in component library to create charts and other visual elements. The components use [Markdoc syntax](<https://markdoc.dev/docs/syntax>).
    
    
    {% line_chart
        data="demo.daily_orders"
        x="date"
        y="sum(sales)"
        date_grain="month"
    %}
    

The easiest way to add a component is using a slash command, and starting to type the name - press tab to select it.

## 

​

SQL

Code fences in Evidence markdown pages run queries and return data. These code fences run the [ClickHouse SQL dialect](<https://clickhouse.com/docs/sql-reference/syntax>) (unless you are using a [Direct connector](</evidence-warehouse#direct-connectors>)).
    
    
    ```sql ten_daily_orders
    select * from demo.daily_orders limit 10
    ```
    

You can then use the data in your markdown using the `data` attribute.
    
    
    {% table
        data="ten_daily_orders"
    %}
    

### 

​

Inline Query References

You can reference other inline queries within your SQL using the `{{query_name}}` syntax. The referenced query will be wrapped in parentheses and substituted in place.
    
    
    ```sql top_categories
    select category, sum(sales) as total_sales
    from demo.order_details
    group by category
    order by total_sales desc
    limit 5
    ```
    
    ```sql category_breakdown
    select
        category,
        item_name,
        sum(sales) as item_sales
    from demo.order_details
    where category in (select category from {{top_categories}})
    group by category, item_sales desc
    order by category, item_sales desc
    ```
    

Was this page helpful?

⌘I
