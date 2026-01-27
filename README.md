# dbt Labs | Fivetran | Snowflake SKO FY27 Hands-On Lab

Welcome to the dbt Labs and Fivetran Hands-On Lab for Snowflake SKO FY27! In this workshop, you'll learn to move and activate data with Fivetran, transform it with dbt, and deliver insights in Snowflake.

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
   - **Destination**: `HOL_DATABASE_1` (pre-configured)
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
4. Click on **HOL_DATABASE_1** database
6. Find your schema (e.g., `yourfirstname_yourlastname_higher_education`)
7. Click **Tables**
8. Click on **HED_RECORDS** table
9. Click **Data Preview** tab to view the Fivetran synced data and scroll all the way to the right to see `_fivetran_synced`

---

## Step 2: Transform Data with dbt

### 2.1 Register for dbt Cloud Workshop Account

1. Navigate to the dbt Workshop registration page: [https://workshops.us1.dbt.com/workshop/](https://workshops.us1.dbt.com/workshop/)
2. Fill in the registration form:
   - **First Name**: Your first name
   - **Last Name**: Your last name
   - **Company Email**: Your email address
   - **Workshop Selection**: Select **Snowflake SKO27 Hands-On Lab** from the dropdown
   - **Workshop Passcode**: Enter the passcode from the [lab credentials page](https://fivetran-lab.web.app/lab-credentials.html)
3. Click **Complete Registration** and wait for the success pop-up. It will include generated dbt Platform credentials for the workshop.
4. Please record this crednetials for the reminder of the workshop, you may need them to log into dbt Platform.
5. Click **Complete Registration** - if you are redirect to a login page use the automatically generated credentials above for access. 

### 2.2 Access dbt Platform

1. In dbt Platform, locate the **Project dropdown** on the left-hand side
2. Select **Snowflake SKO (Higher Education)** from the dropdown
3. Click **Studio** to load dbt Platform

### 2.3 Explore dbt Packages Configuration

1. In the **Project Navigator** (left sidebar), locate and open the `packages.yml` file
2. Review the file contents and note how the `snowflake_semantic_view` package is defined
   - This package enables dbt to create Snowflake Semantic Views
3. For detailed information on this package and how it works, see: [https://hub.getdbt.com/Snowflake-Labs/dbt_semantic_view/latest/](https://hub.getdbt.com/Snowflake-Labs/dbt_semantic_view/latest/)

**Understanding Package Management:**
- When adding new packages, run `dbt deps` to install dependencies
- This command creates or updates `package-lock.yml`, which records specific package versions
- The lock file prevents compatibility issues when collaborating with other users

### 2.4 Examine and Run dbt Models

1. In the **Project Navigator**, expand the `models` folder
2. Expand the `HED` subfolder (contains all Higher Education models)
3. Open the `hed_at_risk_students.sql` file
4. Review the code:
   - This file contains the code to generate a Snowflake Semantic View
   - Note how the syntax looks identical to executing it directly in Snowflake
5. Click the **Run +model (Upstream)** button for this model to build the semantic view and its upstream dependencies it in your local Dev schema
6. Wait for the model to complete successfully

### 2.5 Generate Tests with dbt Co-Pilot

1. In the **Project Navigator**, locate and open `vw_hed_retention_risk_analysis.sql`
   - This is an upstream dependency of the Semantic View
2. Click the **Co-Pilot dropdown** in the editor
3. Select **Generate Generic Test** from the dropdown menu
4. Wait for Co-Pilot to process and generate the associated YAML configuration
5. Review the generated test definitions

### 2.6 Save and Commit Changes

1. Locate the **Git Integration button** in the top-left corner of dbt Platform
2. Click the **Git Integration button** to commit and sync your changes
3. Follow the prompts to commit your work
4. Click to open a pull request in GitHub
5. Review the PR (no need to merge - lab changes won't be merged)

**Note:** This step saves your work and demonstrates the git workflow, but we won't be merging changes during the lab.

### 2.7 Run a Production dbt Job

1. In the left-hand menu, navigate to **Orchestration > Jobs**
2. Locate and select the preconfigured **Prod Job** (running in the **Prod Environment**)
3. Click **Settings** in the top right, then click **Edit**
4. Update the **Execution commands**:
   - Find the command that says `dbt build`
   - Change it to: `dbt build --select source:hed+`
   - This will only build models downstream of the Higher Education data source
5. Ensure **Generate docs on run** is checked
6. Click **Save** in the top right to save your changes

### 2.8 Execute the Production Job

1. Navigate back to the **Job Overview** page using the top navigation
2. Click the **Run Now** button to execute the job
3. Wait for the job to complete

---

## Step 3: Analyze Data with Snowflake Cortex

Now that we've created the Snowflake Semantic View, you can interact with it using a preconfigured Cortex Agent in two ways:

**Agent Details:**  
For information about the agent configuration, see: [Snowflake Agent Config Reference](https://github.com/kellykohlleffel/Snowflake-SKO-FY27-HOL/blob/main/reference_docs/snowflake_agent_config.md)

**Access Options:**

1. **Option 1: Snowflake Intelligence** - [ai.snowflake.com](https://ai.snowflake.com)
2. **Option 2: Snowflake Account Direct Access** - Log in to your Snowflake account

**Prerequisites:**
- Use credentials from the [lab credentials page](https://fivetran-lab.web.app/lab-credentials.html) for either access method

### 3.1 Access the Cortex Agent

1. Choose your preferred access method (Snowflake Intelligence or direct Snowflake login)
2. Log in using credentials from the lab credentials page
3. Locate the **HED_STUDENT_SUCCESS_AGENT_LAB** agent
4. You can now ask questions about:
   - Student retention
   - At-risk students
   - Suggested action plans
---

## Need Help?

Ask a lab instructor for assistance.
