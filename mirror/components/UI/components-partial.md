---
type: external-doc
title: Partial
source: https://docs.evidence.dev/components/partial
source_hash: e5d8be5d16
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Partial

> 原文: <https://docs.evidence.dev/components/partial>

{% partial file="my_partial" /%}
    

## 

​

Examples

### 

​

Basic Usage
    
    
    {% partial file="my_partial" /%}
    

### 

​

Passing Variables from Frontmatter
    
    
    {% partial file="my_partial" variables={
    	my_category=$category
    } /%}
    

### 

​

Passing Hardcoded Values
    
    
    {% partial file="my_partial" variables={
    	my_category="Home"
    	sales_threshold=10000
    } /%}
    

## 

​

Attributes

​

string

required

Path to the partial to render. Leading slash means “from the project root” (recommended, e.g. `/partials/header`); without slash, the path resolves relative to the referencing page’s directory.

​

options group

Variables to pass to the partial, must be variables from frontmatter or hardcoded values.

Was this page helpful?

⌘I
