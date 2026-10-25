---
type: external-doc
title: Image
source: https://docs.evidence.dev/components/image
source_hash: afba275f86
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Image

> 原文: <https://docs.evidence.dev/components/image>

{% image
        url="https://raw.githubusercontent.com/evidence-dev/media-kit/refs/heads/main/png/wordmark-gray-800.png"
        description="Sample placeholder image"
    /%}
    

## 

​

Examples

### 

​

Basic Usage
    
    
    {% image
        url="https://raw.githubusercontent.com/evidence-dev/media-kit/refs/heads/main/png/wordmark-gray-800.png"
        description="Sample placeholder image"
    /%}
    

### 

​

With Dark Mode
    
    
    {% image
        url="https://raw.githubusercontent.com/evidence-dev/media-kit/refs/heads/main/png/wordmark-gray-800.png"
        dark_url="https://raw.githubusercontent.com/evidence-dev/media-kit/refs/heads/main/png/wordmark-white.png"
        description="Logo that changes in dark mode"
    /%}
    

### 

​

From a Data Query
    
    
    {% image
        data="products"
        column="image_url"
        description_column="product_name"
    /%}
    

### 

​

From a Data Query with Filters
    
    
    {% dropdown id="category_filter" data="products" value_column="category" /%}
    
    {% image
        data="products"
        column="image_url"
        description="Best selling product"
        filters=["category_filter"]
        order="sales desc"
    /%}
    

## 

​

Attributes

​

string

The URL of the image. Required unless data is provided.

​

string

The URL of the image to show in dark mode

​

string

Description of the image, used as alt text. Required unless description_column is provided.

​

string

Table or query to load the image URL from; the image updates when applied filters change

​

string

Column containing the image URL; the first row of the query result is displayed

​

string

Column containing the image URL to show in dark mode

​

string

Column containing the image description, used as alt text

​

array

IDs of filters to apply to the query

​

number

Maximum width of the image in pixels

​

boolean

default:"false"

Whether to show a border around the image

​

boolean

default:"false"

Whether to apply dither effect to the image

​

string

default:"center"

Alignment of the image**Allowed values:**

  * `left`
  * `right`
  * `center`

​

string

Additional CSS classes to apply to the image

​

number

Time in seconds between automatic data refreshes (minimum 30). Overrides the page-level auto-refresh setting for this component.

​

string

Custom SQL WHERE condition to apply to the query. For date filters, use date_range instead.

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

Was this page helpful?

⌘I
