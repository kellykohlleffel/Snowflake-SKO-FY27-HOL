# dbt Labs | Fivetran | Snowflake SKO FY27 Hands-On Lab

Welcome to the dbt Labs and Fivetran Hands-On Lab for Snowflake SKO FY27! In this workshop, you'll learn to move and activate data with Fivetran, transform it with dbt, and deliver insights in Snowflake.

This hands-on lab is a **full-stack, end-to-end walkthrough of how modern data teams move from raw ingestion to AI-powered outcomes in Snowflake** without DIY pipelines, brittle logic, or disconnected tools.

Snowflake SEs will **build a Snowflake AI-ready data flow in real time**: ingesting source data with Fivetran (PostgreSQL), transforming it into a **Cortex-ready semantic layer with dbt**, and activating it through **Snowflake Cortex Agents and Snowflake Intelligence**. Along the way, you'll see how this architecture delivers simplicity, speed to business outcomes, automated reliability, semantic consistency, and why this approach wins in the agentic era.

The lab then shifts from how to build to **what’s new with Fivetran**:

- How Fivetran is **activating data natively in Snowflake, with Fivetran Census**
- Building **custom connectors** with the Fivetran Connector SDK
- Moving toward a **single intelligent UI with MCP**

We close with **what’s new from dbt Labs**, including the **Fusion Engine**, **emerging patterns for dbt + Snowflake AI** (Semantics, OSI, dbt MCP Server), and **how dbt fits directly within Snowflake**.

---

<div style="background-color: #fff3cd; padding: 12px 16px; border-left: 6px solid #ffcc00; margin-bottom: 20px; margin-top: 10px;">
  <strong>⚠️ IMPORTANT:</strong><br>
  Before starting the lab, register for your Fivetran account at:<br>
  <strong><a href="https://fivetran-lab.web.app/">https://fivetran-lab.web.app/</a></strong>
</div>

---

## Prerequisites

- Web browser
- Valid email address (Snowflake, Fivetran, or dbt Labs)

## Getting Started

1. Register at [https://fivetran-lab.web.app/](https://fivetran-lab.web.app/)
2. Check your email for your Fivetran invitation
3. Accept the invitation and set your password
4. Get lab credentials: [https://fivetran-lab.web.app/lab-credentials.html](https://fivetran-lab.web.app/lab-credentials.html) (passcode provided by instructor)
5. Log in to Fivetran and get ready to build!

---

## Step 1: Create Fivetran Connector to Snowflake

### 1.1 Configure PostgreSQL Source Connector
1. In Fivetran, click **+ Connector**
2. Search for and select **Google Cloud PostgreSQL**
3. Configure the connector:
   - **Destination**: `Snowflake_SKO_HOL_27_dbt` (pre-configured)
   - **Snowflake Destination Virtual Warehouse**: Keep default
   - **Destination schema prefix**: `yourfirstname_yourlastname` (lowercase, underscores only)
   - **Destination schema names**: Fivetran naming
   - **Host**: Use credentials from lab credentials page
   - **Port**: `5432`
   - **User**: Use credentials from lab credentials page
   - **Password**: Use credentials from lab credentials page
   - **Database**: `industry` (case-sensitive)
   - **Authentication Method**: Connect with a username and password
   - **Connection method**: Connect directly
   - **Update Method**: Query-based
4. Click **Save & Test**

### 1.2 Select Data to Sync
1. The `higher_education` schema and `hed_records` table will be pre-selected
2. Click **Continue**

### 1.3 Handle Schema Changes
1. Select **Allow all** (default)
2. Click **Continue**

### 1.4 Start Initial Sync
1. On the connector Status page, click **Start Initial Sync**

### 1.5 Verify Data in Snowflake (While Sync Runs)
1. Navigate to Snowflake Lab Account using credentials from lab credentials page
2. Log in with provided credentials
3. In Snowflake UI, click **Catalog** in the left navigation
4. Click on **RAW** database
6. Find your schema (e.g., `yourfirstname_yourlastname_higher_education`)
7. Click **Tables**
8. Click on **HED_RECORDS** table
9. Click **Data Preview** tab to view the Fivetran synced data and scroll all the way to the right to see `_fivetran_synced`

---

## Step 2: Transform Data with dbt

*Coming soon - instructor will guide this section*

---

## Need Help?

Ask a lab instructor for assistance.