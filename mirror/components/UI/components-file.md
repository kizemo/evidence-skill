---
type: external-doc
title: File
source: https://docs.evidence.dev/components/file
source_hash: e429a81c3c
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# File

> 原文: <https://docs.evidence.dev/components/file>

{% file
        url="https://files.example.com/reports/q3-board-deck.pdf"
        name="Q3 Board Deck.pdf"
        date="2026-09-29"
    /%}
    

## 

​

Examples

### 

​

Basic Usage
    
    
    {% file
        url="https://files.example.com/reports/q3-board-deck.pdf"
        name="Q3 Board Deck.pdf"
        date="2026-09-29"
    /%}
    

### 

​

Uploaded File
    
    
    {% file
        url="evd_org_01ABC/k7Xq2mB9.xlsx"
        name="Regional Sales Model.xlsx"
        date="2026-09-29"
    /%}
    

## 

​

Attributes

​

string

required

The upload alias returned when a file is dropped into the editor (evd_…), or an external URL to link to.

​

string

Display name for the file. Defaults to the filename from the URL.

​

string

File type label shown on the card (e.g. PDF, Excel). Inferred from the file extension when omitted.

​

string

Date shown on the card, e.g. the upload date. Accepts YYYY-MM-DD, an ISO timestamp, or free text.

​

number

Set the width of this component (in percent) relative to the page width

Was this page helpful?

⌘I
