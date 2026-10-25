---
type: external-doc
title: Table
source: https://docs.evidence.dev/components/table
source_hash: 873d00eac7
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Table

> 原文: <https://docs.evidence.dev/components/table>

![Basic Usage](https://mintcdn.com/evidence/ImoR9rIJqT9N0Esa/images/components/table/table.png?fit=max&auto=format&n=ImoR9rIJqT9N0Esa&q=85&s=dddd34a5a9c01d56dff69902473cc5df)
    
    
    {% table
        data="demo.daily_orders"
    %}
        {% dimension
            value="category"
        /%}
        {% pivot
            value="date"
            date_grain="year"
        /%}
        {% measure
            value="sum(total_sales)"
            fmt="usd1m"
        /%}
    {% /table %}
    

## 

​

Examples

### 

​

Basic Usage

![Basic Usage](https://mintcdn.com/evidence/ImoR9rIJqT9N0Esa/images/components/table/table.png?fit=max&auto=format&n=ImoR9rIJqT9N0Esa&q=85&s=dddd34a5a9c01d56dff69902473cc5df)
    
    
    {% table
        data="demo.daily_orders"
    %}
        {% dimension
            value="category"
        /%}
        {% pivot
            value="date"
            date_grain="year"
        /%}
        {% measure
            value="sum(total_sales)"
            fmt="usd1m"
        /%}
    {% /table %}
    

### 

​

Date Range Filtering

![Date Range Filtering](https://mintcdn.com/evidence/ImoR9rIJqT9N0Esa/images/components/table/example-1.png?fit=max&auto=format&n=ImoR9rIJqT9N0Esa&q=85&s=ee779e0a3880090d682fd4b75eb9eb67)
    
    
    {% table
      data="demo.daily_orders"
    %}
      {% dimension
        value="category"
      /%}
      {% measure
        value="sum(total_sales)"
        date_range={
          range="last 12 months"
          date="date"
        }
        fmt="usd1m"
      /%}
      {% measure
        value="sum(total_sales)"
        date_range={
          range="last 6 months"
          date="date"
        }
        fmt="usd1m"
      /%}
    {% /table %}
    

### 

​

Pivoting

![Pivoting](https://mintcdn.com/evidence/ImoR9rIJqT9N0Esa/images/components/table/example-2.png?fit=max&auto=format&n=ImoR9rIJqT9N0Esa&q=85&s=1f219c77a80e0c18f54d5e42ec50cc58)
    
    
    {% table
      data="demo.daily_orders"
    %}
      {% dimension
        value="category"
      /%}
      {% pivot
        value="date"
        date_grain="year"
      /%}
      {% measure
        value="sum(total_sales)"
        fmt="usd1m"
      /%}
    {% /table %}
    

### 

​

Prior Year Comparison

![Prior Year Comparison](https://mintcdn.com/evidence/ImoR9rIJqT9N0Esa/images/components/table/example-3.png?fit=max&auto=format&n=ImoR9rIJqT9N0Esa&q=85&s=0825fd87f5c827e26c30c380f884ea56)
    
    
    {% table
      data="demo.daily_orders"
    %}
      {% dimension
        value="category"
      /%}
      {% measure
        value="sum(total_sales)"
        date_range={
          range="last 12 months"
          date="date"
        }
        comparison={
          compare_vs="prior year"
        }
      /%}
    {% /table %}
    

### 

​

Calculated Measures

![Calculated Measures](https://mintcdn.com/evidence/ImoR9rIJqT9N0Esa/images/components/table/example-4.png?fit=max&auto=format&n=ImoR9rIJqT9N0Esa&q=85&s=6673bde534b921748107f582b26301b9)
    
    
    {% table
      data="demo.daily_orders"
    %}
      {% dimension
        value="category"
      /%}
      {% measure
        value="sum(total_sales) / sum(transactions) as avg_price" 
        fmt="usd2"
      /%}
    {% /table %}
    

### 

​

Custom Grouping

![Custom Grouping](https://mintcdn.com/evidence/ImoR9rIJqT9N0Esa/images/components/table/example-5.png?fit=max&auto=format&n=ImoR9rIJqT9N0Esa&q=85&s=789b916a81db789b3cbfa4568b044f34)
    
    
    {% table
      data="demo.daily_orders"
    %}
      {% dimension
        value="case when category in ('Home','Clothing') then 'Home & Clothing' else 'Other' end as group"
      /%}
      {% dimension
        value="category"
      /%}
      {% measure
        value="sum(total_sales)"
        fmt="usd1m"
      /%}
      {% measure
        value="sum(total_sales) / sum(transactions) as avg_price"
        fmt="usd2"
      /%}
    {% /table %}
    

### 

​

Viz: Color Scale

![Viz: Color Scale](https://mintcdn.com/evidence/ImoR9rIJqT9N0Esa/images/components/table/example-6.png?fit=max&auto=format&n=ImoR9rIJqT9N0Esa&q=85&s=1d59618e67b8fdf7fbdb639731ea70c6)
    
    
    {% table
      data="demo.daily_orders"
    %}
      {% dimension
        value="category"
      /%}
      {% measure
        value="sum(total_sales)"
        fmt="usd1m"
        viz="color"
      /%}
    {% /table %}
    

### 

​

Viz: Color Scale with Custom Colors

![Viz: Color Scale with Custom Colors](https://mintcdn.com/evidence/ImoR9rIJqT9N0Esa/images/components/table/example-7.png?fit=max&auto=format&n=ImoR9rIJqT9N0Esa&q=85&s=0752d7a5b0fb11bed626c8ba4ecd150e)
    
    
    {% table
      data="demo.daily_orders"
    %}
      {% dimension
        value="category"
      /%}
      {% measure
        value="sum(total_sales)"
        fmt="usd1m"
        viz="color"
        color_options={
          color_scale=["#c0392b","#f4f4f4","#27ae60"]
        }
      /%}
    {% /table %}
    

### 

​

Viz: Bar

![Viz: Bar](https://mintcdn.com/evidence/ImoR9rIJqT9N0Esa/images/components/table/example-8.png?fit=max&auto=format&n=ImoR9rIJqT9N0Esa&q=85&s=05c083d9c6f78221ce63a561ad1bf474)
    
    
    {% table
      data="demo.daily_orders"
    %}
      {% dimension
        value="category"
      /%}
      {% measure
        value="sum(total_sales)"
        fmt="usd1m"
        viz="bar"
      /%}
    {% /table %}
    

### 

​

Viz: Bar with Custom Colors

![Viz: Bar with Custom Colors](https://mintcdn.com/evidence/ImoR9rIJqT9N0Esa/images/components/table/example-9.png?fit=max&auto=format&n=ImoR9rIJqT9N0Esa&q=85&s=a40cb5d5b9e148dcc84f472564fbcb3d)
    
    
    {% table
      data="demo.daily_orders"
    %}
      {% dimension
        value="category"
      /%}
      {% measure
        value="sum(total_sales)"
        fmt="usd1m"
        viz="bar"
        bar_options={
          bar_color="#2c7d00"
        }
      /%}
      {% measure
        value="sum(transactions)"
        viz="bar"
        bar_options={
          bar_color="#339e9c"
        }
      /%}
    {% /table %}
    

### 

​

Viz: Delta

![Viz: Delta](https://mintcdn.com/evidence/ImoR9rIJqT9N0Esa/images/components/table/example-10.png?fit=max&auto=format&n=ImoR9rIJqT9N0Esa&q=85&s=3e840675ebcf3470d0dab2537d84b299)
    
    
    {% table
      data="demo.daily_orders"
    %}
      {% dimension
        value="category"
      /%}
      {% measure
        value="sum(total_sales)"
        date_range={
          range="last 12 months"
          date="date"
        }
        comparison={
          compare_vs="prior year"
        }
        viz="delta"
      /%}
    {% /table %}
    

### 

​

Viz: Sparkline

![Viz: Sparkline](https://mintcdn.com/evidence/ImoR9rIJqT9N0Esa/images/components/table/example-11.png?fit=max&auto=format&n=ImoR9rIJqT9N0Esa&q=85&s=e6691f52d9dec430dee74c2c67a4b81c)
    
    
    {% table
      data="demo.daily_orders"
    %}
      {% dimension
        value="category"
      /%}
      {% pivot
        value="date"
        date_grain="year"
      /%}
      {% measure
        value="sum(total_sales)"
        fmt="usd1m"
        viz="sparkline"
        sparkline_options={
          x="date"
          type="area"
        }
      /%}
    {% /table %}
    

### 

​

Links

![Links](https://mintcdn.com/evidence/ImoR9rIJqT9N0Esa/images/components/table/example-12.png?fit=max&auto=format&n=ImoR9rIJqT9N0Esa&q=85&s=7ae5adec84a0821be3dc78d966dcb91e)
    
    
    {% table
      data="demo.daily_orders"
    %}
      {% dimension
        value="category"
        link="concat('https://www.google.com/search?q=',category)"
        link_new_tab=true
      /%}
      {% measure
        value="sum(total_sales)"
      /%}
    {% /table %}
    

### 

​

Column Info

![Column Info](https://mintcdn.com/evidence/ImoR9rIJqT9N0Esa/images/components/table/example-13.png?fit=max&auto=format&n=ImoR9rIJqT9N0Esa&q=85&s=a6482cc429ba6531c8d43036286624bb)
    
    
    {% table
      data="demo.daily_orders"
    %}
      {% dimension
        value="category"
      /%}
      {% measure
        value="sum(total_sales)"
        fmt="usd1m"
        info="Includes all product sales"
      /%}
    {% /table %}
    

### 

​

Sorting

![Sorting](https://mintcdn.com/evidence/ImoR9rIJqT9N0Esa/images/components/table/example-14.png?fit=max&auto=format&n=ImoR9rIJqT9N0Esa&q=85&s=c094b6a588c185c627a58c80153c8c05)
    
    
    {% table
      data="demo.daily_orders"
    %}
      {% dimension
        value="category"
      /%}
      {% measure
        value="sum(total_sales)"
        fmt="usd1m"
        viz="color"
        sort="asc"
      /%}
    {% /table %}
    

### 

​

Date Grains: Day of Week

![Date Grains: Day of Week](https://mintcdn.com/evidence/ImoR9rIJqT9N0Esa/images/components/table/example-15.png?fit=max&auto=format&n=ImoR9rIJqT9N0Esa&q=85&s=1c5380a7586e673520754c615378d260)
    
    
    {% table
      data="demo.daily_orders"
    %}
      {% dimension
        value="date"
        date_grain="day of week"
      /%}
      {% measure
        value="sum(total_sales)"
      /%}
    {% /table %}
    

### 

​

Repeat Dimension Values
    
    
    {% table
      data="demo.daily_orders"
      repeat_values=true
    %}
      {% dimension
        value="category"
      /%}
      {% dimension
        value="item"
      /%}
      {% measure
        value="sum(total_sales)"
        fmt="usd1m"
      /%}
    {% /table %}
    

## 

​

Attributes

​

string

​

array

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

string

Custom SQL WHERE condition to apply to the query. For date filters, use date_range instead.

​

string

Custom SQL HAVING condition to apply to the query after GROUP BY

​

number

Maximum number of rows to return from the query. Note: When used with tables, limit will disable subtotals to prevent incomplete subtotal rows.

​

string

Column name(s) with optional direction (e.g. “column_name”, “column_name desc”)

​

string

Custom SQL QUALIFY condition to filter windowed results

​

string

Title to display above the table

​

string

Subtitle to display below the title

​

string

Info text to display in a tooltip next to the title. Can only be used with the title prop.

​

string

URL to link the info text to (can only be used with info)

​

string

Create a custom link title for the info link, placed after the info text (can only be used with info_link)

​

array

Array of dimension column names or SQL expressions. Each item must be a string (e.g., [“column_name”, “count(category) as count”])

​

array

Array of pivot column names or SQL expressions. These will be pivoted in the table output. Each item must be a string (e.g., [“category”, “region”])

​

array

Array of SQL expressions for aggregations. Each item must be a string (e.g., [“sum(transactions)”, “avg(value) as average”])

​

boolean

default:"true"

Whether to include subtotals and totals in the table

​

string

default:"Total"

The label to display in total/subtotal rows and columns. Useful when using non-sum aggregations like avg, min, max, or count.

​

boolean

default:"true"

Whether to display the total row at the bottom. Only applies when subtotals=true. Note: Temporal comparisons may hide totals even when this is true.

​

boolean

default:"true"

Whether to display intermediate subtotal rows. Only applies when subtotals=true. Note: Temporal comparisons may hide subtotals even when this is true.

​

boolean

default:"true"

Whether to display the total column in pivoted tables. Only applies when subtotals=true and pivots are used.

​

boolean

default:"true"

Whether to display intermediate subtotal columns in pivoted tables. Only applies when subtotals=true and pivots are used.

​

boolean

default:"false"

Whether to put measures before pivots in the column hierarchy (e.g., Sales > 2021|2022 instead of 2021|2022 > Sales)

​

number

default:"10"

Number of rows to display per page in pagination (maximum 200)

​

boolean

default:"false"

Whether to display a search box above the table for filtering results

​

boolean

default:"true"

Whether to apply formatting to column titles. When false, titles will be displayed as-is.

​

boolean

default:"true"

Whether to allow column titles to wrap across multiple lines. When false, titles will be on a single line.

​

boolean

default:"false"

Whether to allow table cell content to wrap across multiple lines. When false, cell content will be on a single line.

​

boolean

Whether to apply alternating background colors to table rows for easier reading.

​

boolean

Whether to display borders between table rows. When false, row borders are hidden.

​

string

Column name containing URLs to make each row clickable. When specified, clicking a row will navigate to the URL in that column.

​

boolean

default:"false"

Whether to display the link column in the table. Only applies when no explicit columns are specified and the link prop is used.

​

number

Time in seconds between automatic data refreshes (minimum 30). Overrides the page-level auto-refresh setting for this component.

​

number

default:"0"

Number of left-most columns to freeze when scrolling horizontally. Frozen columns remain visible while the rest of the table scrolls.

​

boolean

default:"false"

Whether to repeat dimension values on every row. When true, dimension values are displayed on every row even when they are the same as the row above.

​

boolean

default:"false"

Whether to enable collapsible groups. When enabled, subtotal rows become clickable to expand/collapse their child rows. Only works when subtotals are enabled and there are dimensions. Note: pagination is disabled when collapsible is enabled.

​

boolean

Whether all groups should start collapsed on initial load. Defaults to true when collapsible=true.

​

string

Where to position subtotal rows relative to their group data rows. “top” places subtotals before the detail rows, “bottom” places them after. Defaults to “top” when collapsible=true, otherwise “bottom”.**Allowed values:**

  * `top`
  * `bottom`

​

string

default:"bottom"

Where to position the grand total row. “top” places it as the first row, “bottom” places it as the last row (default).**Allowed values:**

  * `top`
  * `bottom`

​

string

SQL expression that returns a hex color for each row’s background. Used to conditionally highlight entire rows based on data (e.g., “case when sum(amount) > 10000 then ‘#dcfce7’ else null end”).

​

number

Set the width of this component (in percent) relative to the page width

## 

​

Allowed Children

  * [dimension](</components/dimension>)
  * [measure](</components/measure>)
  * [pivot](</components/pivot>)

Was this page helpful?

⌘I
