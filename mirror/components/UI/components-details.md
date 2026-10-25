---
type: external-doc
title: Details
source: https://docs.evidence.dev/components/details
source_hash: 34af7c077a
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Details

> 原文: <https://docs.evidence.dev/components/details>

![Basic Usage](https://mintcdn.com/evidence/Z2CIIsynbphd2HX3/images/components/details/details.png?fit=max&auto=format&n=Z2CIIsynbphd2HX3&q=85&s=74013934360497998f9d4db10de9736b)
    
    
    {% details
        title="Metric Definitions"
    %}
    
    **Sales:** includes sales of all core products in all regions
    
    **Sales Growth:** YoY growth in sales
    
    {% /details %}
    

## 

​

Examples

### 

​

Basic Usage

![Basic Usage](https://mintcdn.com/evidence/Z2CIIsynbphd2HX3/images/components/details/details.png?fit=max&auto=format&n=Z2CIIsynbphd2HX3&q=85&s=74013934360497998f9d4db10de9736b)
    
    
    {% details
        title="Metric Definitions"
    %}
    
    **Sales:** includes sales of all core products in all regions
    
    **Sales Growth:** YoY growth in sales
    
    {% /details %}
    

### 

​

Open on Desktop, Collapsed on Mobile
    
    
    {% details
        title="Metric Definitions"
        open=true
        open_mobile=false
    %}
    
    **Sales:** includes sales of all core products in all regions
    
    **Sales Growth:** YoY growth in sales
    
    {% /details %}
    

## 

​

Attributes

​

string

required

Title of the details section

​

boolean

default:"false"

Whether the details section starts open

​

boolean

Whether the details section starts open on mobile screens. Defaults to the value of `open`

​

number

Set the width of this component (in percent) relative to the page width

Was this page helpful?

⌘I
