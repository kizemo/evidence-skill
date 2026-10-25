---
type: external-doc
title: Dropdown
source: https://docs.evidence.dev/components/dropdown
source_hash: dcfa686bdb
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Dropdown

> 原文: <https://docs.evidence.dev/components/dropdown>

![Using filters](https://mintcdn.com/evidence/Z2CIIsynbphd2HX3/images/components/dropdown/dropdown.png?fit=max&auto=format&n=Z2CIIsynbphd2HX3&q=85&s=6e949de317c957bf8cbbf12bb6463b7b)
    
    
    {% dropdown
        id="category_filter"
        data="demo.daily_orders"
        value_column="category"
    /%}
    
    {% bar_chart
        data="demo.daily_orders"
        x="date"
        y="sum(total_sales)"
        filters=["category_filter"]
        date_grain="month"
    /%}
    

## 

​

Examples

### 

​

Using `filters`

![Using filters](https://mintcdn.com/evidence/Z2CIIsynbphd2HX3/images/components/dropdown/dropdown.png?fit=max&auto=format&n=Z2CIIsynbphd2HX3&q=85&s=6e949de317c957bf8cbbf12bb6463b7b)
    
    
    {% dropdown
        id="category_filter"
        data="demo.daily_orders"
        value_column="category"
    /%}
    
    {% bar_chart
        data="demo.daily_orders"
        x="date"
        y="sum(total_sales)"
        filters=["category_filter"]
        date_grain="month"
    /%}
    

### 

​

Using `where`

![Using where](https://mintcdn.com/evidence/qxp1vq5RbQ6sDlcg/images/components/dropdown/example-1.png?fit=max&auto=format&n=qxp1vq5RbQ6sDlcg&q=85&s=657d862e42daa1a5a800414ea489b0cd)
    
    
    {% dropdown
        id="category_filter"
        data="demo.daily_orders"
        value_column="category"
    /%}
    
    {% bar_chart
        data="demo.daily_orders"
        x="date"
        y="sum(total_sales)"
        where="category = {{category_filter}}"
        date_grain="month"
    /%}
    

### 

​

Using Inline SQL

![Using Inline SQL](https://mintcdn.com/evidence/Z2CIIsynbphd2HX3/images/components/dropdown/example-2.png?fit=max&auto=format&n=Z2CIIsynbphd2HX3&q=85&s=d2100de9b6bf44dc33f38396657f5dc7)
    
    
    {% dropdown
        id="category_filter"
        data="demo.daily_orders"
        value_column="category"
    /%}
    
    ```sql filtered_orders
    select * from demo.daily_orders
    where category = {{category_filter}}
    ```
    
    {% table data="filtered_orders" /%}
    

### 

​

Using `date_range`

![Using date_range](https://mintcdn.com/evidence/Z2CIIsynbphd2HX3/images/components/dropdown/example-3.png?fit=max&auto=format&n=Z2CIIsynbphd2HX3&q=85&s=ef8a488c5c33eeb73ce152e9bd84b67b)
    
    
    {% dropdown
        id="category_filter"
        data="demo.daily_orders"
        value_column="category"
        date_range={
            date="date"
            range="last 12 months"
        }
    /%}
    

### 

​

Cascading Dropdowns (Linked Filters)

![Cascading Dropdowns \(Linked Filters\)](https://mintcdn.com/evidence/Z2CIIsynbphd2HX3/images/components/dropdown/example-4.png?fit=max&auto=format&n=Z2CIIsynbphd2HX3&q=85&s=c8a5ccafc5101dcf4344a1bb58765a35)
    
    
    {% dropdown
        id="category"
        data="demo.order_details"
        value_column="category"
        title="Category"
    /%}
    
    {% dropdown
        id="item"
        data="demo.order_details"
        value_column="item_name"
        title="Item"
        filters=["category"]
    /%}
    
    {% table
        data="demo.order_details"
        filters=["category", "item"]
    /%}
    

## 

​

Attributes

​

string

required

The id of the dropdown to be used in a `filters` prop

​

string

Name of the table to query

​

array

Array of filter IDs to apply when querying for dropdown options. Use this to create cascading/linked dropdowns where selecting a value in one dropdown narrows the available options in this dropdown.

​

string

Column name to use as the value for each option, and the column to filter by when this dropdown’s `id` is used in the `filters` prop of a chart

​

string

Column name to use as the label for each option

​

array

List of options to display in the dropdown

​

string

Text displayed above the dropdown

​

string

Information tooltip text

​

string

URL to link the info text to (can only be used with info)

​

string

Create a custom link title for the info link, placed after the info text (can only be used with info_link)

​

string

Icon to display in the dropdown trigger**Allowed values:**

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

string | number | array

Initial selected value(s)

​

boolean

default:"false"

Automatically select the first option when the component loads

​

number

For a multi-select dropdown, pre-selects the first N options (after `order` is applied) when the component loads. Generalizes `select_first`. Does not limit the option list and has no effect when `multiple=false`.

​

string

Placeholder text displayed when no value is selected

​

boolean

default:"true"

Includes a search input within the dropdown menu

​

boolean

default:"false"

Allows multiple selections

​

boolean

default:"true"

Includes a clear button to unselect the selected value(s)

​

string

Column name(s) with optional direction (e.g. “column_name”, “column_name desc”)

​

string

Custom SQL WHERE condition to apply to the query. For date filters, use date_range instead.

​

options group

Filter data to a time period. `date_range` is an OBJECT with `range` (the period) and optionally `date` (which column to filter on when the table has more than one). Shape: `date_range={ range="last 12 months" date="order_date" }`. `range` accepts predefined values (`last 7 days`, `month to date`), dynamic patterns (`Last 90 days`), custom windows (`2020-01-01 to 2023-03-01`), or partial ranges (`from 2020-01-01`, `until 2023-03-01`). Pass a plain string for `range` — the whole object is NOT a string.**Example:**
    
    
     date_range={
      range = "today"
      date = "string"
    }
    

**Attributes:**

  * range: `string` \- Time period to filter. Use presets like ‘last 7 days’, dynamic patterns like ‘Last 90 days’, custom ranges like ‘2020-01-01 to 2023-03-01’, or partial ranges like ‘from 2020-01-01’.
    * **Allowed values:**
      * `today`
      * `yesterday`
      * `last 7 days`
      * `last 30 days`
      * `last 3 months`
      * `last 6 months`
      * `last 12 months`
      * `previous week`
      * `previous month`
      * `previous quarter`
      * `previous year`
      * `this week`
      * `this month`
      * `this quarter`
      * `this year`
      * `next week`
      * `next month`
      * `next quarter`
      * `next year`
      * `week to date`
      * `month to date`
      * `quarter to date`
      * `year to date`
      * `all time`
  * date: `string` \- Date column to filter on. Required when the data has multiple date columns.

​

number

Set the width of this component (in percent) relative to the page width

## 

​

Using the Filter Variable

Reference this filter using `{{filter_id}}`. The value returned depends on where you use it. The examples below show values for three scenarios:

  * **No selection**
  * **Single select:** “Electronics” selected
  * **Multi select:** “Sports” and “Home” selected (when `multiple=true`)

Context| Default Property| No Selection| Single Select| Multi Select  
---|---|---|---|---  
Inline SQL query| `.selected`| `''`| `'Electronics'`| `('Sports', 'Home')`  
`where` attribute| `.selected`| `''`| `'Electronics'`| `('Sports', 'Home')`  
Text / Markdown| `.literal`| | `Electronics`| `Sports, Home`  
  
### 

​

Available Properties

You can also access specific properties using `{{filter_id.property}}`:

#### 

​

.filter

Returns a complete SQL filter expression ready to use in WHERE clauses. Returns `true` when no value is selected.
    
    
    {% dropdown id="category_filter" data="products" value_column="category" /%}
    
    ```sql filtered_products
    select * from products
    where {{category_filter.filter}}
    ```
    

No Selection| Single Select| Multi Select  
---|---|---  
`true`| `category = 'Electronics'`| `category IN ('Sports', 'Home')`  
  
#### 

​

.selected

Returns the selected value(s) wrapped in quotes, suitable for SQL comparisons. Returns an empty string when no value is selected.
    
    
    {% dropdown id="category_filter" data="products" value_column="category" /%}
    
    ```sql products_by_category
    select * from products
    where category = {{category_filter.selected}}
    ```
    

No Selection| Single Select| Multi Select  
---|---|---  
`''`| `'Electronics'`| `('Sports', 'Home')`  
  
#### 

​

.literal

Returns the raw unescaped selected value(s), useful for display in text or dynamic column selection.
    
    
    {% dropdown id="sort_column" data="products" value_column="column_name" /%}
    
    ```sql dynamic_sort
    select * from products
    order by {{sort_column.literal}}
    ```
    

No Selection| Single Select| Multi Select  
---|---|---  
“| `Electronics`| `Sports, Home`  
  
#### 

​

.label

Returns the display label for the selected option(s). Falls back to the value if no label is defined.
    
    
    {% dropdown id="category_filter" %}
        {% option value="Electronics" label="Electronics" /%}
        {% option value="Sports" label="Sports" /%}
        {% option value="Home" label="Home" /%}
    {% /dropdown %}
    
    Selected: {{category_filter.label}}
    

No Selection| Single Select| Multi Select  
---|---|---  
“| `Electronics`| `Sports, Home`  
  
#### 

​

.fmt

Returns the format string associated with the selected option. For multiple selections, returns the first format.
    
    
    {% dropdown id="metric_selector" %}
        {% option value="revenue" label="Revenue" fmt="usd" /%}
        {% option value="growth_rate" label="Growth Rate" fmt="pct1" /%}
    {% /dropdown %}
    
    {% big_value data={metrics} value=value fmt={{metric_selector.fmt}} /%}
    

No Selection| Single Select| Multi Select  
---|---|---  
“| `usd`| `usd`  
  
## 

​

Allowed Children

  * [option](</components/option>)

Was this page helpful?

⌘I
