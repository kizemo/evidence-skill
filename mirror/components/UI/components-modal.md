---
type: external-doc
title: Modal
source: https://docs.evidence.dev/components/modal
source_hash: bcc3296e93
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Modal

> 原文: <https://docs.evidence.dev/components/modal>

![Basic Usage](https://mintcdn.com/evidence/Z2CIIsynbphd2HX3/images/components/modal/modal.png?fit=max&auto=format&n=Z2CIIsynbphd2HX3&q=85&s=14aace5c4c9ef078cd2e8ee1bd1b3cfe)
    
    
    {% modal
        title="Detailed Trend"
    %}
    Below is a detailed sales trend
    
        {% line_chart
            data="demo.daily_orders"
            x="date"
            y="sum(total_sales)"
            date_grain="quarter"
        /%}
    {% /modal %}
    

## 

​

Examples

### 

​

Basic Usage

![Basic Usage](https://mintcdn.com/evidence/Z2CIIsynbphd2HX3/images/components/modal/modal.png?fit=max&auto=format&n=Z2CIIsynbphd2HX3&q=85&s=14aace5c4c9ef078cd2e8ee1bd1b3cfe)
    
    
    {% modal
        title="Detailed Trend"
    %}
    Below is a detailed sales trend
    
        {% line_chart
            data="demo.daily_orders"
            x="date"
            y="sum(total_sales)"
            date_grain="quarter"
        /%}
    {% /modal %}
    

## 

​

Attributes

​

string

required

Title displayed in the modal header

​

string

Text displayed on the button that opens the modal. Defaults to the title if not provided.

​

string

default:"default"

Button style variant**Allowed values:**

  * `default`
  * `primary`
  * `destructive`
  * `secondary`
  * `ghost`
  * `link`

​

string

Icon to display in the button and modal header**Allowed values:**

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

Show only the icon in the button with a tooltip containing the title

Was this page helpful?

⌘I
