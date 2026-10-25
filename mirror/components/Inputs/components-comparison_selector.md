---
type: external-doc
title: Comparison Selector
source: https://docs.evidence.dev/components/comparison_selector
source_hash: b8f6e9c7e5
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Comparison Selector

> 原文: <https://docs.evidence.dev/components/comparison_selector>

![Using comparison](https://mintcdn.com/evidence/Z2CIIsynbphd2HX3/images/components/comparison_selector/comparison_selector.png?fit=max&auto=format&n=Z2CIIsynbphd2HX3&q=85&s=0487d947bfc5f61d49b99078e5fc468a)
    
    
    {% comparison_selector
        id="comp"
        default_value="prior year"
    /%}
    
    {% big_value
        data="demo.daily_orders"
        value="sum(total_sales)"
        fmt="usd1m"
        date_range={
            date="date"
            range="last 12 months"
        }
        comparison={
            compare_vs={{comp}}
        }
    /%}
    

## 

​

Examples

### 

​

Using `comparison`

![Using comparison](https://mintcdn.com/evidence/Z2CIIsynbphd2HX3/images/components/comparison_selector/comparison_selector.png?fit=max&auto=format&n=Z2CIIsynbphd2HX3&q=85&s=0487d947bfc5f61d49b99078e5fc468a)
    
    
    {% comparison_selector
        id="comp"
        default_value="prior year"
    /%}
    
    {% big_value
        data="demo.daily_orders"
        value="sum(total_sales)"
        fmt="usd1m"
        date_range={
            date="date"
            range="last 12 months"
        }
        comparison={
            compare_vs={{comp}}
        }
    /%}
    

## 

​

Attributes

​

string

required

The id of the comparison selector to be used in SQL query templates

​

array

Optional array of preset comparison values to show. If not provided, all comparison options will be available.

​

string

Default comparison to select on load**Allowed values:**

  * `prior year`
  * `prior period`
  * `target`
  * `benchmark`

​

string

default:"Comparison"

Text displayed above the selector

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

Placeholder text displayed when no value is selected

​

string

default:"triangle"

Icon to display

​

number

Set the width of this component (in percent) relative to the page width

## 

​

Using the Filter Variable

Reference this filter using `{{filter_id}}`. The value returned depends on where you use it.

Context| Default Property| No Selection| Result  
---|---|---|---  
Inline SQL query| `.comparison`| | `{"compare_vs": "prior year"}`  
`where` attribute| `.comparison`| | `{"compare_vs": "prior year"}`  
Text / Markdown| `.comparison`| | `{"compare_vs": "prior year"}`  
  
### 

​

Available Properties

You can also access specific properties using `{{filter_id.property}}`:

#### 

​

.selected

Returns the selected comparison value with quotes. Returns an empty string when no value is selected.
    
    
    {% comparison_selector id="comparison" preset_values=["prior year", "prior period"] /%}
    
    ```sql sales_comparison
    select 
      date,
      sum(sales) as total_sales,
      '{{comparison.selected}}' as comparison_type
    from orders
    group by date
    ```
    

**Example value:** `'prior year'`

#### 

​

.literal

Returns the raw unescaped selected value.
    
    
    {% comparison_selector id="comparison" preset_values=["prior year", "prior period"] /%}
    
    ```sql sales_comparison
    select 
      date,
      sum(sales) as total_sales,
      {{comparison.literal}} as comparison_type
    from orders
    group by date
    ```
    

**Example value:** `prior year`

#### 

​

.compare_vs

Returns the comparison type. **Example value:** `prior year`

#### 

​

.comparison

Returns the full comparison configuration as JSON. Use `{{comp}}` directly with `compare_vs` to dynamically configure comparisons.
    
    
    {% comparison_selector id="comp" /%}
    
    {% big_value 
        data="demo.daily_orders"
        value="sum(total_sales)"
        comparison={ compare_vs={{comp}} }
    /%}
    

**Example value:** `{"compare_vs": "prior year"}`

#### 

​

.agg

For benchmark comparisons: returns the aggregation function. **Example value:** `avg`

#### 

​

.subject

For benchmark comparisons: returns the subject column that defines entities. **Example value:** `region`

#### 

​

.target

For target comparisons: returns the target value or expression. **Example value:** `100000`

## 

​

Allowed Children

  * [benchmark_comparison](</components/benchmark_comparison>)
  * [target_comparison](</components/target_comparison>)

Was this page helpful?

⌘I
