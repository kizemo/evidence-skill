---
type: external-doc
title: MotherDuck
source: https://docs.evidence.dev/direct-connectors/motherduck
source_hash: cc2d15cac8
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# MotherDuck

> 原文: <https://docs.evidence.dev/direct-connectors/motherduck>

The MotherDuck direct connector lets Evidence run queries live against a [MotherDuck](<https://motherduck.com>) database. It connects over MotherDuck’s [Postgres-wire endpoint](<https://motherduck.com/docs/key-tasks/authenticating-and-connecting-to-motherduck/postgres-endpoint/>) — the connection method MotherDuck recommends for applications and serverless environments — so no local DuckDB engine is involved.

## 

​

Connecting

You’ll create a MotherDuck access token, find your database name, then enter both in Evidence.

### 

​

Prerequisites

  * A **MotherDuck account** with a **database** that holds the data you want Evidence to query.
  * Permission to **create access tokens** in your MotherDuck organization.

### 

​

1\. Create an access token

Evidence authenticates with a MotherDuck **access token** rather than a username/password, which keeps the credential headless and scoped.

1

Open token settings

In the [MotherDuck UI](<https://app.motherduck.com>), open **Settings → Access Tokens**.

2

Create a read-scoped token

Click **Create token** , give it a name like `Evidence`, and choose a **read-scoped** token where possible (sufficient for querying). Copy the token value immediately — it’s only shown once.

### 

​

2\. Find your database name

Your database name is the one shown in the MotherDuck UI’s database explorer. This is what you enter as **Database** in Evidence.

MotherDuck’s Postgres endpoint is region-specific. The connector defaults to `pg.us-east-1-aws.motherduck.com`; if your database lives in another region, override the **Host** field with that region’s endpoint.

### 

​

3\. Configure the connector in Evidence

1

Open the connectors page

Go to **Connectors** in the sidebar. In the **Warehouse** card at the top, select **MotherDuck**.

2

Enter connection details

Fill in the **Database** (your MotherDuck database name) and the **Service Token** you created. Leave **Host** at its default unless your database is in a non-us-east-1 region.

3

Test and save

Click **Test Connection**. Once it passes, click **Save**.

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

MotherDuck access (service) token. Sent as the password when connecting to the MotherDuck Postgres endpoint.

​

string

required

MotherDuck database name to query.

​

string

MotherDuck Postgres endpoint host. Region-specific; the default targets us-east-1. Override for other regions (e.g. pg.eu-west-1-aws.motherduck.com). Default: `"pg.us-east-1-aws.motherduck.com"`.

#### 

​

Visibility

​

string[]

Allowlist of schemas exposed to the editor and schema browser. Default: `[]`.
    
    
    type: motherduck
    token: your-motherduck-service-token
    database: my_database
    # schemas: [main, analytics]
    

#### 

​

Credentials

​

string

required

MotherDuck access (service) token. Sent as the password when connecting to the MotherDuck Postgres endpoint.

​

string

required

MotherDuck database name to query.

​

string

MotherDuck Postgres endpoint host. Region-specific; the default targets us-east-1. Override for other regions (e.g. pg.eu-west-1-aws.motherduck.com). Default: `"pg.us-east-1-aws.motherduck.com"`.

#### 

​

Visibility

​

string[]

Allowlist of schemas exposed to the editor and schema browser. Default: `[]`.

## 

​

SQL dialect

MotherDuck runs [DuckDB](<https://duckdb.org>) SQL (PostgreSQL-flavoured). A few things worth knowing when writing queries:

  * Row limiting uses `LIMIT <n> [OFFSET <m>]`, and `GROUP BY ALL` groups by every non-aggregate select expression.
  * Strings concatenate with `||`; use `ILIKE` for case-insensitive matches and `IS NOT DISTINCT FROM` for null-safe equality.
  * Dates: `DATE_TRUNC('month', col)`, `col + INTERVAL 7 DAY`, `DATE_DIFF('day', start, end)`, `STRFTIME(col, '%Y-%m-%d')`.
  * Lists and structs are first-class: `[1, 2, 3]`, `{'a': 1}`, `UNNEST(arr)` to expand, `LIST()`/`ARRAY_AGG()` to aggregate.

## 

​

Security Considerations

Evidence executes queries against your MotherDuck database. Take pragmatic steps to protect it against misuse, whether accidental or malicious.

  1. Use a **read-scoped** access token rather than a read-write token where your workflow allows it.
  2. Point Evidence at a database (or share) that exposes only the tables you intend to query.
  3. Rotate the access token on a schedule.

Was this page helpful?

⌘I
