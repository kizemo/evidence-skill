---
type: external-doc
title: Docs MCP
source: https://docs.evidence.dev/mcp/docs
source_hash: 4c2a5fcaf2
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Docs MCP

> 原文: <https://docs.evidence.dev/mcp/docs>

This is a **developer** MCP — it searches the documentation and explains the Evidence markdown syntax. It does **not** connect to your data. To query your own Evidence instance, use the **[Studio MCP](</mcp/studio>)** instead.

The Evidence Docs MCP server lets AI assistants search and read the [Evidence documentation](<https://docs.evidence.studio>). It’s available at <https://evidence.studio/mcp> and uses Streamable HTTP.

## 

​

Built-in to Evidence Editor

The Docs MCP server is **integrated into the Evidence Editor AI chat by default**. When you use the AI assistant in the Evidence Editor, it automatically has access to all Evidence documentation and component information—no additional configuration needed.

## 

​

Adding to Other AI Tools

You can add the Evidence Docs MCP server to other AI tools that support MCP:

  * Claude Code

  * Cursor

  * ChatGPT

  * Claude.ai

    
    
    claude mcp add --transport http evidence-studio https://evidence.studio/mcp
    

Add the following to your Cursor MCP configuration (`.cursor/mcp.json`):
    
    
    {
      "mcpServers": {
        "evidence-studio": {
          "url": "https://evidence.studio/mcp"
        }
      }
    }
    

ChatGPT requires enabling Developer Mode to use MCP servers:

  1. Go to Settings → [ChatGPT Security](<https://chatgpt.com/plugins#settings/Security>)
  2. Scroll to find Developer Mode, and toggle it on.
  3. Navigate to [ChatGPT Plugins](<https://chatgpt.com/plugins>) in the sidebar, then click the New Plugin button `+`

  * (Optional) Add an image (e.g., the [Evidence logo](<https://github.com/evidence-dev/media-kit/blob/main/png/logo-square-white-on-black.png>))
  * Enter a name (“Evidence Docs”)
  * Enter a description (“Search the Evidence documentation and component reference”)
  * Enter the Remote MCP server URL: `https://evidence.studio/mcp`
  * Set Auth to **No Auth**

  4. Click “I accept and want to continue” then `Create`

The Evidence documentation tools will now be available in your ChatGPT conversations.

  1. Go to [Claude.ai Settings → Connectors](<https://claude.ai/settings/connectors>)
  2. Click **Add Custom Connector**
  3. Enter a name (e.g., “Evidence Docs”)
  4. Paste the Remote MCP server URL: `https://evidence.studio/mcp`
  5. Click **Add** to save

The Evidence documentation tools will now be available in your Claude.ai conversations.

## 

​

Available Tools

The Docs MCP server provides the following tools for AI assistants:

Tool| Description  
---|---  
`list_components`| Get the full list of Evidence components organized by category  
`get_component`| Get metadata for a component including name, description, category, example, and attributes summary  
`get_component_attributes`| Get detailed attribute information for a component (type, required, default, description)  
`get_component_examples`| Get all code examples for a specific component  
`list_docs`| List all documentation pages with titles and paths, optionally filtered by category  
`read_doc`| Get the full content of a documentation page  
`search_docs`| Search across all Evidence documentation  
  
## 

​

LLM-Friendly Documentation

For AI tools that don’t support MCP, the Evidence documentation is also available in plain text formats:

  * [docs.evidence.studio/llms.txt](<https://docs.evidence.studio/llms.txt>) \- Sitemap of all documentation pages
  * [docs.evidence.studio/llms-full.txt](<https://docs.evidence.studio/llms-full.txt>) \- Full documentation in a single markdown file

Was this page helpful?

⌘I
