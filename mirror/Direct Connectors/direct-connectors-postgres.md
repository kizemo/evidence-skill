---
type: external-doc
title: Postgres
source: https://docs.evidence.dev/direct-connectors/postgres
source_hash: 4be9b8911e
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Postgres

> 原文: <https://docs.evidence.dev/direct-connectors/postgres>

The Postgres direct connector lets Evidence run queries live against your own Postgres database. Because it speaks the standard Postgres wire protocol, it also works with **Amazon RDS / Aurora** , **Supabase** , **Neon** , and **Timescale**.

## 

​

Connecting

You need the database **host** , **port** , **database** name, a **user** , and its **password** , plus an appropriate **SSL mode** for managed providers.

1

Gather connection details

From your provider, note the host (e.g. `mydb.abc123.us-east-1.rds.amazonaws.com`), port, database name, and a user. Enter the **hostname only** — not a `postgres://` connection URL.

Direct connections and **session-mode** poolers work fully. A **transaction-mode** pooler (e.g. Supabase’s `6543`, PgBouncer in transaction mode) doesn’t keep session state across pooled queries, so a non-default `search_path` and the statement timeout may not apply there. If your schema isn’t `public`, prefer the **direct endpoint** or fully-qualify table names as `schema.table`.

2

Choose an SSL mode

The default is **Verify full** — TLS with full server-certificate and hostname verification, the secure choice. Providers with publicly-trusted certificates (e.g. Neon) work out of the box. For a **private CA** — notably **Amazon RDS** (its certificates are signed by the [RDS root CA](<https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.SSL.html>), which isn’t in the default trust store) — keep **Verify full** and supply the CA bundle. Lower modes exist (`verify-ca` skips the hostname check; `require` encrypts but does **not** verify the certificate, leaving you exposed to an active man-in-the-middle) but aren’t recommended. Mutual TLS is supported via an optional client certificate and key.

3

Configure the connector in Evidence

Go to **Connectors** in the sidebar, choose **Postgres** in the **Warehouse** card, and fill in host, port, database, user, and password. Set the SSL mode, and (optionally) paste a CA / client certificate under **TLS certificates**.

4

(Optional) Restrict the schema browser

Add schema names to **Schemas** to limit which ones appear in the schema browser. Leave empty to show just the connection’s default schema (`public`).

5

Test and save

Click **Test Connection**. Once it passes, click **Save**.

### 

​

Grant least-privilege access

Create a dedicated read-only role for Evidence rather than reusing an admin/owner login:
    
    
    CREATE ROLE evidence LOGIN PASSWORD '<strong-password>';
    GRANT CONNECT ON DATABASE analytics TO evidence;
    GRANT USAGE ON SCHEMA public TO evidence;
    GRANT SELECT ON ALL TABLES IN SCHEMA public TO evidence;
    -- Include tables created later:
    ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO evidence;
    

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

Postgres server hostname, e.g. db.example.com or mydb.abc123.us-east-1.rds.amazonaws.com.

​

number

Postgres port. Defaults to 5432; Supabase poolers use 6543. Default: `5432`.

​

string

required

Postgres user to connect as.

​

string

required

Password for the Postgres user.

​

string

required

Database to connect to.

​

`disable` | `require` | `verify-ca` | `verify-full`

verify-full (default) = encrypt and verify the server certificate + hostname — the secure choice. verify-ca = verify the cert but not the hostname. require = encrypt WITHOUT verifying the cert (vulnerable to an active man-in-the-middle — avoid). disable = no TLS. For a private CA (e.g. Amazon RDS), keep verify-full and supply the CA bundle in ssl_ca. Default: `"verify-full"`.

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

Schema used when queries reference unqualified table names (sets search_path). Default: `"public"`.

#### 

​

Visibility

​

string[]

Allowlist of schemas exposed to the editor and schema browser. Defaults to just the default schema (the `schema` field) when empty; list additional schemas to include them. Default: `[]`.
    
    
    type: postgres
    host: mydb.abc123.us-east-1.rds.amazonaws.com
    # port: 5432
    user: evidence
    password: <your-password>
    database: analytics
    # sslmode: verify-full
    # ssl_ca_path: ./certs/rds-global-bundle.pem # PEM CA bundle for verify-ca / verify-full
    # ssl_cert_path: ./certs/client.crt # mutual TLS client certificate
    # ssl_key_path: ./certs/client.key # mutual TLS client key
    # schema: public
    # schemas: [public, analytics]
    

#### 

​

Credentials

​

string

required

Postgres server hostname, e.g. db.example.com or mydb.abc123.us-east-1.rds.amazonaws.com.

​

number

Postgres port. Defaults to 5432; Supabase poolers use 6543. Default: `5432`.

​

string

required

Postgres user to connect as.

​

string

required

Password for the Postgres user.

​

string

required

Database to connect to.

​

`disable` | `require` | `verify-ca` | `verify-full`

verify-full (default) = encrypt and verify the server certificate + hostname — the secure choice. verify-ca = verify the cert but not the hostname. require = encrypt WITHOUT verifying the cert (vulnerable to an active man-in-the-middle — avoid). disable = no TLS. For a private CA (e.g. Amazon RDS), keep verify-full and supply the CA bundle in ssl_ca. Default: `"verify-full"`.

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

Schema used when queries reference unqualified table names (sets search_path). Default: `"public"`.

#### 

​

Visibility

​

string[]

Allowlist of schemas exposed to the editor and schema browser. Defaults to just the default schema (the `schema` field) when empty; list additional schemas to include them. Default: `[]`.

## 

​

Row-Level Security

Queries Evidence issues run as the **connection role** (the user you configured above). Postgres [row-level security policies](<https://www.postgresql.org/docs/current/ddl-rowsecurity.html>) on your tables apply to that role, so you can scope what Evidence can read at the database level.

Per-Evidence-viewer RLS (mapping the current viewer’s identity into the query, as the Snowflake and Databricks connectors do via session variables) is not yet available for Postgres — every query runs under the single connection role. If you need per-viewer row filtering today, enforce it in a view or restrict the connection role’s grants.

## 

​

Security Considerations

Evidence executes queries against your Postgres database. Take pragmatic steps to protect it against misuse, whether accidental or malicious.

  1. Use a dedicated role with `SELECT`-only grants — never the database owner or a superuser.
  2. Scope grants to the specific schemas Evidence should see.
  3. Require TLS (`verify-full` where possible) so credentials and data aren’t sent in the clear.
  4. Restrict network access to Evidence’s [egress IPs](</evidence-warehouse#ip-whitelisting>) plus your team’s networks (e.g. an RDS security group or Supabase/Neon IP allowlist).
  5. Rotate the connection password regularly.

Was this page helpful?

⌘I
