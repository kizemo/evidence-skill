---
type: external-doc
title: Stack
source: https://docs.evidence.dev/components/stack
source_hash: 9985db961a
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Stack

> 原文: <https://docs.evidence.dev/components/stack>

<!-- 2 big values stacked vertically -->
    {% stack  %}
        {% big_value
            data="demo.daily_orders"
            value="sum(total_sales)"
        /%}
        {% big_value
            data="demo.daily_orders"
            value="avg(avg_transaction_value)"
        /%}
    {% /stack %}
    

## 

​

Examples

### 

​

Basic Usage
    
    
    <!-- 2 big values stacked vertically -->
    {% stack  %}
        {% big_value
            data="demo.daily_orders"
            value="sum(total_sales)"
        /%}
        {% big_value
            data="demo.daily_orders"
            value="avg(avg_transaction_value)"
        /%}
    {% /stack %}
    

## 

​

Attributes

​

string

default:"stretch"

How to horizontally align items in this stack**Allowed values:**

  * `left`
  * `center`
  * `right`
  * `stretch`

​

boolean

default:"false"

Display the stack contents as a single card when card mode is enabled

​

string

default:"auto"

Controls page breaks inside this component in PDF exports. `auto` allows content to flow across pages, `avoid` attempts to keep all content together on one page.**Allowed values:**

  * `auto`
  * `avoid`

​

number

Set the width of this component (in percent) relative to the page width

Was this page helpful?

⌘I
