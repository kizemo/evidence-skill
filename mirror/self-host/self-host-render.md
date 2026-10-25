---
type: external-doc
title: Self Host on Render
source: https://docs.evidence.dev/self-host/render
source_hash: 7e4c77e462
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Self Host on Render

> 原文: <https://docs.evidence.dev/self-host/render>

Render builds a `Dockerfile` from a connected repo.

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

Create a Web Service

In the Render dashboard: **New → Web Service** , then connect the project’s GitHub repo.Choose **Web Service**. Render detects the `Dockerfile` automatically and selects the Docker runtime; no build or start commands are needed.

4

Set environment variables

Add under **Environment** :

  * `EVIDENCE_BASIC_USER` — HTTP Basic Auth username
  * `EVIDENCE_BASIC_PASSWORD` — HTTP Basic Auth password
  * every `${VAR}` referenced in `connection.yaml`, here `SNOWFLAKE_USER` and `SNOWFLAKE_PRIVATE_KEY`

Click **Create Web Service** to deploy. Pushes to the connected branch redeploy automatically.

## 

​

Notes

  * The free tier pauses the service when idle. Cold starts will cause longer load times when restarting.

Was this page helpful?

⌘I
