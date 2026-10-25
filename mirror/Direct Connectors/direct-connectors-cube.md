---
type: external-doc
title: Cube
source: https://docs.evidence.dev/direct-connectors/cube
source_hash: 4141e8944c
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Cube

> 原文: <https://docs.evidence.dev/direct-connectors/cube>

The Cube direct connector points Evidence at [Cube’s SQL API](<https://cube.dev/docs/product/apis-integrations/core-data-apis/sql-api>), which speaks the Postgres wire protocol. Evidence connects as a Postgres client and queries your cubes and views as if they were tables, so your metrics stay defined once in Cube.

## 

​

Connecting

You need the SQL API **host** , **port** , **database** name, a **user** (`CUBEJS_SQL_USER`) and its **password** (`CUBEJS_SQL_PASSWORD`), plus an **SSL mode**.

1

Enable and find the SQL API

On **Cube Cloud** , open your deployment → **BI integrations** (or **Integrations → SQL API**) to get the host, port, and credentials. On **self-hosted Cube** , set `CUBEJS_PG_SQL_PORT` (commonly `15432`) and `CUBEJS_SQL_USER` / `CUBEJS_SQL_PASSWORD`. Enter the **hostname only** — not a `postgres://` URL.

2

Choose an SSL mode

Cube Cloud serves a publicly-trusted certificate, so keep the default **Verify full** (TLS with full certificate + hostname verification). For a **local self-hosted Cube** without TLS, use **Disable**. Lower modes exist (`verify-ca` skips the hostname check; `require` encrypts but does **not** verify the certificate) but aren’t recommended.

3

Configure the connector in Evidence

Go to **Connectors** in the sidebar, choose **Cube** in the **Warehouse** card, and fill in host, port, database, user, and password. Set the SSL mode.

4

Test and save

Click **Test Connection**. Once it passes, click **Save**.

## 

​

What Evidence queries

Cube’s SQL API implements a documented **subset** of PostgreSQL functions and operators ([reference](<https://docs.cube.dev/reference/core-data-apis/sql-api/reference>)). Evidence’s Cube dialect restricts the SQL-expression validator to that subset, so the editor flags an unsupported function before you run it rather than failing at query time. A few things to keep in mind when writing queries against Cube:

  * **Aggregates go through Cube’s model.** Use `MEASURE(cube.measure_name)` to reference measures defined in your data model, alongside the standard aggregates Cube supports (`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`, `STRING_AGG`, `PERCENTILE_CONT`, and the statistical aggregates).
  * **Joins between cubes are not arbitrary SQL joins** — they follow the joins defined in your Cube model. See Cube’s [SQL API joins](<https://cube.dev/docs/backend/sql/reference/joins>) documentation.
  * **Very large, post-processed`ORDER BY`** (beyond ~50k rows) can be applied inconsistently by Cube — filter or aggregate in the model where possible.

## 

​

Configuration reference

  * Studio

  * CLI (connection.yaml)

#### 

​

Credentials

​

string

required

Cube SQL API host, e.g. your-deployment.aws-us-east-1.cubecloudapp.dev (Cube Cloud) or the host running Cube.

​

number

Cube SQL API port. Cube Cloud uses 5432; self-hosted Cube uses CUBEJS_PG_SQL_PORT (commonly 15432). Default: `5432`.

​

string

required

Cube SQL API user (CUBEJS_SQL_USER).

​

string

required

Cube SQL API password (CUBEJS_SQL_PASSWORD).

​

string

required

Database name from Cube’s SQL API connection details (self-hosted Cube accepts any value).

​

`disable` | `require` | `verify-ca` | `verify-full`

verify-full (default) = encrypt and verify the server certificate + hostname — the secure choice, and what Cube Cloud serves. verify-ca = verify the cert but not the hostname. require = encrypt WITHOUT verifying the cert (vulnerable to an active man-in-the-middle — avoid). disable = no TLS, for a local self-hosted Cube without SSL. Default: `"verify-full"`.

​

string

PEM CA bundle used to verify the server certificate.

​

string

PEM client certificate for mutual TLS.

​

string

PEM client private key for mutual TLS.

​

string

Schema used when queries reference unqualified names. Cube exposes cubes and views in the public schema. Default: `"public"`.

#### 

​

Visibility

​

string[]

Allowlist of schemas exposed to the editor and schema browser. Defaults to just the default schema (the `schema` field) when empty. Default: `[]`.
    
    
    type: cube
    host: your-deployment.aws-us-east-1.cubecloudapp.dev
    # port: 5432
    user: <cubejs-sql-user>
    password: <cubejs-sql-password>
    database: cube
    # sslmode: verify-full
    # ssl_ca_path: ./certs/ca.pem # PEM CA bundle for verify-ca / verify-full
    # ssl_cert_path: ./certs/client.crt # mutual TLS client certificate
    # ssl_key_path: ./certs/client.key # mutual TLS client key
    # schema: public
    # schemas: [public]
    

#### 

​

Credentials

​

string

required

Cube SQL API host, e.g. your-deployment.aws-us-east-1.cubecloudapp.dev (Cube Cloud) or the host running Cube.

​

number

Cube SQL API port. Cube Cloud uses 5432; self-hosted Cube uses CUBEJS_PG_SQL_PORT (commonly 15432). Default: `5432`.

​

string

required

Cube SQL API user (CUBEJS_SQL_USER).

​

string

required

Cube SQL API password (CUBEJS_SQL_PASSWORD).

​

string

required

Database name from Cube’s SQL API connection details (self-hosted Cube accepts any value).

​

`disable` | `require` | `verify-ca` | `verify-full`

verify-full (default) = encrypt and verify the server certificate + hostname — the secure choice, and what Cube Cloud serves. verify-ca = verify the cert but not the hostname. require = encrypt WITHOUT verifying the cert (vulnerable to an active man-in-the-middle — avoid). disable = no TLS, for a local self-hosted Cube without SSL. Default: `"verify-full"`.

​

string

PEM CA bundle used to verify the server certificate.

​

string

Path to a PEM CA bundle, resolved relative to connection.yaml.

​

string

PEM client certificate for mutual TLS.

​

string

Path to a PEM client certificate, resolved relative to connection.yaml.

​

string

PEM client private key for mutual TLS.

​

string

Path to a PEM client private key, resolved relative to connection.yaml.

​

string

Schema used when queries reference unqualified names. Cube exposes cubes and views in the public schema. Default: `"public"`.

#### 

​

Visibility

​

string[]

Allowlist of schemas exposed to the editor and schema browser. Defaults to just the default schema (the `schema` field) when empty. Default: `[]`.

## 

​

Security Considerations

Evidence executes queries against your Cube deployment through the SQL API.

  1. Use a dedicated SQL API user for Evidence rather than sharing credentials.
  2. Scope what Evidence can read via your Cube data model and any [member-level access control](<https://cube.dev/docs/product/auth>) you configure in Cube.
  3. Require TLS (`verify-full` where possible) so credentials and data aren’t sent in the clear.
  4. Restrict network access to Evidence’s [egress IPs](</evidence-warehouse#ip-whitelisting>) plus your team’s networks.
  5. Rotate the SQL API password regularly.

Was this page helpful?

⌘I
