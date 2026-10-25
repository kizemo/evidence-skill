---
type: external-doc
title: Download
source: https://docs.evidence.dev/components/download
source_hash: 406084488d
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Download

> 原文: <https://docs.evidence.dev/components/download>

![Basic Usage](https://mintcdn.com/evidence/Z2CIIsynbphd2HX3/images/components/download/download.png?fit=max&auto=format&n=Z2CIIsynbphd2HX3&q=85&s=740eb48391624dd255dbcd63a73bba61)
    
    
    {% download data="demo.daily_orders" /%}
    

## 

​

Examples

### 

​

Basic Usage

![Basic Usage](https://mintcdn.com/evidence/Z2CIIsynbphd2HX3/images/components/download/download.png?fit=max&auto=format&n=Z2CIIsynbphd2HX3&q=85&s=740eb48391624dd255dbcd63a73bba61)
    
    
    {% download data="demo.daily_orders" /%}
    

### 

​

Custom Label and Filename

![Custom Label and Filename](https://mintcdn.com/evidence/Z2CIIsynbphd2HX3/images/components/download/example-1.png?fit=max&auto=format&n=Z2CIIsynbphd2HX3&q=85&s=609f822f817d220dc9447aadf4ff2b7e)
    
    
    {% download
      data="demo.daily_orders"
      label="Export Orders"
      filename="daily_orders_export"
    /%}
    

### 

​

With Variant

![With Variant](https://mintcdn.com/evidence/Z2CIIsynbphd2HX3/images/components/download/example-2.png?fit=max&auto=format&n=Z2CIIsynbphd2HX3&q=85&s=f572827a3030a35fb6f748d58eda91f0)
    
    
    {% download
      data="demo.daily_orders"
      label="Export Data"
      variant="primary"
    /%}
    

### 

​

With Large Limit

![With Large Limit](https://mintcdn.com/evidence/Z2CIIsynbphd2HX3/images/components/download/example-3.png?fit=max&auto=format&n=Z2CIIsynbphd2HX3&q=85&s=b6d619bd936f3376162a0bef8a9df527)
    
    
    {% download
      data="demo.daily_orders"
      label="Download Recent Orders"
      limit=50000
    /%}
    

## 

​

Attributes

​

string

required

Table or view to download

​

string

default:"Download"

Text displayed on the download button

​

string

Name of the downloaded file (without extension)

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

array

IDs of filters to apply to the query

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

Was this page helpful?

⌘I
