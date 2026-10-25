---
type: external-doc
title: Toggle
source: https://docs.evidence.dev/components/toggle
source_hash: 3681467303
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Toggle

> 原文: <https://docs.evidence.dev/components/toggle>

![Basic Usage](https://mintcdn.com/evidence/lyRfl9dhvfuFUnx3/images/components/toggle/toggle.png?fit=max&auto=format&n=lyRfl9dhvfuFUnx3&q=85&s=aabf0876e9a4a0c433b1f5214ff3c254)
    
    
    {% toggle
        id="show_legend"
        label="Show Legend"
    /%}
    

## 

​

Examples

### 

​

Basic Usage

![Basic Usage](https://mintcdn.com/evidence/lyRfl9dhvfuFUnx3/images/components/toggle/toggle.png?fit=max&auto=format&n=lyRfl9dhvfuFUnx3&q=85&s=aabf0876e9a4a0c433b1f5214ff3c254)
    
    
    {% toggle
        id="show_legend"
        label="Show Legend"
    /%}
    

### 

​

Using `where`

![Using where](https://mintcdn.com/evidence/ImoR9rIJqT9N0Esa/images/components/toggle/example-1.png?fit=max&auto=format&n=ImoR9rIJqT9N0Esa&q=85&s=b174f1c0e655d6ed78f2da1702d9b314)
    
    
    {% toggle
        id="active_only"
        label="Active Only"
    /%}
    
    {% table
        data="demo.daily_orders"
        where="{{active_only}} = false or total_sales > 1000"
    /%}
    

### 

​

Using Inline SQL

![Using Inline SQL](https://mintcdn.com/evidence/ImoR9rIJqT9N0Esa/images/components/toggle/example-2.png?fit=max&auto=format&n=ImoR9rIJqT9N0Esa&q=85&s=3c4eafc986159f9a7045bb0c6ee8d881)
    
    
    {% toggle
        id="active_only"
        label="Active Only"
    /%}
    
    ```sql filtered_orders
    select * from demo.daily_orders
    where {{active_only}} = false or total_sales > 1000
    ```
    
    {% table data="filtered_orders" /%}
    

## 

​

Attributes

​

string

required

The id of the toggle to be used in a `filters` prop

​

string

Text displayed next to the toggle inside the box. Defaults to the id if not provided.

​

string

Information tooltip text that appears after the label

​

boolean

default:"false"

Invert the boolean value output. When true, checked = false and unchecked = true. Useful when toggle label semantics are opposite to the filter logic (e.g., a “Show inactive” toggle that filters for is_active = false when checked).

​

boolean

default:"false"

Initial state of the toggle

## 

​

Using the Filter Variable

Reference this filter using `{{filter_id}}`. The value returned depends on where you use it.

Context| Default Property| No Selection| Result  
---|---|---|---  
Inline SQL query| `.value`| | `true`  
`where` attribute| `.value`| | `true`  
Text / Markdown| `.value`| | `true`  
  
### 

​

Available Properties

You can also access specific properties using `{{filter_id.property}}`:

#### 

​

.value

Returns the boolean value of the toggle.
    
    
    {% toggle id="active_filter" /%}
    
    ```sql active_users
    select * from users
    where is_active = {{active_filter}}
    ```
    

**Example value:** `true`

Was this page helpful?

⌘I
