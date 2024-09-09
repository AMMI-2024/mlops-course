# Day 3 Lab: Data Engineering and ETL Pipelines

## Table of Contents

- [Day 3 Lab: Data Engineering and ETL Pipelines](#day-3-lab-data-engineering-and-etl-pipelines)
  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
  - [Theoretical Concepts](#theoretical-concepts)
    - [What is ETL (Extract, Transform, Load)?](#what-is-etl-extract-transform-load)
    - [What is Fivetran?](#what-is-fivetran)
    - [What is dbt (Data Build Tool)?](#what-is-dbt-data-build-tool)
    - [Why Combine Fivetran and dbt?](#why-combine-fivetran-and-dbt)
    - [What is Data Lineage in dbt?](#what-is-data-lineage-in-dbt)
  - [Lab Instructions](#lab-instructions)
    - [Part 1: Setting up the Extract-Load process](#part-1-setting-up-the-extract-load-process)
      - [Task 1: Exploring Fivetran Destinations (Registering BigQuery)](#task-1-exploring-fivetran-destinations-registering-bigquery)
      - [Task 2: Exploring Fivetran Connections](#task-2-exploring-fivetran-connections)
        - [2.1 Registering Azure Blob Storage](#21-registering-azure-blob-storage)
        - [2.2 Registering Google Drive as a Source](#22-registering-google-drive-as-a-source)
      - [Task 3: Sync Connections to BigQuery](#task-3-sync-connections-to-bigquery)
    - [Part 2: Setting up the Transform process](#part-2-setting-up-the-transform-process)
      - [Task 1: Creating a dbt Cloud Account and Connecting to BigQuery](#task-1-creating-a-dbt-cloud-account-and-connecting-to-bigquery)
      - [Task 2: Exploring dbt Cloud through Managed Repo](#task-2-exploring-dbt-cloud-through-managed-repo)
      - [Task 3: Running Jobs in Environments](#task-3-running-jobs-in-environments)
      - [Task 4: Freshness Check](#task-4-freshness-check)
      - [Task 5: Connecting Fivetran to dbt Cloud](#task-5-connecting-fivetran-to-dbt-cloud)
    - [Part 3: Add New Data and Re-Sync](#part-3-add-new-data-and-re-sync)
      - [Task 1: Re-Sync New Data in Fivetran and Re-Run dbt Job](#task-1-re-sync-new-data-in-fivetran-and-re-run-dbt-job)
  - [Conclusion](#conclusion)
  - [Useful Links](#useful-links)

## Overview

In this lab, we will explore how to build a scalable data pipeline using Fivetran and dbt Cloud. You will extract data from different sources, load it into BigQuery, and then transform that data using dbt Cloud. This lab will guide you step by step through:

- Extracting and loading data using Fivetran
- Setting up dbt Cloud for data transformation
- Automating data sync and transformations using Fivetran and dbt Cloud

---

## Theoretical Concepts

### What is ETL (Extract, Transform, Load)?

ETL stands for **Extract, Transform, Load**. It's a process used to collect data from various sources, transform the data into a format suitable for analysis, and then load it into a database or data warehouse.

1. **Extract**: Retrieve data from multiple sources, such as databases, APIs, or files.
2. **Transform**: Clean and preprocess the data by performing operations like filtering, aggregating, or reshaping.
3. **Load**: Store the transformed data in a destination system such as a data warehouse (e.g., BigQuery, Snowflake).

In modern architectures, the process may also follow the **ELT (Extract, Load, Transform)** approach, where data is extracted and loaded into the destination first, and transformations happen within the destination (e.g., with tools like dbt).

### What is Fivetran?

**Fivetran** is a fully managed data pipeline platform that automates the process of moving data from different sources (e.g., databases, SaaS apps, files) to a destination like BigQuery or Snowflake. Fivetran provides a range of connectors to seamlessly extract and load data into your data warehouse.

- **Key Features**:
  - **Source Connectors**: Fivetran provides connectors for various sources like databases, cloud storage, APIs, and more.
  - **Destination Connectors**: You can sync data to destinations like Google BigQuery, Snowflake, Redshift, and more.
  - **Automatic Syncing**: Fivetran schedules and runs data syncing automatically.

### What is dbt (Data Build Tool)?

**dbt** is a transformation tool that allows data teams to transform data in their warehouse by writing SQL queries. dbt helps in managing the transformation layer of your data pipeline.

- **Key Features**:
  - **Data Transformation**: Write SQL transformations to clean and process raw data into analytical tables.
  - **Testing and Documentation**: Ensure data quality with built-in tests and documentation.
  - **Orchestration**: dbt can run transformation jobs on a schedule or in response to triggers (like a data sync).
  
  dbt Cloud provides a managed environment where you can develop, schedule, and run dbt jobs without managing the underlying infrastructure.

### Why Combine Fivetran and dbt?

Fivetran and dbt are complementary tools:

- **Fivetran** handles the **E**xtract and **L**oad steps by moving raw data from sources into your data warehouse (e.g., BigQuery).
- **dbt** handles the **T**ransformation step, allowing you to clean and structure the data for downstream analytics and machine learning.

### What is Data Lineage in dbt?

**Data lineage** is the ability to track how data flows through the entire data pipeline, from source to transformation to final destination. dbt automatically creates a visual representation of your data lineage, allowing you to understand how your models are constructed, what data is used, and how it transforms over time.

---

## Lab Instructions

### Part 1: Setting up the Extract-Load process

#### Task 1: Exploring Fivetran Destinations ([Registering BigQuery](https://fivetran.com/docs/destinations/bigquery/setup-guide))

1. **Create Fivetran Account**:
   - Go to [Fivetran](https://www.fivetran.com/) and sign up for a free account.
   - Once logged in, navigate to the **Destinations** tab.

2. **Register BigQuery as Destination**:
   - Click on **Add Destination**.
   - Select **Google BigQuery** as the destination.
   - Fill in the necessary details:
     - **Project ID**: (Your GCP Project ID).
     - **Dataset**: Name the dataset where your data will be synced.
     - **Location**: Choose `US` as the dataset location.
   - Complete the authentication process using your Google account.

#### Task 2: Exploring Fivetran Connections

##### 2.1 [Registering Azure Blob Storage](https://fivetran.com/docs/connectors/files/azure-blob-storage/setup-guide)

1. **Navigate to Connections**:
   - Click on **Add Connector**.
   - Select **Azure Blob Storage** as the source.
   - Choose destination where to sync.

2. **Set up the Blob Storage Data Source**:
   - Configure the Blob storage source: destination schema, table, container name, string...
   - Set up the sync schedule (e.g., Manual or hourly).

---

##### 2.2 [Registering Google Drive as a Source](https://fivetran.com/docs/connectors/files/one-drive/setup-guide)

1. **Navigate to Connections**:
   - Click on **Add Connector**.
   - Select **Google Drive** as the source.
   - Choose destination where to sync.
  
2. **Set up the Google Drive Source**:
   - Authenticate using your Google account and grant Fivetran the necessary permissions.
   - Select a specific folder to sync from Google Drive.
   - Set up the sync schedule (e.g., Manual or hourly).

---

#### Task 3: Sync Connections to BigQuery

1. **Sync Azure Blob Storage**:
   - Navigate to the Azure Blob Storage connector you created.
   - Click on **Sync Now** to trigger a sync between Azure Blob and BigQuery.

2. **Sync Google Drive Data**:
   - Similarly, navigate to the Google Drive connection and click **Sync Now**.
   - The data from both sources will be available in BigQuery after the sync completes.

---

### Part 2: Setting up the Transform process

#### Task 1: Creating a dbt Cloud Account and Connecting to BigQuery

1. **Sign up for dbt Cloud**:
   - Go to [dbt Cloud](https://cloud.getdbt.com/) and create an account.
   - Once registered, click **Start Developing** and create a new project.

2. **[Connect to BigQuery](https://docs.getdbt.com/guides/bigquery?step=5)**:
   - In the project settings, connect dbt Cloud to your BigQuery project by selecting **BigQuery** as the warehouse.
   - [Add the required credentials (Service Account JSON file from GCP)](https://docs.getdbt.com/guides/bigquery?step=4).
   - Test the connection to ensure dbt Cloud can access BigQuery.

#### Task 2: [Exploring dbt Cloud through Managed Repo](https://docs.getdbt.com/guides/bigquery?step=6)

1. **[Create Sources](https://docs.getdbt.com/docs/build/sources)**:
   - In the dbt Cloud IDE, create sources for the synced data (Azure Blob and Google Drive).
   - Write SQL queries to define the source tables for each dataset.

2. **[Create Models](https://docs.getdbt.com/guides/bigquery?step=11)**:
   - Create new models in the **models** folder.
   - Join the data from Blob Storage and Google Drive using SQL queries.

3. **[Add Tests](https://docs.getdbt.com/guides/bigquery?step=12)**:
   - Create tests for the schemas you defined, such as checking for unique values or null values in key fields.

4. **[Data Lineage](https://docs.getdbt.com/terms/data-lineage)**:
   - Examine the data lineage automatically generated by dbt, which shows how the data flows from sources to models.

5. **Explore Results in BigQuery**:
   - Once the models are created and transformations are complete, you can query the resulting tables in BigQuery.

#### Task 3: Running Jobs in Environments

1. **[Create a Deployment Environment](https://docs.getdbt.com/guides/bigquery?step=15)**:
   - Go to the **Environments** tab in dbt Cloud and click **New Environment**.
   - Name the environment "Production" and set the target to BigQuery.

2. **Create Jobs**:
   - Create a new job that runs `dbt run` to apply transformations.
   - Add `dbt test` to the job to test the data after the transformation.

3. **Run dbt Job**:
   - Trigger the `dbt run` job to transform the data.
   - Once the run completes, trigger `dbt test` to validate the transformed data.

#### Task 4: Freshness Check

1. **[Freshness Check](https://docs.getdbt.com/docs/deploy/source-freshness)**:
   - In dbt, set up a **freshness check** on the data source to ensure the data is current.
   - Demonstrate how to schedule a freshness check on dbt Cloud.

#### Task 5: Connecting Fivetran to dbt Cloud

1. **[Connect Fivetran to dbt Cloud](https://fivetran.com/docs/transformations/dbt-cloud/setup-guide)**:
   - In Fivetran, go to the **Transformations** tab.
   - Click on **Add Transformation** and select **dbt Cloud** as the transformation tool.
   - Authenticate Fivetran to use dbt Cloud by adding your dbt Cloud API key.

2. **Set Up dbt Job**:
   - Configure the dbt job to run after the connector sync completes.

---

### Part 3: Add New Data and Re-Sync

#### Task 1: Re-Sync New Data in Fivetran and Re-Run dbt Job

1. **Add New Data to Google Drive or Azure Blob**:
   - Add new files to the synced folders in Google Drive or Blob Storage.

2. **Re-Sync Connections**:
   - In Fivetran, click **Sync Now** for both Azure Blob and Google Drive connectors to sync the new data.

3. **Auto Re-run dbt Job**:
   - After the sync, the dbt job will automatically be re-run, applying the transformation and generating the new results in BigQuery.

---

## Conclusion

In this lab, we explored how to set up Fivetran connectors for Azure Blob Storage and Google Drive, sync them to BigQuery, and use dbt Cloud to transform the data. We also automated the process by connecting Fivetran to dbt Cloud, ensuring that new data syncs automatically trigger dbt transformations.

---

## Useful Links

- **dbt Documentation**: Comprehensive guide on using dbt to build and manage your transformation layer.
  - [Introduction to dbt](https://docs.getdbt.com/docs/introduction)
  - [dbt Cloud Getting Started Guide](https://docs.getdbt.com/docs/dbt-cloud/cloud-quickstart)
  - [dbt Models](https://docs.getdbt.com/docs/build/models)
  - [dbt Sources](https://docs.getdbt.com/docs/building-a-dbt-project/using-sources)

- **Fivetran Documentation**: Step-by-step guide to getting started with Fivetran, setting up connectors, and syncing data.
  - [Fivetran Getting Started](https://fivetran.com/docs/getting-started)
  - [Fivetran Connectors](https://fivetran.com/docs/connectors)
  - [Syncing Data to BigQuery](https://fivetran.com/docs/destinations/bigquery)

- **Google BigQuery Documentation**: Learn how to use Google BigQuery for querying, storing, and managing large datasets.
  - [BigQuery Documentation](https://cloud.google.com/bigquery/docs/introduction)

- **Azure Blob Storage Documentation**: A detailed guide on how to use Azure Blob Storage.
  - [Azure Blob Storage Documentation](https://docs.microsoft.com/en-us/azure/storage/blobs/)
