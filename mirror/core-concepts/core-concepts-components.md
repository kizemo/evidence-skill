---
type: external-doc
title: Components
source: https://docs.evidence.dev/core-concepts/components
source_hash: 40a3c0a795
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Components

> 原文: <https://docs.evidence.dev/core-concepts/components>

Components allow you to add data visualization to your reports.

## 

​

Syntax

Components are defined using special curly braces `{%` and `%}` tags.
    
    
    {% table                         # component name
        data="demo.order_details"    # attributes
    /%}
    

Bring up a list of available components by typing a `/` at the start of a line.

### 

​

Component Names

Component names are:

  * **Declarative** : they describe what type of UI element will be rendered
  * **Snake case** : they are written in snake_case
  * **Validated** : if you type an invalid component name, the editor will underline it in red

### 

​

Attributes

Attributes define the configuration of the component. Attbutes are:

  * **Required or Optional** : For a given component
  * **Typed** :
    * String `data="demo.order_details"`
    * Number `limit=20`
    * Boolean `stacked=true`
    * Object `x_axis_options={title='Date'}`
  * **Auto-completed** : When you hit enter or space, the editor will show a list of available attributes
  * **Validated** : if you type an invalid attribute name, or provide an invalid value, the editor will underline it in red, and provider detailed error messages on hover

### 

​

Quotes Inside Attributes

Attribute values are wrapped in double quotes. If the value itself needs to contain a double quote, escape it with a backslash (`\"`). This comes up with quoted SQL identifiers — for example a Snowflake column whose name contains a space:
    
    
    {% bar_chart
        data="DEMO.ORDER_DETAILS"
        x="DATE"
        y="SUM(\"Total Sales\")"
    /%}
    

The escaped quotes are passed through to the query as real double quotes, so the database receives `SUM("Total Sales")`.

### 

​

Self Closing Components

Some components contain content.
    
    
    {% callout %}
    This is a note
    {% /callout %}
    

Others do not.
    
    
    {% table
        data="demo.order_details"
    /%}
    

This is equivalent to:
    
    
    {% table data="demo.order_details" %}
    {% /table %}
    

Note the `/` at the end of the first example replaces the `{% /table %}` tag. Only components that do not require content can be self closed.

Was this page helpful?

⌘I
