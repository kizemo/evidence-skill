---
type: external-doc
title: Accordion Title
source: https://docs.evidence.dev/components/accordion_title
source_hash: f2d7b78003
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Accordion Title

> 原文: <https://docs.evidence.dev/components/accordion_title>

## 

​

Examples

### 

​

Accordion item with a big_value in the title
    
    
    {% accordion %}
        {% accordion_item %}
            {% accordion_title padding_top=8 padding_bottom=8 %}
                {% big_value data="orders" value="sum(sales)" fmt="usd" /%}
            {% /accordion_title %}
            Body content for this section.
        {% /accordion_item %}
    {% /accordion %}
    

## 

​

Attributes

​

number

default:"16"

Top padding of the accordion trigger, in pixels. Lower this when the slotted content already provides its own top padding.

​

number

default:"16"

Bottom padding of the accordion trigger, in pixels. Lower this when the slotted content already provides its own bottom padding.

## 

​

Allowed Parents

  * [accordion_item](</components/accordion_item>)

Was this page helpful?

⌘I
