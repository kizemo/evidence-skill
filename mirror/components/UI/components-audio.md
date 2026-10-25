---
type: external-doc
title: Audio
source: https://docs.evidence.dev/components/audio
source_hash: be4e109bca
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Audio

> 原文: <https://docs.evidence.dev/components/audio>

{% audio url="https://files.example.com/podcast/episode-01.mp3" /%}
    

## 

​

Examples

### 

​

Basic Audio Player
    
    
    {% audio url="https://files.example.com/podcast/episode-01.mp3" /%}
    

### 

​

WAV File with Title
    
    
    {% audio url="https://files.example.com/recordings/call.wav" type="wav" title="Call Recording" /%}
    

## 

​

Attributes

​

string

required

URL of the audio file (may include query parameters such as SAS tokens)

​

string

default:"mpeg"

MIME subtype of the audio file (e.g. mpeg, wav, ogg, mp4). Mapped to audio/[type] for the browser.

​

string

Accessible label for the audio player

Was this page helpful?

⌘I
