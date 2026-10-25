---
type: external-doc
title: Row
source: https://docs.evidence.dev/components/row
source_hash: 928a1d2602
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Row

> 原文: <https://docs.evidence.dev/components/row>

<!-- 2 side-by-side charts -->
    {% row %}
        {% line_chart
            data="demo.daily_orders"
            x="date"
            date_grain="quarter"
            y="sum(total_sales)"
        /%}
        {% bar_chart
            data="demo.daily_orders"
            x="category"
            y="sum(transactions)"
        /%}
    {% /row %}
    

## 

​

Examples

### 

​

Basic Usage
    
    
    <!-- 2 side-by-side charts -->
    {% row %}
        {% line_chart
            data="demo.daily_orders"
            x="date"
            date_grain="quarter"
            y="sum(total_sales)"
        /%}
        {% bar_chart
            data="demo.daily_orders"
            x="category"
            y="sum(transactions)"
        /%}
    {% /row %}
    

### 

​

Heading with a button on the right
    
    
    {% row align="center" justify="between" %}
    # Sales Overview
    {% link_button url="https://example.com" title="Full report" /%}
    {% /row %}
    

### 

​

Heading with a filter on the right
    
    
    {% row align="center" justify="between" %}
    ## Orders by category
    {% dropdown id="category" data="demo.daily_orders" value_column="category" /%}
    {% /row %}
    

### 

​

Filter on the right
    
    
    {% row justify="end" %}
    {% dropdown id="category" data="demo.daily_orders" value_column="category" /%}
    {% /row %}
    

### 

​

Centered big values
    
    
    {% row justify="center" %}
    {% big_value data="demo.daily_orders" value="sum(total_sales)" title="Sales" /%}
    {% big_value data="demo.daily_orders" value="sum(transactions)" title="Transactions" /%}
    {% /row %}
    

### 

​

Setting an item’s width
    
    
    {% row justify="end" %}
    {% dropdown id="category" data="demo.daily_orders" value_column="category" width=40 /%}
    {% /row %}
    

## 

​

How items are sized

By default, items in a row stretch to share its width. Each component has a minimum width; items that don’t fit wrap onto the next line.

  * Charts, tables and other content blocks stretch to fill the row.
  * Inputs and big values also stretch, unless the row sets `justify` to `center`, `end` or `between` and the screen is at least 1024px wide (or the page is a PDF). Then they keep their minimum width (about 200px for inputs, 180px for big values) so `justify` can move them.
  * Buttons, text and headings keep their natural width.
  * Set `width` (a percentage) on an item to size it yourself.

`justify` only moves items when there is free space, so it has no visible effect on a row of charts. Rows wrap on narrow screens, and `justify` applies to each line. On screens narrower than 1024px, such as phones and tablets, inputs and big values stretch to the full width again. A `stack` inside a row stretches like a content block.

## 

​

Attributes

​

string

default:"stretch"

How to vertically align items in this row**Allowed values:**

  * `top`
  * `center`
  * `bottom`
  * `stretch`

​

string

default:"start"

How to place items horizontally when they leave free space. `start` packs them to the left, `center` to the middle, `end` to the right, and `between` pushes the first and last items to the edges. With `center`, `end` or `between`, inputs and big values keep their minimum width instead of stretching on screens at least 1024px wide and in PDFs. Charts and tables still fill the row.**Allowed values:**

  * `start`
  * `center`
  * `end`
  * `between`

​

boolean

default:"false"

Display the row contents as a single card when card mode is enabled

​

number

Set the width of this component (in percent) relative to the page width

Was this page helpful?

⌘I
