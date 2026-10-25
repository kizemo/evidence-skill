---
type: external-doc
title: Print Group
source: https://docs.evidence.dev/components/print_group
source_hash: f76e5655c8
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Print Group

> 原文: <https://docs.evidence.dev/components/print_group>

## 

​

Examples

### 

​

Basic Print Group
    
    
    {% print_group %}
    This content will stay together when printed.
    {% /print_group %}
    

### 

​

Hidden Print Group
    
    
    {% print_group hide=true %}
    This content will be hidden when printed.
    {% /print_group %}
    

### 

​

Print-Only Group
    
    
    {% print_group print_only=true %}
    This content only appears when printed.
    {% /print_group %}
    

## 

​

Attributes

​

boolean

default:"false"

Whether to hide this group when printing. To hide content without grouping it, use `hidden_in_print`

​

boolean

default:"false"

Show this group only when printing or generating PDFs; hidden on screen. To do this without grouping, use `only_in_print`

Was this page helpful?

⌘I
