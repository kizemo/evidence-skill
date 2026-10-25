---
type: external-doc
title: Variables
source: https://docs.evidence.dev/core-concepts/variables
source_hash: 13ad031b43
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Variables

> 原文: <https://docs.evidence.dev/core-concepts/variables>

## 

​

Defining Variables

### 

​

In Frontmatter

Variables can be defined in [YAML](<https://quickref.me/yaml.html>) frontmatter at the top of your document:
    
    
    ---
    company:
      name: Acme Corp
      industry: Manufacturing
      employees: 121
    category: Home
    ---
    

These live in the same frontmatter block as a page’s [settings](</features/page-settings>) — any key that isn’t a reserved page setting is treated as a variable.

### 

​

In Inputs

Some components create variables that can be used elsewhere in your markdown, for example the [dropdown](</components/dropdown>) component. When you add an input component, it creates a variable using the `id` attribute:
    
    
    {% dropdown
      id="category_dropdown" 
      data="demo.daily_orders"
      value_column="category"
    /%}
    

All available filters for your page can be found in the **Filters** tab in the sidebar.

### 

​

From a Reporting Period

A page that declares a [`workflow.period`](</features/page-settings#workflow-period>) block gets a `period` variable and a period picker, without adding any component:
    
    
    ---
    workflow:
      period:
        grain: month
    ---
    
    # {{ period.label }} Review
    

## 

​

Built-in Variables

Some variables are automatically available based on the logged-in user:

Variable| Example  
---|---  
`{{ $user.first_name }}`| Edward  
`{{ $user.last_name }}`| Tufte  
`{{ $user.email }}`| [edward@tufte.com](<mailto:edward@tufte.com>)  
`{{ $organization.name }}`| Beautiful Evidence  
  
## 

​

Using Variables

Reference variables using double curly braces `{{ my_variable }}`. Variables defined in frontmatter use the `$` prefix: `{{ $my_variable }}`.

### 

​

In Markdown

Use variables inline in your text content:

#### 

​

From Frontmatter
    
    
    # {{ $company.name }} Dashboard
    
    {{ $company.name }} has {{ $company.employees }} employees.
    

#### 

​

From Inputs
    
    
    # {{ category_dropdown }} Performance
    

### 

​

In Component Attributes

Reference variables in component attributes with curly braces `{{}}`:

#### 

​

From Frontmatter
    
    
    {% line_chart
      data="demo.daily_orders"
      title="Revenue for {{ $company.name }}" 
      x="date"
      y="sum(total_sales)"
    /%}
    

Frontmatter variables also support direct assignment, useful for passing arrays and objects.
    
    
    {% line_chart
      data="demo.daily_orders"
      title=$company.name 
      x="date"
      y="sum(total_sales)"
    /%}
    

#### 

​

From Inputs
    
    
    {% line_chart
      data="demo.daily_orders"
      title="Sales for {{ category_dropdown }}" 
      x="date"
      y="sum(total_sales)"
    /%}
    

Note that not all attributes support input variables.

### 

​

In SQL

#### 

​

From Frontmatter
    
    
    ```sql home_sales
    select * from demo.daily_orders
    where category = '{{ $category }}'
    ```
    

#### 

​

From Inputs

Use variables in SQL queries for dynamic filtering:
    
    
    ```sql home_sales
    select * from demo.daily_orders
    where category = {{ category_dropdown }} 
    ```
    

## 

​

Advanced Usage

### 

​

Fallback Values

Add fallback values using pipes `|`:
    
    
    ```sql category_analysis
    select * from demo.daily_orders
    where category = {{ category_dropdown | 'Electronics' }} 
    ```
    

### 

​

Conditional Blocks

Conditional blocks `[[...]]` will only be included if **all** variables within it resolve to non-empty values:
    
    
    ```sql category_analysis
    select * from demo.order_details
    where transaction_amount > 500
    [[ and category = {{ category_dropdown }} ]] 
    ```
    

### 

​

Context-Aware Defaults

Input component variables automatically use context-aware defaults, so you can reference them directly without specifying a property:

  * **In SQL contexts** (SQL queries, `where` clauses): Uses `.selected` (includes quotes)
  * **In text contexts** (titles, markdown text): Uses `.literal` (excludes quotes)

    
    
    ```sql category_analysis
    select * from demo.daily_orders
    where category = {{ category_dropdown }} 
    ```
    
    
    
    {% line_chart
      data="category_analysis"
      title="Sales for {{ category_dropdown }}" 
    /%}
    

### 

​

Explicit Property Reference

You can explicitly specify which property to use. Each filter component offers different properties. For example, the [dropdown component](</components/dropdown>) offers `.selected`, `.literal`, and `.filter` properties:
    
    
    ```sql category_analysis
    select * from demo.daily_orders
    where category = {{ category_dropdown.selected }} 
    ```
    

### 

​

Nested Access

Use dot notation to access nested properties in frontmatter variables, and square brackets to access array indices:
    
    
    {{ $company.name }} # Acme
    {{ $targets.revenue }} # 1000000
    {{ $colors[0] }} # red
    

### 

​

Data Types

Variables can be:

  * **Strings** : `name: John Doe`
  * **Numbers** : `age: 30`
  * **Booleans** : `active: true`
  * **Arrays** : `colors: [red, green, blue]`
  * **Objects** : `company: { name: Acme, industry: Manufacturing }`

Was this page helpful?

⌘I
