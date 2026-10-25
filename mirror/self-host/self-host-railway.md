---
type: external-doc
title: Self Host on Railway
source: https://docs.evidence.dev/self-host/railway
source_hash: 70028e8935
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Self Host on Railway

> 原文: <https://docs.evidence.dev/self-host/railway>

Railway builds the `Dockerfile` from a connected repo.

1

Add a Dockerfile
    
    
    FROM evidencedev/serve:latest
    COPY --chown=evidence:evidence . /project
    

2

Modify and commit connection.yaml

The project must use a [direct connector](</cli/connections>), with secrets replaced by `${VAR}` environment references:
    
    
    type: snowflake
    account: xy12345.us-east-1
    user: ${SNOWFLAKE_USER}
    private_key: ${SNOWFLAKE_PRIVATE_KEY}
    warehouse: COMPUTE_WH
    database: ANALYTICS
    

`connection.yaml` is gitignored by default. Once its secrets are `${VAR}` placeholders, remove it from `.gitignore` and commit it — the container reads it at runtime.

3

Create the service

In the Railway dashboard: **New Project → Deploy from GitHub repo** , and select the project’s repo. Railway detects the `Dockerfile` automatically.

4

Set service variables

On the service, open the **Variables** tab and add:

  * `EVIDENCE_BASIC_USER` — HTTP Basic Auth username
  * `EVIDENCE_BASIC_PASSWORD` — HTTP Basic Auth password
  * every `${VAR}` referenced in `connection.yaml`, here `SNOWFLAKE_USER` and `SNOWFLAKE_PRIVATE_KEY`

Use plain **service variables** — shared variables are for referencing values across multiple services. Mark secrets as sealed to hide their values after saving. No `PORT` variable is needed — Railway provides one and the image binds to it automatically.

Was this page helpful?

⌘I
