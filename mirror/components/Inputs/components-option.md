---
type: external-doc
title: Option
source: https://docs.evidence.dev/components/option
source_hash: 8fc59bcd85
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Option

> 原文: <https://docs.evidence.dev/components/option>

{% dropdown id="category" label="Category" %}
        {% option value="electronics" label="Electronics" /%}
        {% option value="clothing" label="Clothing" /%}
        {% option value="home" label="Home & Garden" /%}
    {% /dropdown %}
    

## 

​

Examples

### 

​

In a Dropdown
    
    
    {% dropdown id="category" label="Category" %}
        {% option value="electronics" label="Electronics" /%}
        {% option value="clothing" label="Clothing" /%}
        {% option value="home" label="Home & Garden" /%}
    {% /dropdown %}
    

### 

​

In a Button Group
    
    
    {% button_group id="metric" label="Metric" %}
        {% option value="revenue" label="Revenue" fmt="usd" /%}
        {% option value="quantity" label="Quantity" fmt="num0" /%}
        {% option value="margin" label="Margin" fmt="pct1" /%}
    {% /button_group %}
    

## 

​

Attributes

​

string | number

required

Value of the option

​

string

Label text displayed in the option

​

string

Format string to associate with this option (e.g., “usd” or “pct1”)

## 

​

Allowed Parents

  * [dropdown](</components/dropdown>)
  * [button_group](</components/button_group>)
  * [input_tabs](</components/input_tabs>)

Was this page helpful?

⌘I
