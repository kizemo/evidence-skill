---
type: external-doc
title: Map
source: https://docs.evidence.dev/components/map
source_hash: 42b574c925
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Map

> 原文: <https://docs.evidence.dev/components/map>

![Basic Map](https://mintcdn.com/evidence/Z2CIIsynbphd2HX3/images/components/map/map.png?fit=max&auto=format&n=Z2CIIsynbphd2HX3&q=85&s=40bb7db3ec9cdc8a0215c48393d81563)
    
    
    {% map %}
    {% /map %}
    

## 

​

Examples

### 

​

Basic Map

![Basic Map](https://mintcdn.com/evidence/Z2CIIsynbphd2HX3/images/components/map/map.png?fit=max&auto=format&n=Z2CIIsynbphd2HX3&q=85&s=40bb7db3ec9cdc8a0215c48393d81563)
    
    
    {% map %}
    {% /map %}
    

### 

​

Map with Title

![Map with Title](https://mintcdn.com/evidence/Z2CIIsynbphd2HX3/images/components/map/example-1.png?fit=max&auto=format&n=Z2CIIsynbphd2HX3&q=85&s=1f2f1e4e97366cd24cec800f785dfe7c)
    
    
    {% map title="Sales by Region" %}
    {% /map %}
    

### 

​

Custom Height

![Custom Height](https://mintcdn.com/evidence/Z2CIIsynbphd2HX3/images/components/map/example-2.png?fit=max&auto=format&n=Z2CIIsynbphd2HX3&q=85&s=0666986b3d6947170d007f858af07cda)
    
    
    {% map height=500 %}
    {% /map %}
    

### 

​

Custom Zoom Level

![Custom Zoom Level](https://mintcdn.com/evidence/Z2CIIsynbphd2HX3/images/components/map/example-3.png?fit=max&auto=format&n=Z2CIIsynbphd2HX3&q=85&s=58630157fd46734cde230f72dfb5f57d)
    
    
    {% map zoom=10 %}
    {% /map %}
    

### 

​

Map with Choropleth Layer

![Map with Choropleth Layer](https://mintcdn.com/evidence/Z2CIIsynbphd2HX3/images/components/map/example-4.png?fit=max&auto=format&n=Z2CIIsynbphd2HX3&q=85&s=15cb2349486e795e6f6a997f9aa47752)
    
    
    ```sql east_coast_states
        select 'Maine' as state, 1362359 as population union all 
        select 'New Hampshire', 1377529 union all 
        select 'Vermont', 643077 union all 
        select 'Massachusetts', 7029917 union all 
        select 'Rhode Island', 1097379 union all 
        select 'Connecticut', 3605944 union all 
        select 'New York', 20201249 union all 
        select 'New Jersey', 9288994 union all 
        select 'Pennsylvania', 13002700 union all 
        select 'Delaware', 989948 union all 
        select 'Maryland', 6177224
    ```
    
    {% map %}
        {% area_layer
            geography="us_states"
            match_by="name"
            data="east_coast_states"
            area_id="state"
            value="population"
        /%}
    {% /map %}
    

### 

​

Map with Point Layer

![Map with Point Layer](https://mintcdn.com/evidence/Z2CIIsynbphd2HX3/images/components/map/example-5.png?fit=max&auto=format&n=Z2CIIsynbphd2HX3&q=85&s=4d6564f56c3364cc3d7cdf95d984bad0)
    
    
    ```sql us_stores
    select 40.7128 as lat, -74.0060 as lng, 1250000 as sales union all
    select 34.0522, -118.2437, 980000 union all
    select 41.8781, -87.6298, 875000 union all
    select 29.7604, -95.3698, 720000 union all
    select 33.4484, -112.0740, 650000 union all
    select 39.9526, -75.1652, 590000 union all
    select 29.4241, -98.4936, 480000 union all
    select 32.7157, -117.1611, 520000 union all
    select 32.7767, -96.7970, 810000 union all
    select 37.3382, -121.8863, 690000 union all
    select 30.2672, -97.7431, 560000 union all
    select 47.6062, -122.3321, 920000
    ```
    
    {% map title="Store Locations" %}
        {% point_layer
            data="us_stores"
            lat="lat"
            lng="lng"
            color_value="sales"
            color_palette=["#feedde", "#fdd0a2", "#fdae6b", "#fd8d3c", "#e6550d", "#a63603"]
        /%}
    {% /map %}
    

### 

​

Map with Heatmap Layer
    
    
    {% map title="Activity Density" %}
        {% heatmap_layer
            data="events"
            lat="latitude"
            lng="longitude"
            weight="event_count"
        /%}
    {% /map %}
    

## 

​

Attributes

​

string

Title to display above the map

​

string

Subtitle to display below the title

​

string

Information tooltip text (can only be used with title)

​

string

URL to link the info text to (can only be used with info)

​

string

Create a custom link title for the info link, placed after the info text (can only be used with info_link)

​

number

default:"300"

Height of the map in pixels

​

array

Initial map center position as [latitude, longitude]. Overrides auto-zoom to data bounds

​

number

Zoom level (0-22, where higher is more zoomed in). When provided without initial_position, centers on data at this zoom level. When provided with initial_position, uses this zoom at that position

​

boolean

default:"true"

Allow users to zoom in/out on the map

​

boolean

default:"true"

Allow users to pan/drag the map

​

string

default:"mono"

Base map style: “mono” for theme-aware monochrome basemap, “blank” for solid background only**Allowed values:**

  * `mono`
  * `blank`

​

string

default:"flat"

Map projection: “globe” for 3D globe view, “flat” for 2D flat map**Allowed values:**

  * `globe`
  * `flat`

​

boolean

default:"true"

Show legends for map layers

​

string

default:"bottom_right"

Location of the legend within the map**Allowed values:**

  * `top_left`
  * `top_right`
  * `bottom_left`
  * `bottom_right`

​

number

Set the width of this component (in percent) relative to the page width

## 

​

Allowed Children

  * [area_layer](</components/area_layer>)
  * [point_layer](</components/point_layer>)
  * [heatmap_layer](</components/heatmap_layer>)

Was this page helpful?

⌘I
