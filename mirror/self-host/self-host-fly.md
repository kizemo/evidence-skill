---
type: external-doc
title: Self Host on Fly.io
source: https://docs.evidence.dev/self-host/fly
source_hash: 3b514ec494
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Self Host on Fly.io

> 原文: <https://docs.evidence.dev/self-host/fly>

Fly.io runs the container as Firecracker VMs close to your users, with optional scale-to-zero. Fly needs two pieces of configuration the other platforms don’t: **memory** (the default 256mb machine is too small and the server gets OOM-killed at boot) and **port** (Fly routes to `internal_port` but does not provide `PORT` to the container, so the server listens on its default `3000`).

1

Add a Dockerfile
    
    
    FROM evidencedev/serve:latest
    COPY --chown=evidence:evidence . /project
    

2

Modify and commit connection.yaml

The project must use a [direct connector](</cli/connections>), with secrets replaced by `${VAR}` environment references, for example:
    
    
    type: snowflake
    account: xy12345.us-east-1
    user: ${SNOWFLAKE_USER}
    private_key: ${SNOWFLAKE_PRIVATE_KEY}
    warehouse: COMPUTE_WH
    database: ANALYTICS
    

`connection.yaml` is gitignored by default. Once its secrets are `${VAR}` placeholders, remove it from `.gitignore` and commit it — the container reads it at runtime.

3

Deploy

  * Using the Fly UI

  * Using the Fly CLI

In the [dashboard](<https://fly.io/dashboard>), click **Launch an App** , choose **Launch an App from GitHub** , and select the project’s repo, granting Fly access if prompted.Click **Customize deploy** and set:

  * **Internal port** : `3000`
  * **Memory** : at least 512MB memory
  * **Environment Variables** : `EVIDENCE_BASIC_USER`, `EVIDENCE_BASIC_PASSWORD`, and every `${VAR}` referenced in `connection.yaml`, here `SNOWFLAKE_USER` and `SNOWFLAKE_PRIVATE_KEY`

Deploy, then optionally enable **Auto Deploy** under **Deployments → Settings** so pushes to the default branch redeploy automatically.

Launch without deploying to generate `fly.toml`:
    
    
    fly launch --no-deploy
    

Edit `fly.toml` so the port and memory are right:
    
    
    [http_service]
      internal_port = 3000   # the port the image listens on
    
    [[vm]]
      memory = '512mb'       # or more - default 256mb OOM-kills the server
    

Use a single memory key. `fly launch` scaffolds sometimes include `memory_mb` — if both `memory` and `memory_mb` are present with different values, the wrong one can win.

Set the secrets:
    
    
    fly secrets set \
      SNOWFLAKE_USER=... \
      SNOWFLAKE_PRIVATE_KEY=... \
      EVIDENCE_BASIC_USER=reports \
      EVIDENCE_BASIC_PASSWORD=changeme
    

Then deploy:
    
    
    fly deploy
    

Healthy logs show `Ready at http://localhost:3000`.

## 

​

Notes

  * Ensure fly has enough memory or else you may get `Out of memory: Killed process (evidence)` in `fly logs`. Increase memory with `fly scale memory 512` or in `fly.toml`.

Was this page helpful?

⌘I
