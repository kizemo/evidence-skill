---
type: external-doc
title: Connect your CLI to your data
source: https://docs.evidence.dev/cli/connections
source_hash: 3b84a001ec
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Connect your CLI to your data

> 原文: <https://docs.evidence.dev/cli/connections>

How the CLI gets data depends on how your project is set up.

## 

​

Evidence Warehouse

If you use the [Evidence Warehouse](</evidence-warehouse>), run `evidence login`. No further setup is needed.

## 

​

Other warehouses

If you use a [direct connection](</evidence-warehouse#direct-connectors>) to a warehouse you own, add a `connection.yaml` file to the root of your project. When this file is present, the CLI queries your warehouse **directly from your machine** — no login required.

### 

​

Scaffold with `evidence init`

The quickest way to create a `connection.yaml` is to pass `--warehouse` to `evidence init`:
    
    
    evidence init my-project --warehouse snowflake
    # or
    evidence init my-project --warehouse bigquery
    

This writes a `connection.yaml` in the project root pre-filled with placeholder values, and adds it to `.gitignore` (the file holds credentials — keep it out of version control).

`connection.yaml` and any files it references — the Snowflake private-key `.pem` or the BigQuery service-account JSON — contain credentials. Never commit them. `evidence init --warehouse` gitignores `connection.yaml` for you, but **not** the referenced key files: add those to `.gitignore` yourself, or use the inline `private_key` / `keyfile_json` fields so there’s no separate file to leak.

Fill in the placeholders with your own connection details. If `--warehouse` is omitted, no `connection.yaml` is written and the project uses the Evidence Warehouse.

### 

​

Configuration

  * Snowflake

  * BigQuery

    
    
    type: snowflake
    account: xy12345.us-east-1
    user: evidence_user
    private_key_path: ./snowflake-key.pem
    warehouse: COMPUTE_WH
    database: ANALYTICS
    # role: EVIDENCE_READER
    # schema: { production: ANALYTICS, devSchemas: [DEV_ALICE] }
    

Provide exactly one of `password`, `private_key`, or `private_key_path`. Paths (`private_key_path`) are resolved relative to `connection.yaml`.See the [Snowflake direct connector](</direct-connectors/snowflake>) for the full field reference and how to create a read-only Snowflake user.
    
    
    type: bigquery
    project: my-gcp-project
    keyfile: ./evidence-bq-primary.json
    datasets:
      - analytics
      - reporting
    # dataset: analytics
    # location: US
    

Provide exactly one of `keyfile` or `keyfile_json`. The `keyfile` path is resolved relative to `connection.yaml`.See the [BigQuery direct connector](</direct-connectors/bigquery>) for the full field reference and the service-account setup steps.

Was this page helpful?

⌘I
