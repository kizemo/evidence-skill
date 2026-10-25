---
type: external-doc
title: Accordion Item
source: https://docs.evidence.dev/components/accordion_item
source_hash: 32c189f50e
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Accordion Item

> 原文: <https://docs.evidence.dev/components/accordion_item>

{% accordion %}
        {% accordion_item title="Section 1" open=true %}
            Content for section 1.
        {% /accordion_item %}
        {% accordion_item title="Section 2" %}
            Content for section 2.
        {% /accordion_item %}
        {% accordion_item title="Section 3" icon="settings" %}
            Content for section 3 with an icon.
        {% /accordion_item %}
    {% /accordion %}
    

## 

​

Examples

### 

​

Accordion with Items
    
    
    {% accordion %}
        {% accordion_item title="Section 1" open=true %}
            Content for section 1.
        {% /accordion_item %}
        {% accordion_item title="Section 2" %}
            Content for section 2.
        {% /accordion_item %}
        {% accordion_item title="Section 3" icon="settings" %}
            Content for section 3 with an icon.
        {% /accordion_item %}
    {% /accordion %}
    

## 

​

Attributes

​

string

Title displayed in the accordion item header. For rich titles (e.g. containing another component), nest an `accordion_title` tag instead. When both are provided, the nested `accordion_title` takes precedence and this attribute is ignored.

​

string

Icon to display in the accordion item header**Allowed values:**

  * `trending-up`
  * `trending-down`
  * `clock`
  * `calendar`
  * `check`
  * `x`
  * `info`
  * `alert-circle`
  * `help-circle`
  * `eye`
  * `eye-off`
  * `user`
  * `users`
  * `settings`
  * `cog`
  * `plus`
  * `minus`
  * `up`
  * `down`
  * `right`
  * `left`
  * `star`
  * `heart`
  * `search`
  * `file`
  * `file-text`
  * `home`
  * `mail`
  * `filter`
  * `share`
  * `bell`
  * `trash`
  * `credit-card`
  * `globe`
  * `key`
  * `croissant`
  * `map`
  * `rotate`
  * `rewind`
  * `bank`
  * `receipt`
  * `activity`
  * `chart-column`
  * `chart-pie`
  * `chart-no-axes-combined`
  * `goal`
  * `rocket`
  * `trophy`
  * `apple`
  * `cookie`
  * `donut`
  * `beef`
  * `cake`
  * `soup`
  * `utensils`
  * `milk`
  * `nut`
  * `pyramid`
  * `triangle`
  * `arrow-down`
  * `arrow-left`
  * `arrow-right`
  * `arrow-up`
  * `chevron-down`
  * `chevron-left`
  * `chevron-right`
  * `chevron-up`
  * `chevrons-down`
  * `chevrons-left`
  * `chevrons-right`
  * `chevrons-up`
  * `menu`
  * `external-link`
  * `check-circle`
  * `x-circle`
  * `edit`
  * `trash-2`
  * `copy`
  * `save`
  * `download`
  * `upload`
  * `send`
  * `refresh`
  * `redo`
  * `undo`
  * `folder`
  * `folder-open`
  * `image`
  * `file-image`
  * `user-plus`
  * `user-minus`
  * `user-check`
  * `lock`
  * `unlock`
  * `log-in`
  * `log-out`
  * `message-square`
  * `message-circle`
  * `phone`
  * `phone-call`
  * `bell-off`
  * `video`
  * `video-off`
  * `play`
  * `pause`
  * `stop`
  * `skip-back`
  * `skip-forward`
  * `volume`
  * `volume-1`
  * `volume-2`
  * `volume-off`
  * `volume-x`
  * `bookmark`
  * `tag`
  * `link`
  * `unlink`
  * `share-2`
  * `alert-triangle`
  * `loader`
  * `more-vertical`
  * `more-horizontal`
  * `grid`
  * `list`
  * `maximize`
  * `minimize`
  * `zoom-in`
  * `zoom-out`
  * `thumbs-up`
  * `thumbs-down`
  * `shopping-cart`
  * `dollar-sign`
  * `camera`
  * `printer`
  * `monitor`
  * `smartphone`
  * `laptop`
  * `calculator`
  * `cloud-sun-rain`
  * `sun-snow`
  * `thermometer-sun`
  * `thermometer-snowflake`
  * `cloudy`
  * `cloud-rain-wind`
  * `cloud-rain`
  * `wind`
  * `sun`
  * `cloud-snow`
  * `thermometer`
  * `cloud-drizzle`
  * `cloud-sun`
  * `cloud`
  * `cloud-lightning`
  * `snowflake`
  * `flame`
  * `atom`
  * `fuel`
  * `magnet`
  * `factory`
  * `tree-deciduous`
  * `waypoints`
  * `plug`
  * `dam`
  * `battery`

​

boolean

default:"false"

Whether the accordion item is initially open

## 

​

Allowed Parents

  * [accordion](</components/accordion>)

Was this page helpful?

⌘I
