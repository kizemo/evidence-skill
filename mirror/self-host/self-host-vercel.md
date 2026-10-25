---
type: external-doc
title: Self Host on Vercel
source: https://docs.evidence.dev/self-host/vercel
source_hash: 9ec58c2906
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Self Host on Vercel

> 原文: <https://docs.evidence.dev/self-host/vercel>

Vercel runs container images on [Vercel Functions](<https://vercel.com/docs/functions/container-images>) with autoscaling and Active CPU pricing.

1

Add a Dockerfile.vercel
    
    
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

Deploy

  * Using the Vercel UI

  * Using the Vercel CLI

Go to [vercel.com/new](<https://vercel.com/new>) and import the project’s GitHub repo.In the **Environment Variables** section of the import screen, add:

  * `EVIDENCE_BASIC_USER` — HTTP Basic Auth username
  * `EVIDENCE_BASIC_PASSWORD` — HTTP Basic Auth password
  * every `${VAR}` referenced in `connection.yaml`, here `SNOWFLAKE_USER` and `SNOWFLAKE_PRIVATE_KEY`

Click **Deploy**.

Create the project:
    
    
    vercel link
    

Accept the defaults — “No framework detected” is expected.Set the environment variables:
    
    
    vercel env add SNOWFLAKE_USER production
    vercel env add SNOWFLAKE_PRIVATE_KEY production
    vercel env add EVIDENCE_BASIC_USER production
    vercel env add EVIDENCE_BASIC_PASSWORD production
    

Repeat with `preview` to make preview deployments work too. Then deploy:
    
    
    vercel deploy --prod
    

For automatic deploys on every push, connect the repo:
    
    
    vercel git connect
    

Pushes to the default branch then deploy to production; other branches and PRs get preview URLs.

No `PORT` variable is needed — Vercel provides one and the image binds to it automatically.

## 

​

Notes

  * Container Images are in Beta on Vercel.
  * Functions scale to zero when idle; the first request after idle incurs a cold start.

Was this page helpful?

⌘I
