---
type: external-doc
title: Dimension
source: https://docs.evidence.dev/components/dimension
source_hash: 01394a69cd
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Dimension

> 原文: <https://docs.evidence.dev/components/dimension>

{% table data="demo.daily_orders" %}
        {% dimension value="category" /%}
        {% dimension value="date" date_grain="month" title="Month" /%}
    {% /table %}
    

## 

​

Examples

### 

​

Table with Dimensions
    
    
    {% table data="demo.daily_orders" %}
        {% dimension value="category" /%}
        {% dimension value="date" date_grain="month" title="Month" /%}
    {% /table %}
    

### 

​

With Image
    
    
    {% table data="demo.daily_orders" %}
        {% dimension value="category" image="image_url" /%}
        {% measure value="sum(total_sales)" /%}
    {% /table %}
    

### 

​

With Link
    
    
    {% table data="demo.daily_orders" %}
        {% dimension value="category" link="concat('https://www.google.com/search?q=', category)" /%}
        {% measure value="sum(total_sales)" /%}
    {% /table %}
    

### 

​

With Logo
    
    
    {% table data="sales" %}
        {% dimension value="vendor" logo="domain" /%}
        {% measure value="sum(amount)" /%}
    {% /table %}
    

### 

​

Conditional Colors
    
    
    {% table data="demo.daily_orders" %}
        {% dimension value="category" conditional_colors="case when sum(total_sales) > 20000000 then '#22c55e' when sum(total_sales) > 10000000 then '#f59e0b' else '#ef4444' end" /%}
        {% measure value="sum(total_sales)" fmt="usd1m" /%}
    {% /table %}
    

## 

​

Attributes

​

string

required

​

string

​

string

**Allowed values:**

  * `left`
  * `center`
  * `right`

​

boolean

Whether to allow content in this dimension column to wrap across multiple lines

​

string

​

string

URL to link the info text to (can only be used with info)

​

string

Create a custom link title for the info link, placed after the info text (can only be used with info_link)

​

boolean

default:"false"

Whether to hide this column from the table display. Hidden columns are still included in queries and can be referenced by other columns.

​

boolean

default:"false"

Whether to render the dimension value as HTML

​

options group

Image display options (used when image prop is set)**Attributes:**

  * height: `number` \- Height of the image in pixels
  * width: `number` \- Width of the image in pixels
  * alt: `string` \- Alt text for the image
  * hide_label: `boolean` \- Whether to hide the text label and show only the image

​

string

Column name or SQL expression containing the image URL for this dimension

​

string

Column name or SQL expression containing the domain to look up a company logo for this dimension

​

options group

Logo display options (used when logo prop is set)**Example:**
    
    
     logo_options={
      size = "sm"
      grayscale = true
      hide_label = true
    }
    

**Attributes:**

  * size: `string` \- Logo size: sm, base, lg, xl
    * **Allowed values:**
      * `sm`
      * `base`
      * `lg`
      * `xl`
  * grayscale: `boolean` \- Display logo in grayscale
  * hide_label: `boolean` \- Whether to hide the text label and show only the logo

​

string

Column name or SQL expression containing the URL for linking this dimension

​

string

Static text to use as the link label instead of the cell content

​

boolean

default:"false"

Whether to open the link in a new tab

​

string

Bucket dates into a grain. Pass the raw date column as `x` and the chart truncates and groups for you. Temporal grains (`day`, `week`, `month`, `quarter`, `year`, `hour`) preserve the year — use for time-series. Seasonality grains (`day of week`, `day of month`, `day of year`, `week of year`, `month of year`, `quarter of year`) collapse across years — use for cyclical patterns like “which month sells most regardless of year”.**Allowed values:**

  * `day`
  * `week`
  * `month`
  * `quarter`
  * `year`
  * `hour`
  * `day of week`
  * `day of month`
  * `day of year`
  * `week of year`
  * `month of year`
  * `quarter of year`

​

string

Format code for displaying dimension values (e.g., “yyyy” for years, “mmm” for months)

​

string

Sort direction for this dimension column. When specified, the table will be sorted by this column.**Allowed values:**

  * `asc`
  * `desc`

​

string

Group name for this column. Columns with matching group names will be visually grouped under a shared header.

​

boolean

Whether to repeat this dimension value on every row. When true, the value is displayed on every row even when it is the same as the row above. Overrides the table-level repeat_values setting for this dimension.

​

string

SQL expression that returns color values for each row. Used to conditionally color dimension cells based on data (e.g., “case when count(*) > 0 then ‘#22c55e’ else null end”).

## 

​

Allowed Parents

  * [table](</components/table>)

Was this page helpful?

⌘I
