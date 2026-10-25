---
type: external-doc
title: Combo Chart
source: https://docs.evidence.dev/components/combo_chart
source_hash: fb6169ea17
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Combo Chart

> 原文: <https://docs.evidence.dev/components/combo_chart>

![Basic Usage](https://mintcdn.com/evidence/oSDKudj1ZJ3RGuev/images/components/combo_chart/combo_chart.png?fit=max&auto=format&n=oSDKudj1ZJ3RGuev&q=85&s=384e377a0a1cc468be16ce4fc41448b9)
    
    
    {% combo_chart
        data="demo.daily_orders"
        x="date"
        date_grain="month"
    %}
        {% line
            y="sum(total_sales)"
        /%}
        {% bar
            y="sum(transactions)"
            axis="y2"
        /%}
    {% /combo_chart %}
    

## 

​

Examples

### 

​

Basic Usage

![Basic Usage](https://mintcdn.com/evidence/oSDKudj1ZJ3RGuev/images/components/combo_chart/combo_chart.png?fit=max&auto=format&n=oSDKudj1ZJ3RGuev&q=85&s=384e377a0a1cc468be16ce4fc41448b9)
    
    
    {% combo_chart
        data="demo.daily_orders"
        x="date"
        date_grain="month"
    %}
        {% line
            y="sum(total_sales)"
        /%}
        {% bar
            y="sum(transactions)"
            axis="y2"
        /%}
    {% /combo_chart %}
    

### 

​

Combo Chart with Area and Line

![Combo Chart with Area and Line](https://mintcdn.com/evidence/oSDKudj1ZJ3RGuev/images/components/combo_chart/example-1.png?fit=max&auto=format&n=oSDKudj1ZJ3RGuev&q=85&s=124a287d7d01e790eafbdfcdfe2b01d5)
    
    
    {% combo_chart
        data="demo.daily_orders"
        x="date"
        date_grain="month"
    %}
        {% area
            y="sum(total_sales)"
        /%}
        {% line
            y="avg(avg_transaction_value)"
            axis="y2"
        /%}
    {% /combo_chart %}
    

### 

​

Combo Chart with Formatting

![Combo Chart with Formatting](https://mintcdn.com/evidence/oSDKudj1ZJ3RGuev/images/components/combo_chart/example-2.png?fit=max&auto=format&n=oSDKudj1ZJ3RGuev&q=85&s=52b68f36b6fcfe09ef8df12e3175be8e)
    
    
    {% combo_chart
        data="demo.daily_orders"
        x="date"
        y_fmt="usd"
        y2_fmt="num0"
        title="Sales and Transactions Overview"
        subtitle="Combined view of sales and transaction volume"
        date_grain="month"
    %}
        {% line
            y="sum(total_sales)"
        /%}
        {% bar
            y="sum(transactions)"
            axis="y2"
        /%}
    {% /combo_chart %}
    

### 

​

Raw ECharts overrides

![Raw ECharts overrides](https://mintcdn.com/evidence/oSDKudj1ZJ3RGuev/images/components/combo_chart/example-3.png?fit=max&auto=format&n=oSDKudj1ZJ3RGuev&q=85&s=fad7734919ba555ca004e8674251d8ff)
    
    
    {% combo_chart
        data="demo.daily_orders"
        x="date"
        date_grain="month"
        echarts_options={
            dataZoom=[{ type="slider" }]
            tooltip={ position="top" }
        }
        echarts_series_options={
            itemStyle={ borderRadius=4 }
        }
    %}
        {% bar y="sum(total_sales)" /%}
    {% /combo_chart %}
    

## 

​

Attributes

​

string

Name of the table to query. Required unless every child series uses `metric="..."` — metric children resolve their own base from the metric view.

​

array

IDs of filters to apply to the query

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

default:"connect"

How to handle missing data points. “connect” auto-connects points (default), “gaps” shows visual breaks, “zero” fills with zeros.**Allowed values:**

  * `connect`
  * `gaps`
  * `zero`

​

string

Column name for x-axis. Required unless every child series uses `metric="..."` — metric children fall back to the metric view’s time column.

​

string

Format for x values and axis labels. See [Value Formatting](</core-concepts/value-formatting>) for available formats.

​

string

Format for y values and axis labels. See [Value Formatting](</core-concepts/value-formatting>) for available formats.

​

string

Format for y2 values and axis labels. See [Value Formatting](</core-concepts/value-formatting>) for available formats.

​

string

Column name for series

​

string

Column name for individual point labels displayed at the top of the tooltip

​

string

Title to display above the component

​

string

Subtitle to display below the title

​

string

Information tooltip text (can only be used with title). Displays an info icon next to the title.

​

string

URL to link the info text to (can only be used with info)

​

string

Create a custom link title for the info link, placed after the info text (can only be used with info_link)

​

number

Time in seconds between automatic data refreshes (minimum 30). Overrides the page-level auto-refresh setting for this component.

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

number

Set the width of this component (in percent) relative to the page width

​

number

Set a fixed height for the chart in pixels

​

string

Link this chart to others sharing the same id, syncing their tooltips, axis-pointer, and zoom

​

string

Sort order for x-axis categories. Options: `asc` (alphabetical), `desc` (reverse alphabetical), `data` (preserve query order), or an array for custom order like `["A", "B", "C"]`**Allowed values:**

  * `asc`
  * `desc`
  * `data`

​

options group

Configure the y-axis**Example:**
    
    
     y_axis_options={
      title = "string"
      title_position = "top"
      ticks = true
      baseline = true
      labels = true
      gridlines = true
      min = 0
      max = 0
      fit_to_data = true
      interval = 0
    }
    

**Attributes:**

  * title: `string`
  * title_position: `string` \- Position of the axis title. “top” places it horizontally at the top, “side” places it vertically along the axis. Defaults to “side” for 100% stacked charts, “top” otherwise.
    * **Allowed values:**
      * `top`
      * `side`
  * ticks: `boolean`
  * baseline: `boolean`
  * labels: `boolean` \- Show/hide axis labels
  * gridlines: `boolean` \- Show/hide gridlines
  * min: `number` \- Minimum value for this axis (number for numeric axes, date string for date axes)
  * max: `number` \- Maximum value for this axis (number for numeric axes, date string for date axes)
  * fit_to_data: `boolean` \- Fit the axis to the data instead of including 0
  * interval: `number` \- Interval between axis ticks for numeric axes. This option is a suggestion, the actual interval may differ.

​

options group

Configure the secondary y-axis**Example:**
    
    
     y2_axis_options={
      title = "string"
      title_position = "top"
      ticks = true
      baseline = true
      labels = true
      gridlines = true
      min = 0
      max = 0
      fit_to_data = true
      interval = 0
    }
    

**Attributes:**

  * title: `string`
  * title_position: `string` \- Position of the axis title. “top” places it horizontally at the top, “side” places it vertically along the axis. Defaults to “side” for 100% stacked charts, “top” otherwise.
    * **Allowed values:**
      * `top`
      * `side`
  * ticks: `boolean`
  * baseline: `boolean`
  * labels: `boolean` \- Show/hide axis labels
  * gridlines: `boolean` \- Show/hide gridlines
  * min: `number` \- Minimum value for this axis (number for numeric axes, date string for date axes)
  * max: `number` \- Maximum value for this axis (number for numeric axes, date string for date axes)
  * fit_to_data: `boolean` \- Fit the axis to the data instead of including 0
  * interval: `number` \- Interval between axis ticks for numeric axes. This option is a suggestion, the actual interval may differ.

​

options group

Configure the x-axis**Example:**
    
    
     x_axis_options={
      title = "string"
      show_title = true
      label_wrap = true
      ticks = true
      baseline = true
      labels = true
      gridlines = true
      min = 0
      max = 0
      fit_to_data = true
      min_interval = "year"
      max_interval = "year"
      interval = 0
      label_rotate = 0
      title_arrow = true
      max_label_length = 0
    }
    

**Attributes:**

  * title: `string`
  * show_title: `boolean` \- When `true`, renders the auto-derived axis title (the x column name) below the chart. Ignored when `title` is set explicitly. Defaults to `false` — auto-derived column-name titles usually read as visual noise and the axis labels speak for themselves.
  * label_wrap: `boolean`
  * ticks: `boolean`
  * baseline: `boolean`
  * labels: `boolean` \- Show/hide axis labels
  * gridlines: `boolean` \- Show/hide gridlines
  * min: `number` \- Minimum value for this axis (number for numeric axes, date string for date axes)
  * max: `number` \- Maximum value for this axis (number for numeric axes, date string for date axes)
  * fit_to_data: `boolean` \- Fit the axis to the data instead of including 0. Defaults to true for numeric x-axes, false otherwise.
  * min_interval: `string` \- Minimum interval between axis ticks for time-based axes. This option is a suggestion, the actual interval may differ.
    * **Allowed values:**
      * `year`
      * `quarter`
      * `month`
      * `week`
      * `day`
      * `hour`
  * max_interval: `string` \- Maximum interval between axis ticks for time-based axes. This option is a suggestion, the actual interval may differ.
    * **Allowed values:**
      * `year`
      * `quarter`
      * `month`
      * `week`
      * `day`
      * `hour`
  * interval: `number` \- Interval between axis ticks for numeric axes. This option is a suggestion, the actual interval may differ.
  * label_rotate: `number` \- Rotation angle of axis label in degrees. Positive values rotate clockwise, negative values rotate counter-clockwise.
  * title_arrow: `boolean` \- Show/hide the arrow (→) on the axis title
  * max_label_length: `number` \- Maximum character length for axis labels. Labels exceeding this length will be truncated with an ellipsis. Defaults to 20 characters when labels are rotated.

​

boolean

default:"true"

Show legend. Studio’s built-in legend renders a compact color swatch + series name. For chart-wide style overrides that need the legend to reflect them precisely (line width, custom symbols, richer styling), set `legend=false` and provide `legend={ show=true ... }` inside `echarts_options` to use ECharts’ native legend instead.

​

string

default:"top"

Position of the legend (top or bottom)**Allowed values:**

  * `top`
  * `bottom`

​

array

Array of series names to define the order of series in the chart and legend. Series not in the array will appear after the ordered ones.

​

options group

Studio-shaped chart styling shortcuts (palette, series colors, zoom, padding). For raw ECharts overrides, use `echarts_options` instead.**Example:**
    
    
     chart_options={
      color_palette = ["#3b82f6", "#8b5cf6", "#ec4899"]
      series_colors = {
        "Series A" = "#3b82f6"
        "Series B" = "#10b981"
        "Series C" = "#f59e0b"
      }
      zoom = true
      top_padding = 0
    }
    

**Attributes:**

  * color_palette: `array of strings` \- Array of hex color codes to use for series colors
  * series_colors: `map of key-value pairs` \- Map of series names to hex color codes for custom series coloring
  * zoom: `boolean` \- Enables zoom by dragging on the chart area
  * top_padding: `number` \- Additional padding (in px) above the chart area to prevent labels from being cut off

​

map

Raw [ECharts options](<https://echarts.apache.org/en/option.html>) deep-merged over the chart’s final configuration. Use for anything the structured props do not expose — `dataZoom`, `visualMap`, `graphic`, tooltip styling, and so on. Partial overrides win key-by-key without clobbering Studio’s computed siblings. For per-series overrides, use `echarts_series_options` or the per-series `echarts_options` on a `line`/`bar`/etc. child.**Example:**
    
    
     echarts_options={
        tooltip={ position="top" }
        dataZoom=[{ type="slider" }]
    }
    

​

map

Raw [ECharts series options](<https://echarts.apache.org/en/option.html#series>) deep-merged into every data series in the chart. Use when the same override should apply to all series. Skips reference lines/areas/points. For a single series, set `echarts_options` on the series child instead.**Example:**
    
    
     echarts_series_options={
        itemStyle={ borderRadius=8 }
        markLine={ data=[{ type="average" }] }
    }
    

## 

​

Allowed Children

  * [area](</components/area>)
  * [bar](</components/bar>)
  * [bubble](</components/bubble>)
  * [line](</components/line>)
  * [scatter](</components/scatter>)
  * [reference_line](</components/reference_line>)
  * [reference_area](</components/reference_area>)
  * [reference_point](</components/reference_point>)

Was this page helpful?

⌘I
