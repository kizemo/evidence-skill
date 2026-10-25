---
type: external-doc
title: BigQuery
source: https://docs.evidence.dev/direct-connectors/bigquery
source_hash: 0079edc9dd
crawled_at: 2026-10-15T00:00:00.000Z
tags:
  - crawled
  - docs
  - evidence
---

# BigQuery

> 原文: <https://docs.evidence.dev/direct-connectors/bigquery>

The BigQuery direct connector lets Evidence run queries live against your BigQuery project.

## 

​

Connecting

You’ll create a service account in GCP, give it permission to run BigQuery jobs and read your datasets, then upload its JSON key to Evidence.

The steps below use the `gcloud` and `bq` CLIs, but every step has an equivalent in the [GCP Console](<https://console.cloud.google.com>)

### 

​

Prerequisites

  * The `gcloud` CLI, authenticated as a user with **IAM Admin** and **BigQuery Admin** roles on the target project (or **Owner**).
  * A GCP project that contains the datasets you want to query.

In the snippets below, replace `<project>` with your GCP project ID.

### 

​

1\. Create a service account for Evidence
    
    
    gcloud iam service-accounts create evidence-bq-primary \
      --project=<project> \
      --display-name="Evidence BigQuery primary"
    

### 

​

2\. Grant it BigQuery access

#### 

​

Job user
    
    
    gcloud projects add-iam-policy-binding <project> \
      --member="serviceAccount:evidence-bq-primary@<project>.iam.gserviceaccount.com" \
      --role="roles/bigquery.jobUser"
    

#### 

​

Data viewer
    
    
    gcloud projects add-iam-policy-binding <project> \
      --member="serviceAccount:evidence-bq-primary@<project>.iam.gserviceaccount.com" \
      --role="roles/bigquery.dataViewer"
    

### 

​

3\. Generate a JSON key
    
    
    gcloud iam service-accounts keys create <some/local/path>/evidence-bq-primary.json \
      --iam-account="evidence-bq-primary@<project>.iam.gserviceaccount.com"
    

### 

​

4\. Configure the connector in Evidence

1

Open the connectors page

Go to **Connectors** in the sidebar. In the **Warehouse** card at the top, select **BigQuery**.

2

Upload the JSON key

Click the **Service account JSON** upload area and choose `evidence-bq-primary.json`.

3

Choose datasets

Add the datasets you want to use with Evidence in **Visible datasets** (comma-separated).

4

Test and save

Click **Test Connection**. Once all steps pass, click **Save**.

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

GCP project that owns the BigQuery datasets you want to query.

​

{ client_email: string; private_key: string; project_id?: string }

Inline service-account key JSON.

​

string

Default query location (e.g. US, EU, us-central1).

​

string

Dataset used when queries reference unqualified table names.

#### 

​

Visibility

​

string[]

required

Allowlist of datasets exposed to the editor and schema browser.

#### 

​

Row-level security

​

{ name: string; serviceAccountEmail: string }[]

Service accounts the primary SA can impersonate for RLS. Each must have iam.serviceAccountTokenCreator granted to the primary SA. Default: `[]`.
    
    
    type: bigquery
    project: my-gcp-project
    keyfile: ./evidence-bq-primary.json
    # dataset: analytics
    datasets:
      - analytics
      - reporting
    # roles:
    #   - name: customer_a
    #     serviceAccountEmail: customer-a@my-gcp-project.iam.gserviceaccount.com
    

#### 

​

Credentials

 _Provide exactly one of`keyfile_json`, `keyfile`._

​

string

required

GCP project that owns the BigQuery datasets you want to query.

​

{ client_email: string; private_key: string; project_id?: string }

Inline service-account key JSON.

​

string

Path to a service-account key JSON file, resolved relative to connection.yaml.

​

string

Default query location (e.g. US, EU, us-central1).

​

string

Dataset used when queries reference unqualified table names.

#### 

​

Visibility

​

string[]

required

Allowlist of datasets exposed to the editor and schema browser.

#### 

​

Row-level security

​

{ name: string; serviceAccountEmail: string }[]

Service accounts the primary SA can impersonate for RLS. Each must have iam.serviceAccountTokenCreator granted to the primary SA. Default: `[]`.

## 

​

Row-Level Security

Evidence supports row-level security on the BigQuery direct connector via **service account impersonation**.

### 

​

How it works

You create **one BigQuery service account per RLS role** (per customer, per tenant, per region — whatever your access boundary is) and attach BigQuery row-access policies to those service accounts. Evidence’s primary SA is granted permission to impersonate each role SA, and at query time chooses which service account to use based on the user’s assigned role in Evidence.

### 

​

Setup

The example below creates two roles, `customer_a` and `customer_b`, with filters on a `client_id` column. Substitute your own role names, table, and filter logic.

1

Create a role service account per RLS role
    
    
    gcloud iam service-accounts create customer-a \
      --project=<project> \
      --display-name="Evidence RLS role: customer_a"
    

Repeat for each role.

2

Let the primary SA impersonate each role SA
    
    
    gcloud iam service-accounts add-iam-policy-binding \
      customer-a@<project>.iam.gserviceaccount.com \
      --member="serviceAccount:evidence-bq-primary@<project>.iam.gserviceaccount.com" \
      --role="roles/iam.serviceAccountTokenCreator"
    

This grants the impersonation right _on the role SA as a resource_ — the primary SA can mint tokens for this specific role SA only, not project-wide.Repeat for each role.

3

Grant each role SA BigQuery access
    
    
    # jobUser
    gcloud projects add-iam-policy-binding <project> \
      --member="serviceAccount:customer-a@<project>.iam.gserviceaccount.com" \
      --role="roles/bigquery.jobUser" \
      --condition=None
    
    
    
    # dataViewer
    gcloud projects add-iam-policy-binding <project> \
      --member="serviceAccount:customer-a@<project>.iam.gserviceaccount.com" \
      --role="roles/bigquery.dataViewer" \
      --condition=None
    

Repeat for each role.

4

Attach row-access policies
    
    
    CREATE OR REPLACE ROW ACCESS POLICY customer_a_filter
    ON `<project>.<dataset>.<table>`
    GRANT TO ('serviceAccount:customer-a@<project>.iam.gserviceaccount.com')
    FILTER USING (client_id = 'customer_a');
    

Repeat per role. The `FILTER USING` clause is plain SQL — you can use any column, comparison, or join.

5

Decide on a fallback for unassigned users

Once any policy exists on a table, **every** principal querying it needs a matching policy or they see zero rows — including Evidence’s primary SA.To preserve the existing behaviour for users without a role assignment (they see all rows), add a fallback policy:
    
    
    CREATE OR REPLACE ROW ACCESS POLICY primary_fallback
    ON `<project>.<dataset>.<table>`
    GRANT TO ('serviceAccount:evidence-bq-primary@<project>.iam.gserviceaccount.com')
    FILTER USING (TRUE);
    

Skip this step if you’d rather unassigned users see nothing. That’s the more secure default but you’ll need every viewer to have a role assigned.

6

Register the roles in Evidence

In **Settings → Access Rules → Roles** , add one row per role:

Service account| Role label  
---|---  
`customer-a@<project>.iam.gserviceaccount.com`| `customer_a`  
`customer-b@<project>.iam.gserviceaccount.com`| `customer_b`  
  
7

Assign roles to users

In **Settings → Access Rules → Roles** and assign a role to each user.

## 

​

Security Considerations

Evidence executes queries against your BigQuery instance. You should take pragmatic steps to protect your instance against misuse, whether accidental or malicious.

  1. Scope data viewer to specific datasets
  2. Keep the service account scoped
  3. Cap cost per query
  4. Audit logs and alerts

### 

​

1\. Scope data viewer to specific datasets

The setup above grants `bigquery.dataViewer` project-wide, which also gives read access to every other dataset in the project and to `INFORMATION_SCHEMA`. To narrow this, grant `dataViewer` per-dataset instead:
    
    
    bq add-iam-policy-binding \
      --member="serviceAccount:evidence-bq-primary@<project>.iam.gserviceaccount.com" \
      --role="roles/bigquery.dataViewer" \
      <project>:<dataset>
    

Then remove the project-wide grant:
    
    
    gcloud projects remove-iam-policy-binding <project> \
      --member="serviceAccount:evidence-bq-primary@<project>.iam.gserviceaccount.com" \
      --role="roles/bigquery.dataViewer"
    

Repeat for each role SA you’ve set up for RLS.

### 

​

2\. Keep the service account scoped

The setup grants only `bigquery.jobUser` and `bigquery.dataViewer`. Don’t add more — broader roles like `bigquery.user`, `bigquery.admin`, or `bigquery.connectionUser` enable behaviour Evidence doesn’t need and unlock features like `EXTERNAL_QUERY` and remote functions. Audit the SA’s IAM bindings with:
    
    
    gcloud projects get-iam-policy <project> \
      --flatten="bindings[].members" \
      --filter="bindings.members:evidence-bq-primary@<project>.iam.gserviceaccount.com" \
      --format="table(bindings.role)"
    

### 

​

3\. Cap cost per query

Set a project-level ceiling on bytes billed per job so a runaway query can’t drain budget. You can also set [per-query](<https://cloud.google.com/bigquery/docs/best-practices-costs#limit-query-costs>) `maximum_bytes_billed` on the query itself or [custom quotas](<https://cloud.google.com/bigquery/docs/custom-quotas>) per user.

### 

​

4\. Audit logs and alerts

Enable [BigQuery audit logs](<https://cloud.google.com/bigquery/docs/reference/auditlogs>) and route them to a sink you monitor. E.g. alert on:

  * `EXPORT DATA` jobs (data leaving the warehouse)
  * DDL or DCL statements run by Evidence’s SAs (they shouldn’t be running any)
  * Unusually large scans by a single SA

Was this page helpful?

⌘I
