---
type: external-doc
title: Snowflake
source: https://docs.evidence.dev/direct-connectors/snowflake
source_hash: 9c9f68526e
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# Snowflake

> 原文: <https://docs.evidence.dev/direct-connectors/snowflake>

The Snowflake direct connector lets Evidence run queries live against your Snowflake account.

## 

​

Connecting

You’ll create a dedicated user in Snowflake, grant it access to the warehouse and data you want Evidence to read, then enter the credentials in Evidence.

### 

​

Prerequisites

  * A Snowflake account with a warehouse and at least one database you want Evidence to query.
  * A user with `SECURITYADMIN` and `SYSADMIN` (or `ACCOUNTADMIN`) to run the setup SQL.

### 

​

1\. Create a role and user for Evidence

If you already have a read-only user suitable for BI workloads, skip to step 2. Otherwise, run the following, replacing `<warehouse>`, `<database>`, `<schema>`, and `<strong-password>` with your own values:

SQL to create user
    
    
    USE ROLE SECURITYADMIN;
    
    CREATE ROLE EVIDENCE_READER;
    
    CREATE USER EVIDENCE_USER
      PASSWORD = '<strong-password>' -- OR RSA_PUBLIC_KEY = '<your-public-key>'
      DEFAULT_ROLE = EVIDENCE_READER
      DEFAULT_WAREHOUSE = <warehouse>
      MUST_CHANGE_PASSWORD = FALSE;
    
    GRANT ROLE EVIDENCE_READER TO USER EVIDENCE_USER;
    
    USE ROLE SYSADMIN;
    
    GRANT USAGE ON WAREHOUSE <warehouse> TO ROLE EVIDENCE_READER;
    GRANT USAGE ON DATABASE <database> TO ROLE EVIDENCE_READER;
    GRANT USAGE ON SCHEMA <database>.<schema> TO ROLE EVIDENCE_READER;
    
    -- Grants access to all existing tables and views in <database>.<schema>.
    -- You can also enumerate specific tables/views instead.
    GRANT SELECT ON ALL TABLES IN SCHEMA <database>.<schema> TO ROLE EVIDENCE_READER;
    GRANT SELECT ON ALL VIEWS IN SCHEMA <database>.<schema> TO ROLE EVIDENCE_READER;
    
    -- (Optional) Grants access to tables and views added to <database>.<schema> in the future.
    GRANT SELECT ON FUTURE TABLES IN SCHEMA <database>.<schema> TO ROLE EVIDENCE_READER;
    GRANT SELECT ON FUTURE VIEWS IN SCHEMA <database>.<schema> TO ROLE EVIDENCE_READER;
    

For key-pair authentication, replace the `PASSWORD = '<strong-password>'` line with `RSA_PUBLIC_KEY = '<your-public-key>'`. See the [Snowflake docs on key-pair authentication](<https://docs.snowflake.com/en/user-guide/key-pair-auth>) for how to generate the key.

### 

​

2\. Configure the connector in Evidence

1

Open the connectors page

Go to **Connectors** in the sidebar. In the **Warehouse** card at the top, select **Snowflake**.

2

Enter connection details

Fill in the form with the credentials of the user you created.

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

Snowflake account identifier, e.g. xy12345.us-east-1.

​

string

required

Snowflake login name.

​

string

Password for password auth.

​

string

PEM-encoded RSA private key for key-pair auth.

​

string

Passphrase for the private key, if it is encrypted.

​

string

required

Compute warehouse to use for queries.

​

string

required

Default database.

​

string

Role applied per session.

#### 

​

Visibility

​

{ production: string; devSchemas: string[] }

Production schema, plus optional developer schemas that act as preview environments.

#### 

​

Row-level security

​

{ snowflakeVariable: string; evidenceVariable: `user.email` | `user.id` | `user.name` | `organization.id` }[]

Evidence identity → Snowflake session variable, ALTERed per query for RLS. Default: `[]`.
    
    
    type: snowflake
    account: xy12345.us-east-1
    user: evidence_user
    private_key_path: ./snowflake-key.pem
    warehouse: COMPUTE_WH
    database: ANALYTICS
    # role: EVIDENCE_READER
    # schema: { production: ANALYTICS, devSchemas: [DEV_ALICE] }
    # session_variables:
    #   - snowflakeVariable: APP_USER_EMAIL
    #     evidenceVariable: user.email
    

#### 

​

Credentials

 _Provide exactly one of`password`, `private_key`, `private_key_path`._

​

string

required

Snowflake account identifier, e.g. xy12345.us-east-1.

​

string

required

Snowflake login name.

​

string

Password for password auth.

​

string

PEM-encoded RSA private key for key-pair auth.

​

string

Path to PEM-encoded RSA private key, resolved relative to connection.yaml.

​

string

Passphrase for the private key, if it is encrypted.

​

string

required

Compute warehouse to use for queries.

​

string

required

Default database.

​

string

Role applied per session.

#### 

​

Visibility

​

{ production: string; devSchemas: string[] }

Production schema, plus optional developer schemas that act as preview environments.

#### 

​

Row-level security

​

{ snowflakeVariable: string; evidenceVariable: `user.email` | `user.id` | `user.name` | `organization.id` }[]

Evidence identity → Snowflake session variable, ALTERed per query for RLS. Default: `[]`.

## 

​

Row-Level Security

Documentation for RLS is not yet available for Snowflake. Please contact support to set up RLS with the Snowflake direct connector.

Was this page helpful?

⌘I
