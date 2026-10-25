---
type: external-doc
title: Commentary
source: https://docs.evidence.dev/components/commentary
source_hash: bd4d967ccb
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Commentary

> 原文: <https://docs.evidence.dev/components/commentary>

{% commentary id="overview" /%}
    

## 

​

Examples

### 

​

Page-wide commentary
    
    
    {% commentary id="overview" /%}
    

### 

​

Reporting-period commentary
    
    
    {% commentary id="monthly-review" scope="period" /%}
    

### 

​

Commentary per period and input
    
    
    {% dropdown id="category" data="demo.daily_orders" value_column="category" select_first=true /%}
    
    {% commentary id="category-review" scope=["period", "category"] /%}
    

## 

​

Attributes

​

string

required

Unique identifier for the commentary component

​

string

default:"page"

What the commentary is kept for: `page` (one for the whole page), `period` (one per reporting period; requires `workflow.period` in the frontmatter), an input id, or a list such as `["period", "fund"]` (one per combination of selected values). Inputs must be single-select: dropdown, button_group, input_tabs, toggle or period_selector.

​

string

Placeholder text to show when the commentary is empty

​

string

Additional CSS classes to apply to the component

​

array

Emails of allowed commentary editors

​

string

Title to display above the commentary

​

string

default:"print"

When to hide the edit metadata**Allowed values:**

  * `always`
  * `never`
  * `print`

​

string

default:"normal"

Style of the commentary**Allowed values:**

  * `quote`
  * `normal`

​

number

Set the width of this component (in percent) relative to the page width

Was this page helpful?

⌘I
