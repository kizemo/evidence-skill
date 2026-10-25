---
type: external-doc
title: Commands
source: https://docs.evidence.dev/cli/commands
source_hash: 628a6c4c81
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Commands

> 原文: <https://docs.evidence.dev/cli/commands>

## 

​

Development

### 

​

`evidence init`

Scaffold a new Evidence project.
    
    
    evidence init my-project
    evidence init my-project --warehouse snowflake
    

Option| Description  
---|---  
`--warehouse`| Scaffold a `connection.yaml` for a direct warehouse connection: `snowflake` or `bigquery`. See [Connecting a Warehouse](</cli/connections>).  
  
Without `--warehouse`, the project uses the managed Evidence Warehouse. With it, the CLI writes a `connection.yaml` (gitignored) so queries run directly against your own warehouse.

### 

​

`evidence dev`

Start the local development server.
    
    
    evidence dev
    evidence dev --port 8080
    

Option| Description  
---|---  
`--port`, `-p`| Port to run on (default: 3000)  
  
## 

​

Deploy

### 

​

`evidence launch`

Connect this project to Evidence Studio and GitHub, then deploy by pushing to your repo. Interactive by default — it walks you through installing the GitHub App and creating (or linking) the Studio project.
    
    
    evidence launch
    evidence launch --name "My Project" --upload-credentials
    

Option| Description  
---|---  
`--name`| Name for the Studio project to create.  
`--branch`| Branch to publish from (default: auto-detected).  
`--root-directory`| Project root within the repo (default: auto-detected).  
`--upload-credentials`| Upload warehouse credentials from [`connection.yaml`](</cli/connections>) to your org. Prompted for interactively; required to opt in when running non-interactively.  
  
`evidence launch` recognizes GitHub.com and GitHub Enterprise Cloud (`*.ghe.com`) remotes over HTTPS and SSH. If the detected GHE.com host is not connected yet, the CLI opens Organization Settings to guide an Evidence admin through creating and installing the private Enterprise GitHub App. GHE.com connections require Evidence Studio on the Enterprise plan.

### 

​

`evidence link`

Attach this repo to an **existing** Studio project. The project’s content becomes git-managed — the next `git push` overwrites it with the repo’s contents.
    
    
    evidence link my-project
    evidence link --project my-project
    

Option| Description  
---|---  
`--project`| Studio project to attach, by slug or id. Also accepted as a positional argument.  
`--branch`| Branch to publish from (default: auto-detected).  
`--root-directory`| Project root within the repo (default: auto-detected).  
  
The repository host must match the GitHub host connected to the Evidence organization.

### 

​

`evidence unlink`

Disconnect this repo from its Studio project. The org’s GitHub App install and the project itself are left intact.
    
    
    evidence unlink
    evidence unlink --project my-project
    

Option| Description  
---|---  
`--project`| Studio project to unlink, by slug or id. Inferred from the repo’s binding when omitted. Also accepted as a positional argument.  
  
## 

​

Data

### 

​

`evidence query`

Run a SQL query against connected data sources.
    
    
    evidence query "SELECT * FROM orders LIMIT 10"
    evidence query --file query.sql
    cat query.sql | evidence query -
    

Option| Description  
---|---  
`--format`| Output format: `json` (default), `jsonl`, `table`, `csv`  
`--limit`, `-l`| Limit number of rows  
`--quiet`, `-q`| Suppress status messages  
`--columns`| Output only column names and types  
`--file`| Read SQL from a file  
`--output`, `-o`| Write results to a file  
  
### 

​

`evidence tables`

List available data tables.
    
    
    evidence tables
    

### 

​

`evidence describe`

Show schema for a table.
    
    
    evidence describe demo_daily_orders
    

### 

​

`evidence schema`

Show all tables with their columns.
    
    
    evidence schema
    

### 

​

`evidence connectors`

List all connectors with health status.
    
    
    evidence connectors
    evidence connectors --json
    

Shows connector name, type, status, last sync, schedule, table count, and reference count for all connector types.

### 

​

`evidence models`

List all models with refresh status.
    
    
    evidence models
    evidence models --json
    

Shows model name, status, last refresh time, and schedule.

### 

​

`evidence lineage`

Show where connections and tables are used across projects, pages, models, and queries.
    
    
    evidence lineage
    evidence lineage table orders
    evidence lineage --branch dev/alex
    evidence lineage --json
    

Option| Description  
---|---  
`table <name>`| Show where a specific table is referenced  
`--branch`| Query a specific branch (default: main)  
`--json`| Output JSON for tooling  
  
## 

​

Project

### 

​

`evidence validate`

Validate all markdown files in the current project.
    
    
    evidence validate
    evidence validate --json
    

### 

​

`evidence docs`

Browse and search component documentation.
    
    
    evidence docs                         # Overview
    evidence docs search "chart"          # Search docs
    evidence docs component bar_chart     # Component details
    

## 

​

Authentication

### 

​

`evidence signup`

Create an account and workspace. Opens Evidence in your browser; once done, run `evidence login` to connect the CLI.
    
    
    evidence signup
    

### 

​

`evidence login`

Authenticate with Evidence Studio using your browser.
    
    
    evidence login
    

### 

​

`evidence logout`

Clear stored credentials.
    
    
    evidence logout
    

### 

​

`evidence whoami`

Show the currently authenticated user.
    
    
    evidence whoami
    

### 

​

`evidence orgs`

List available organizations.
    
    
    evidence orgs
    

### 

​

`evidence switch`

Switch to a different organization.
    
    
    evidence switch "My Organization"
    

### 

​

`evidence token`

Generate a token for headless/CI use. Prints an `EVIDENCE_AUTH_TOKEN` value you can set as an environment variable to authenticate without an interactive login. Requires an active login and a selected organization.
    
    
    evidence token
    

The generated token contains a long-lived access token. Treat it as a secret.

## 

​

Utility

### 

​

`evidence version`

Print the current CLI version.
    
    
    evidence version
    

### 

​

`evidence upgrade`

Upgrade to the latest CLI version.
    
    
    evidence upgrade
    

### 

​

`evidence help`

Show help and available commands.
    
    
    evidence help
    

### 

​

`evidence telemetry`

Show or change anonymous usage reporting. See [Telemetry](</cli#telemetry>) for what is sent.
    
    
    evidence telemetry            # same as status
    evidence telemetry status
    evidence telemetry disable
    evidence telemetry enable
    

## 

​

Self-hosting

### 

​

`evidence serve`

Serve the project as a self-hosted site. Runs the built-in production server — no Evidence Studio account or session required.
    
    
    evidence serve
    evidence serve --port 8080
    evidence serve --host 0.0.0.0
    

Option| Description  
---|---  
`--port`, `-p`| Port to run on (default: 3000)  
`--host`| Bind address (default: `127.0.0.1`)  
`--open`| Open the browser after starting (default: off)  
  
Requires a direct connector configured in [`connection.yaml`](</cli/connections>) — the managed query engine is not supported for self-hosting. To serve beyond localhost, set `EVIDENCE_BASIC_USER` and `EVIDENCE_BASIC_PASSWORD` to protect the site with HTTP Basic Auth. The server refuses to start on a non-localhost `--host` without these set. On a trusted private network (VPN, Tailscale, internal VPC), `EVIDENCE_AUTH_DISABLED=true` skips authentication entirely.

Was this page helpful?

⌘I
