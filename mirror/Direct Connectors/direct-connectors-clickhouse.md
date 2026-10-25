---
type: external-doc
title: ClickHouse
source: https://docs.evidence.dev/direct-connectors/clickhouse
source_hash: aaf34cd636
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# ClickHouse

> 原文: <https://docs.evidence.dev/direct-connectors/clickhouse>

The ClickHouse direct connector lets Evidence run queries live against your own ClickHouse instance — ClickHouse Cloud, self-hosted, or any other deployment exposing the HTTP(S) interface.

## 

​

Connecting

You only need three things: the **host** , a **username** , and a **password** (or a JWT access token for ClickHouse Cloud).

1

Find your connection details

**ClickHouse Cloud:** open your service in the Cloud console and click **Connect**. Copy the **host** (e.g. `abc123.us-east-1.aws.clickhouse.cloud`) and the **password** for the `default` user. Use port `8443` (HTTPS).**Self-hosted:** use the host and port configured for your instance. The HTTP interface defaults to port `8123`, or `8443` when TLS is enabled.

2

Configure the connector in Evidence

Go to **Connectors** in the sidebar, choose **ClickHouse** in the **Warehouse** card, and fill in the form. Leave **Use TLS** on for Cloud; turn it off only for a plain-HTTP self-hosted instance.

3

(Optional) Restrict the schema browser

Add database names to **Databases** to limit which ones appear in the schema browser. Leave empty to show just the connection’s default database.

4

Test and save

Click **Test Connection**. Once it passes, click **Save**.

### 

​

Create a dedicated read-only user

The setup above will work with the `default` user, but for production you should give Evidence its own credentials with `SELECT`-only access. In the SQL console (or via `clickhouse-client`):
    
    
    CREATE USER evidence IDENTIFIED WITH sha256_password BY '<strong-password>';
    
    CREATE ROLE evidence_reader;
    GRANT SELECT ON <database>.* TO evidence_reader;
    GRANT evidence_reader TO evidence;
    ALTER USER evidence DEFAULT ROLE evidence_reader;
    

For ClickHouse Cloud, run this from the **SQL console** in the Cloud UI.

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

ClickHouse HTTP(S) interface hostname, e.g. abc123.us-east-1.aws.clickhouse.cloud.

​

number

HTTP(S) interface port — 8443 for TLS (the ClickHouse Cloud default), 8123 for plain HTTP. Default: `8443`.

​

boolean

Connect over HTTPS. Leave on for ClickHouse Cloud; turn off for a plain-HTTP self-hosted instance. Default: `true`.

​

string

ClickHouse user to connect as. Default: `"default"`.

​

string

Password for the ClickHouse user.

​

string

JWT access token. Supported by ClickHouse Cloud only; mutually exclusive with password.

​

string

Default database for unqualified table references. Default: `"default"`.

#### 

​

Visibility

​

string[]

Allowlist of databases exposed to the editor and schema browser. Defaults to the connection database when empty. Default: `[]`.
    
    
    type: clickhouse
    host: abc123.us-east-1.aws.clickhouse.cloud
    # port: 8443
    # secure: true
    # username: default
    # database: default
    # databases: [analytics, reporting]
    

#### 

​

Credentials

 _Provide exactly one of`password`, `access_token`._

​

string

required

ClickHouse HTTP(S) interface hostname, e.g. abc123.us-east-1.aws.clickhouse.cloud.

​

number

HTTP(S) interface port — 8443 for TLS (the ClickHouse Cloud default), 8123 for plain HTTP. Default: `8443`.

​

boolean

Connect over HTTPS. Leave on for ClickHouse Cloud; turn off for a plain-HTTP self-hosted instance. Default: `true`.

​

string

ClickHouse user to connect as. Default: `"default"`.

​

string

Password for the ClickHouse user.

​

string

JWT access token. Supported by ClickHouse Cloud only; mutually exclusive with password.

​

string

Default database for unqualified table references. Default: `"default"`.

#### 

​

Visibility

​

string[]

Allowlist of databases exposed to the editor and schema browser. Defaults to the connection database when empty. Default: `[]`.

## 

​

Row-Level Security

Any [row policies](<https://clickhouse.com/docs/sql-reference/statements/create/row-policy>) defined in your ClickHouse instance apply to queries issued by Evidence. Policies evaluate against the connection-level user — the credentials configured for the warehouse — rather than the individual Evidence viewer.

**Per-viewer row-level security is planned for an upcoming release.** Evidence will propagate each viewer’s identity to ClickHouse, allowing row policies to filter results on a per-user basis. To discuss your requirements or request early access, contact [support@evidence.dev](<mailto:support@evidence.dev>).

## 

​

Security Considerations

Evidence executes queries against your ClickHouse instance. Take pragmatic steps to protect it against misuse, whether accidental or malicious.

  1. Use a dedicated user with `SELECT`-only grants (see above). Avoid reusing the `default` account in production.
  2. Scope `GRANT SELECT` to the specific databases (or tables) Evidence should see — not `*.*`.
  3. Cap query cost via a [settings profile](<https://clickhouse.com/docs/operations/settings/settings-profiles>) on the user — `max_execution_time`, `max_memory_usage`, `max_rows_to_read` are a good baseline. Avoid `readonly = 1`; it blocks the per-query setting Evidence uses for number formatting. Use `readonly = 2` if you want a read-only enforcement at the profile level (it still allows the client to override session settings).
  4. For ClickHouse Cloud, restrict the service’s [IP access list](<https://clickhouse.com/docs/cloud/security/setting-ip-filters>) to Evidence’s [egress IPs](</evidence-warehouse#ip-whitelisting>) plus your team’s networks.
  5. Monitor the [query log](<https://clickhouse.com/docs/operations/system-tables/query_log>) and alert on non-`SELECT` statements issued by the Evidence user (there shouldn’t be any).

Was this page helpful?

⌘I
