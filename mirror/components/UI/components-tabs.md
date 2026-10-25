---
type: external-doc
title: Tabs
source: https://docs.evidence.dev/components/tabs
source_hash: d9d0f6b1ec
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Tabs

> 原文: <https://docs.evidence.dev/components/tabs>

![Basic Usage](https://mintcdn.com/evidence/ImoR9rIJqT9N0Esa/images/components/tabs/tabs.png?fit=max&auto=format&n=ImoR9rIJqT9N0Esa&q=85&s=7fc4b777e196466e49d4e77646217667)
    
    
    {% tabs %}
    	{% tab title="Tab 1" icon="trending-up" %}
    		Content for tab 1
    	{% /tab %}
    	{% tab title="Tab 2" icon="alert-circle" %}
    		Content for tab 2
    	{% /tab %}
    {% /tabs %}
    

## 

​

Examples

### 

​

Basic Usage

![Basic Usage](https://mintcdn.com/evidence/ImoR9rIJqT9N0Esa/images/components/tabs/tabs.png?fit=max&auto=format&n=ImoR9rIJqT9N0Esa&q=85&s=7fc4b777e196466e49d4e77646217667)
    
    
    {% tabs %}
    	{% tab title="Tab 1" icon="trending-up" %}
    		Content for tab 1
    	{% /tab %}
    	{% tab title="Tab 2" icon="alert-circle" %}
    		Content for tab 2
    	{% /tab %}
    {% /tabs %}
    

### 

​

Well Variant

![Well Variant](https://mintcdn.com/evidence/ImoR9rIJqT9N0Esa/images/components/tabs/example-1.png?fit=max&auto=format&n=ImoR9rIJqT9N0Esa&q=85&s=c3172bf113b4bff2c440c566ecd6f62f)
    
    
    {% tabs variant="well" %}
    	{% tab title="Tab 1" icon="trending-up" %}
    		Content for tab 1
    	{% /tab %}
    	{% tab title="Tab 2" icon="alert-circle" %}
    		Content for tab 2
    	{% /tab %}
    {% /tabs %}
    

## 

​

Attributes

​

string

default:"default"

Visual style variant of the tabs**Allowed values:**

  * `default`
  * `well`

​

boolean

default:"false"

Whether the tabs should take the full width of their container

​

string

Custom color for active tab text and underline. Uses CSS color values.

​

string

default:"left"

Horizontal alignment of tabs. Note: align right only affects the default variant.**Allowed values:**

  * `left`
  * `right`

​

string

default:"auto"

Controls page breaks inside this component in PDF exports. `auto` allows content to flow across pages, `avoid` attempts to keep all content together on one page.**Allowed values:**

  * `auto`
  * `avoid`

​

number

Set the width of this component (in percent) relative to the page width

## 

​

Allowed Children

  * [tab](</components/tab>)

Was this page helpful?

⌘I
