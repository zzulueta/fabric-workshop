---
  title: 'Lab Fabric Intermediate: Lakehouse, Warehouse & Direct Lake Analytics'
  module: 'Microsoft Fabric Bootcamp – Intermediate Lab'
---

# Lab Fabric Intermediate – Lakehouse, Warehouse & Direct Lake Analytics

## Lab Introduction

In this intermediate lab you build a complete end-to-end analytics solution using
Microsoft Fabric's SaaS-first unified analytics platform. You will work through
the following tasks:

- Provision Microsoft Fabric workspaces backed by Fabric capacity and understand the SaaS-first analytics model
- Build a Lakehouse on OneLake and ingest data using shortcuts, file uploads, and Dataflows Gen2
- Transform raw data with PySpark notebooks and the SQL analytics endpoint over a single OneLake copy
- Design a Fabric Data Warehouse with T-SQL COPY INTO, stored procedures, and dimensional modeling
- Build a Direct Lake semantic model that reads Delta tables natively without import or DirectQuery overhead
- Create Power BI reports layered directly on Fabric semantic models for live, low-latency insights
- Apply medallion architecture (Bronze / Silver / Gold) to organize raw, refined, and curated data layers

## Pre-requisites

- An active Azure subscription with Fabric capacity (F2)
- Owner or Admin rights on a Fabric workspace
- Basic familiarity with SQL, Python, and Power BI

## Estimated Timing: 180 minutes

## Lab Scenario

Your organisation is modernising its data analytics platform by migrating to Microsoft
Fabric. The current landscape includes raw data landing in various formats (CSV, Parquet,
JSON) from multiple sources including Azure Data Lake Storage Gen2, on-premises databases,
and SaaS applications. The business requires:

- A scalable data lakehouse that can ingest and process structured and semi-structured data
- PySpark-based transformation pipelines for cleansing and enriching raw data
- A dimensional data warehouse for aggregated business metrics and historical reporting
- Real-time Power BI dashboards with low latency that don't require full data imports
- Clear data quality zones (Bronze, Silver, Gold) aligned with the medallion architecture

You have been tasked with:

- Provisioning a Fabric workspace and configuring OneLake storage
- Building a Lakehouse with multiple ingestion patterns (shortcuts, uploads, Dataflows Gen2)
- Developing PySpark notebooks to implement Bronze → Silver → Gold transformations
- Creating a Fabric Data Warehouse with T-SQL DDL, COPY INTO, and stored procedures
- Designing a Direct Lake semantic model that virtualizes data from Delta tables
- Building Power BI reports connected to the semantic model for live analytics
- Documenting the medallion architecture and data lineage

## Architecture Overview

```
Microsoft Fabric Workspace (FabricWorkspace-Intermediate)
├── Fabric Capacity (F2)
│
├── OneLake Storage (auto-provisioned, Delta Lake format)
│   ├── Bronze Layer   — raw, unprocessed data
│   │   ├── Shortcut to ADLS Gen2 (external data sources)
│   │   ├── Manual file uploads (CSV product catalog and customers)
│   │   ├── orders.csv (order header data)
│   │   └── order_details.csv (order line item data)
│   │
│   ├── Silver Layer   — cleansed, validated, conformed data
│   │   ├── PySpark notebook: bronze_to_silver_customers
│   │   ├── PySpark notebook: bronze_to_silver_products
│   │   ├── PySpark notebook: bronze_to_silver_orders
│   │   ├── PySpark notebook: bronze_to_silver_order_details
│   │   └── SQL analytics endpoint (read-only T-SQL queries)
│   │
│   └── Gold Layer     — business-level aggregates and dimensions
│       ├── PySpark notebook: silver_to_gold_fact_sales
│       ├── PySpark notebook: silver_to_gold_dim_customers
│       └── Delta tables: FactSales, DimCustomers, DimProducts, DimDate
│
├── Lakehouse
│   ├── Name: LakehouseIntermediate
│   ├── Tables: Bronze, Silver, Gold (Delta format)
│   └── SQL Analytics Endpoint (auto-generated)
│       └── Query Gold tables via T-SQL without ETL
│
├── Data Warehouse
│   ├── Name: WarehouseIntermediate
│   ├── Schema: Sales (dimensional model)
│   │   ├── FactSales (grain: one row per order line item)
│   │   ├── DimCustomers (SCD Type 1)
│   │   ├── DimProducts (SCD Type 1)
│   │   └── DimDate (pre-populated calendar)
│   ├── Ingestion: T-SQL COPY INTO from Lakehouse Gold tables
│   └── Stored Procedures: sp_Load_FactSales, sp_Load_Dimensions
│
├── Direct Lake Semantic Model
│   ├── Name: SemanticModel_SalesAnalytics
│   ├── Mode: Direct Lake (no import, no DirectQuery aggregations)
│   ├── Tables: FactSales, DimCustomers, DimProducts, DimDate
│   ├── Relationships: Star schema (fact → dimensions)
│   └── Measures: Total Revenue, Order Count, Avg Order Value
│
└── Power BI Report
    ├── Name: Sales Dashboard – Intermediate
    ├── Data source: SemanticModel_SalesAnalytics (Direct Lake)
    ├── Pages:
    │   ├── Executive Summary (KPIs, trends)
    │   ├── Customer Analysis (top customers, segmentation)
    │   └── Product Performance (category sales, inventory)
    └── Refresh: Real-time (no scheduled refresh required)

Data Flow:
External Sources → Bronze (Lakehouse) → Silver (PySpark) → Gold (PySpark)
                                                           ↓
                                            Warehouse (T-SQL COPY INTO)
                                                           ↓
                                        Direct Lake Semantic Model ← Power BI Reports
```

## Job Skills

- Task 1: Provision a Fabric workspace and configure capacity
- Task 2: Create a Lakehouse and understand OneLake storage
- Task 3: Ingest data into Bronze layer using shortcuts, file uploads, and Dataflows Gen2
- Task 4: Transform Bronze to Silver using PySpark notebooks
- Task 5: Transform Silver to Gold using PySpark and implement dimensional modeling
- Task 6: Query the Lakehouse using the SQL analytics endpoint
- Task 7: Create a Fabric Data Warehouse and load data with T-SQL COPY INTO
- Task 8: Build stored procedures for warehouse data loads
- Task 9: Design a Direct Lake semantic model on Gold layer Delta tables
- Task 10: Create Power BI reports connected to the Direct Lake semantic model
- Task 11: Validate the medallion architecture and review data lineage

---

## Task 1: Provision a Fabric Workspace and Configure Capacity

Microsoft Fabric workspaces are the foundational containers for all Fabric items
(Lakehouses, Warehouses, Notebooks, Semantic Models, Reports). A workspace must be
backed by a Fabric capacity (F SKU) to enable Fabric-specific features.

> **Design note:** Fabric capacities are billed based on compute usage (CU seconds),
> not storage. OneLake storage is billed separately per GB stored. This lab uses an
> F2 capacity, which provides a good balance of performance and cost for learning
> and development scenarios.

### Deploy a Fabric Capacity (F2) in Azure Portal

Before creating a Fabric workspace, you need to provision a Fabric capacity in Azure. 
This capacity provides the compute resources for all Fabric workloads.

1. Sign in to the [Azure Portal](https://portal.azure.com).

2. In the search bar at the top, type **Microsoft Fabric** and select **Microsoft Fabric** 
   from the results.

3. Select **+ Create** to create a new Fabric capacity.

4. On the **Basics** tab, configure the following settings:

   | Setting | Value |
   | --- | --- |
   | Subscription | Select your Azure subscription |
   | Resource group | Select **Create new** and type `rg-fabric-workshop` |
   | Capacity name | Enter `fabriccapacityworkshop` |
   | Region | Select a region close to your location (e.g., Australia East or West Europe) |
   | Size | Select **F2** (2 capacity units) |

   > **Note:** F2 is the smallest Fabric capacity SKU and is suitable for development 
   > and learning scenarios. Production workloads typically require F4 or higher.

5. Select **Review + create**.

6. Review the configuration and estimated cost. Fabric capacities are billed per hour 
   while running.

7. Select **Create** to deploy the capacity.

8. Wait for the deployment to complete (typically 2-5 minutes). You can monitor progress 
   in the **Notifications** panel.

9. Once deployed, select **Go to resource** to view your Fabric capacity.

> **Cost Management Tip:** Fabric capacities can be paused when not in use to avoid 
> charges. To pause a capacity, navigate to the capacity resource in Azure Portal and 
> select **Pause** from the toolbar. Remember to resume it before starting the lab.

### Create a Fabric Workspace

1. Sign in to the [Fabric portal](https://app.fabric.microsoft.com), select **Workspaces** from the left navigation pane.

2. Select **+ New workspace**.

3. Configure the workspace:

   | Setting | Value |
   | --- | --- |
   | Name | `FabricWorkspace-Intermediate` |
   | Description | `Intermediate lab for Lakehouse, Warehouse, and Direct Lake analytics` |
   | Workspace type | **Fabric** |
   | Details | Select your **F2** capacity from the dropdown |

4. Select **Apply** to create the workspace.

5. You are now in your workspace. The workspace is empty and ready for Fabric items.

---

## Task 2: Create a Lakehouse and Understand OneLake Storage

A Lakehouse in Fabric combines the flexibility of a data lake (file storage, open formats)
with the query performance of a data warehouse (SQL analytics endpoint, automatic schema
discovery). Every Lakehouse is backed by OneLake, which stores data in Delta Lake format.

### Create a Lakehouse

1. In your workspace (**FabricWorkspace-Intermediate**), select **+ New item**.

2. Under **Store data**, select **Lakehouse**.

3. Name the Lakehouse `LakehouseIntermediate`, enable Lakehouse schemas and select **Create**.

4. After a few moments, your Lakehouse opens. You will see two main sections:
   - **Tables** — structured Delta tables with schema and indexing
   - **Files** — unstructured or semi-structured files (Parquet, CSV, JSON)

5. On the upper left, notice the **Notebook** and **SQL analytics endpoint** selections. The SQL
   analytics endpoint is automatically created and allows you to query tables using T-SQL
   without any ETL.

### Understand OneLake and Delta Lake

1. In the Lakehouse explorer, select **Files**.

2. OneLake automatically organizes files and tables under your workspace. All data is
   stored in **Delta Lake format** by default, which provides:
   - ACID transactions
   - Schema enforcement and evolution
   - Time travel (query historical versions)
   - Efficient upserts and deletes

3. OneLake is accessible via ABFS paths (Azure Blob File System) and supports shortcuts
   to external storage accounts (ADLS Gen2, AWS S3, etc.) without copying data.

---

## Task 3: Ingest Data into Bronze Layer Using Shortcuts, File Uploads, and Dataflows Gen2

The Bronze layer contains raw, unprocessed data ingested from source systems. In this
task you use three different ingestion methods to populate the Bronze layer.

### Method 1: Create a Shortcut to External Azure Blob Storage

Shortcuts in Fabric allow you to access external data without copying it into OneLake. This
is ideal for large datasets that already exist in cloud storage. In this exercise, you'll
create a shortcut to an Azure Blob Storage container containing orders and order
details data.

> **What is a Shortcut?** A shortcut creates a virtual folder in your Lakehouse that points
> to external storage (ADLS Gen2, AWS S3, etc.). The data stays in the original location,
> but you can query it as if it were in OneLake. Benefits include:
> - No data duplication
> - Reduced storage costs
> - Near real-time access to source data
> - Single source of truth for shared datasets

#### Create the Shortcut

1. In the Lakehouse (**LakehouseIntermediate**), expand **Files** in the left navigation.

2. Right-click on **Files** and select **New subfolder**. Enter `bronze` as the folder name and select **Create**.

3. Right-click on the **bronze** folder and select **New shortcut**.

4. In the **New shortcut** dialog, select **Azure Blob Storage**.

5. Select New connection and configure the connection settings:

   | Setting | Value |
   | --- | --- |
   | **URL** | `https://fabricstoragezz.blob.core.windows.net/` |
   | **Connection** | Create new connection |
   | **Connection name** | `FabricWorkshop_BlobStorage` |
   | **Authentication kind** | Account key |
   | **Account key** | `Request from your Instructor` |

6. Select **Next**.

7. In the **Select a bucket or directory** step, **select** `northwind-orders`.

8. Select **Next**.

9. In the Transform section (if it appears), select **Skip** to keep the data in its original CSV format.

10. Review the details and select **Create** to create the shortcut.

#### Verify the Shortcut

1. Navigate to **Files/bronze/northwind-orders** folder in the Lakehouse explorer. The folder must have a clipboard icon indicating it's a shortcut.

2. You should see the container contents:
   - **orders.csv**
   - **order_details.csv**

3. Select one of the files to preview its contents. You'll see the order data with columns
   like OrderID, CustomerID, OrderDate, etc.

> **Important:** These files remain in Azure Blob Storage (`fabricstoragezz`). The shortcut
> provides a virtual view into the external storage without moving or copying data. Any
> updates to the source files in Azure Blob Storage will be immediately visible in the Lakehouse.

> **Production Tip:** In enterprise scenarios, you would typically create shortcuts to:
> - Corporate data lakes (ADLS Gen2)
> - AWS S3 buckets (cross-cloud scenarios)
> - Partner data shares
> - Historical archives that don't need to be copied

### Method 2: Upload CSV Files (Product Catalog)

1. Download the **products.csv** file from the sample-data folder of this lab repository.

2. In the Lakehouse, under **Files/bronze**, right click to create a new subfolder named **products**.

3. Select the **products** folder, then select **Get data** → **Upload files**.

4. Upload the **products.csv** file.

5. Once uploaded, exit the upload dialog. The file appears under **Files/bronze/products/**.

### Method 3: Ingest Customer Data Using Dataflows Gen2

Dataflows Gen2 is Fabric's Power Query-based ETL tool that provides a visual, low-code
interface for data ingestion and transformation. In this section, you'll create a
real dataflow to ingest data from the Northwind OData service.

> **Note:** Dataflows Gen2 supports connecting to various data sources including Azure SQL,
> on-premises SQL Server, REST APIs, OData services, and more. This example uses an OData
> service for simplicity, but the same principles apply to any data source.

#### Create a Dataflow Gen2

1. Go back to your workspace (**FabricWorkspace-Intermediate**), select **+ New item** → **Dataflow Gen2** found under Get data.

2. Name the dataflow `Dataflow_Bronze_Customers` and select **Create**.

3. The Dataflow editor opens with a blank canvas.

#### Get Data from OData Source

1. In the dataflow editor, select **Get data from another source**.

2. In **New sources**, select **View more**.

3. In the search box, type **OData** and select the **OData** connector.

4. In the connection settings, enter the following URL and select **Next**:
   ```
   https://services.odata.org/v4/northwind/northwind.svc/
   ```
   > **Note:** Accept the default authentication method (Anonymous) since this is a public OData service.

5. In the Choose data dialog, select the **Customers** table and select **Create**.

   > **Note:** The Northwind OData service provides sample data that matches the structure
   > of the CSV files in this lab. You're loading only the Customers table for this dataflow.

#### Apply Transformations in Power Query

1. Enable **Data Profiling tools** for better visibility:
   - Select **Home** → **Options** (looks like a list icon) → **Global Options**.
   - Under **Column profile**, check all four options:
     - Enable column profile
     - Show column quality details in data preview
     - Show column value distribution in data preview
     - Show column profile in details pane
   - Select **OK**.

2. Enable **Diagram View** for a visual representation of your queries:
   - Select the **View** tab → **Diagram view**.

3. Review the Customers data in the preview pane. You should see columns like CustomerID,
   CompanyName, ContactName, Country, etc.

4. Apply transformations to cleanse the data:
   - Select the **ContactName** column.
   - Right-click and select **Transform column** → **Text transforms** → **Trim** to remove leading/trailing spaces.
   
5. Standardize the Country column:
   - Select the **Country** column.
   - Select **Transform** tab → **Format** (pencil icon) → **UPPERCASE**.

6. Remove unnecessary columns to keep only relevant fields:
   - Select **Home** tab → **Choose Columns** (highlighted table icon).
   - Check only: **CustomerID**, **CompanyName**, **ContactName**, **ContactTitle**, **City**, **Country**.
   - Select **OK**.

7. Rename the query to make it more descriptive:
   - Expand the **Query settings** pane on the right, change the **Name** from "Customers" to **"Bronze_Customers"**.

#### Configure Data Destination

1. In the **Query settings** pane, scroll down and select **+ (Add data destination)**.

2. Select **Lakehouse** as the destination type.

3. In the **Connect to data destination** dialog:
   - Select **Create new connection** for Connection.
   - Enter a **Connection name:** `Bronze_Customers_Connection`
   - Select **Next**.

4. In the **Choose destination target** dialog:
   - Select **LakehouseIntermediate > dbo** under FabricWorkspace-Intermediate.
   - Enter table name: **bronze_customers_dataflow**.
   - Select **Next**.

5. In the **Choose destination settings** dialog:
   - Select **Use automatic settings**.
   - Select **Save settings**.

#### Save and Run the Dataflow

1. Review your dataflow in Diagram view. You should see:
   - The **Bronze_Customers** query
   - A destination indicator showing it's connected to your Lakehouse

2. Select **Save & run** under the Home tab to save and run your dataflow.

3. You should see a message indicating the dataflow is running at the bottom of the screen.

4. Wait for the run to complete (a green checkmark appears at the bottom of the screen).


#### Verify the Ingested Data

1. Navigate back to your **LakehouseIntermediate**.

2. In the **Tables > dbo** section, right-click and select **Refresh** to update the table list.

3. You should see the new **bronze_customers_dataflow** table.

4. Select the table to preview the data and confirm it was ingested successfully.

#### (Optional) Schedule Dataflow Refresh

1. In your workspace, go to the dataflow name and select the ellipsis icon.

2. Select **Schedule**.

2. Select **Add schedule**.

3. Configure the refresh schedule:
   - **Repeat**: Daily
   - **Time of day**: Select a time (e.g., 4:00 AM)
   - **Start date and time**: Set to the current date and time or a future time
   - **End date and time**: Set to a future date (e.g., 12 months from now)
   - **Time zone**: Select your preferred time zone

4. Select **Save** to save the schedule.

> **Note:** For this lab, scheduled refresh is optional since we're working with static
> sample data. In production scenarios, you would schedule refreshes to keep your data
> current.

### Summary: Bronze Layer Data Sources

You now have Bronze data from three different sources:

1. **Shortcut** (Method 1): Files/bronze/northwind-orders/orders.csv and Files/bronze/northwind-orders/order_details.csv
   - Source: Azure Blob Storage (fabricstoragezz/northwind-orders)
   - Accessed without copying data into OneLake

2. **Upload** (Method 2): Files/bronze/products/products.csv
   - Source: Local CSV file uploaded directly to OneLake

3. **Dataflow Gen2** (Method 3): bronze_customers_dataflow table
   - Source: Northwind OData API with Power Query transformations

### Summary: Multiple Ingestion Patterns

In this task, you've explored three different data ingestion patterns:

1. **Shortcuts**: Virtual folders that reference external Azure Blob Storage
   - **Data source**: Azure Blob Storage (fabricstoragezz/northwind-orders)
   - **Files**: orders.csv, order_details.csv
   - **Use case**: Large datasets in Azure Blob Storage, AWS S3, or other cloud storage
   - **Benefits**: No data duplication, reduced storage costs, near real-time access to source data
   - **How it works**: Creates a virtual pointer - data stays in Azure Blob, queries execute against external storage

2. **File uploads**: Direct upload of CSV files to OneLake
   - **Data source**: Local files
   - **Files**: products.csv
   - **Use case**: Small to medium datasets, one-time imports, sample data
   - **Benefits**: Simple, quick, no external dependencies

3. **Dataflows Gen2**: Visual ETL tool with Power Query transformations
   - **Data source**: Northwind OData API
   - **Tables**: Customers (with transformations)
   - **Use case**: Connecting to databases, APIs, SaaS applications with transformation logic
   - **Benefits**: Low-code, reusable, scheduled refresh, built-in data quality

In production environments, you would typically choose one primary ingestion method based
on your data sources, team skills, and operational requirements. This lab demonstrates all
three to give you hands-on experience with Fabric's flexible ingestion capabilities.

---

## Task 4: Transform Bronze to Silver Using PySpark Notebooks

The Silver layer contains cleansed, validated, and conformed data. In this task you
create PySpark notebooks to transform Bronze files into Silver Delta tables.

### Create a Notebook for Customer Silver Transformation

1. Go back to your workspace, select **+ New item** → **Notebook** (under Prepare data).

2. Name the notebook `bronze_to_silver_customers`. Ensure the location is set to the FabricWorkspace-Intermediate. Select **Create**.

3. In the Explorer pane on the left, go to the Data items section. Select **Add data items → From OneLake catalog**, then select LakehouseIntermediate. Click **Add**.
> You should see the LakehouseIntermediate appear under Data items. Expand it to see the tables and files.

4. Go to your notebook, Add code cell (by pressing **+ Code**), and paste the following PySpark code:

   ```python
   # Load raw customer data from Bronze
   from pyspark.sql.functions import col, trim, upper
   
   # Load from Dataflow Gen2 table
   df_customers_bronze = spark.read.table("bronze_customers_dataflow")
   print("✓ Loaded customer data from Dataflow Gen2 table")
   
   # Cleanse and validate
   df_customers_silver = df_customers_bronze \
       .withColumn("CompanyName", trim(col("CompanyName"))) \
       .withColumn("ContactName", trim(col("ContactName"))) \
       .withColumn("Country", upper(trim(col("Country")))) \
       .filter(col("CustomerID").isNotNull()) \
       .dropDuplicates(["CustomerID"])
   
   # Write to Silver layer as Delta table
   df_customers_silver.write.format("delta") \
       .mode("overwrite") \
       .option("overwriteSchema", "true") \
       .saveAsTable("silver_customers")
   
   print(f"✓ Silver customers table created successfully with {df_customers_silver.count()} rows.")
   ```

   > **Note:** This notebook loads customer data directly from the Dataflow Gen2 table
   > created in Task 3. All data cleansing and transformations are applied consistently
   > to produce the Silver layer table.

5. Select **Run cell** to execute the notebook.

6. Once complete, navigate to the Lakehouse **Tables** section. Refresh the Tables. You should see a new table named **silver_customers**.

7. Stop the session by clicking the stop button in the navigation bar at the top of the notebook.

### Create a Notebook for Product Silver Transformation

1. Create a new notebook named `bronze_to_silver_products`.

2. Add the Lakehouse in the Data items section.

3. Add the following PySpark code:

   ```python
   # Load raw product data from Bronze
   df_products_bronze = spark.read.format("csv") \
       .option("header", "true") \
       .option("inferSchema", "true") \
       .load("Files/bronze/products/products.csv")
   
   # Cleanse and validate
   from pyspark.sql.functions import col, trim, round
   
   df_products_silver = df_products_bronze \
       .withColumnRenamed("productID", "ProductID") \
       .withColumnRenamed("productName", "ProductName") \
       .withColumnRenamed("quantityPerUnit", "QuantityPerUnit") \
       .withColumnRenamed("unitPrice", "UnitPrice") \
       .withColumnRenamed("discontinued", "Discontinued") \
       .withColumnRenamed("categoryID", "CategoryID") \
       .withColumnRenamed("unitCost", "UnitCost") \
       .withColumn("ProductName", trim(col("ProductName"))) \
       .withColumn("UnitPrice", round(col("UnitPrice"), 2)) \
       .withColumn("UnitCost", round(col("UnitCost"), 2)) \
       .filter(col("ProductID").isNotNull()) \
       .dropDuplicates(["ProductID"])
   
   # Write to Silver layer as Delta table
   df_products_silver.write.format("delta") \
       .mode("overwrite") \
       .option("overwriteSchema", "true") \
       .saveAsTable("silver_products")
   
   print("Silver products table created successfully.")
   ```

4. Run the notebook. Verify the **silver_products** table appears in the Lakehouse.

5. Stop the session.

### Create Notebooks for Orders and Order Details Silver Transformation

1. Create a new notebook named `bronze_to_silver_orders`.

2. Add the Lakehouse in the Data items section.

3. Add the following PySpark code:

   ```python
   # Load raw orders data from Bronze (via Shortcut)
   df_orders_bronze = spark.read.format("csv") \
       .option("header", "true") \
       .option("inferSchema", "true") \
       .load("Files/bronze/northwind-orders/orders.csv")
   
   # Note: This file is accessed via shortcut from Azure Blob Storage (fabricstoragezz)
   # The data is not copied into OneLake - we're reading directly from external storage
   
   # Cleanse and validate
   from pyspark.sql.functions import col, to_date, round
   
   df_orders_silver = df_orders_bronze \
       .withColumnRenamed("orderID", "OrderID") \
       .withColumnRenamed("customerID", "CustomerID") \
       .withColumnRenamed("employeeID", "EmployeeID") \
       .withColumnRenamed("orderDate", "OrderDate") \
       .withColumnRenamed("requiredDate", "RequiredDate") \
       .withColumnRenamed("shippedDate", "ShippedDate") \
       .withColumnRenamed("shipperID", "ShipperID") \
       .withColumnRenamed("freight", "Freight") \
       .withColumn("OrderDate", to_date(col("OrderDate"), "yyyy-MM-dd")) \
       .withColumn("RequiredDate", to_date(col("RequiredDate"), "yyyy-MM-dd")) \
       .withColumn("ShippedDate", to_date(col("ShippedDate"), "yyyy-MM-dd")) \
       .withColumn("Freight", round(col("Freight"), 2)) \
       .filter(col("OrderID").isNotNull()) \
       .dropDuplicates(["OrderID"])
   
   # Write to Silver layer as Delta table
   df_orders_silver.write.format("delta") \
       .mode("overwrite") \
       .option("overwriteSchema", "true") \
       .saveAsTable("silver_orders")
   
   print("Silver orders table created successfully.")
   ```

4. Run the notebook. Verify the **silver_orders** table appears in the Lakehouse.

5. Stop the session.

6. Create another new notebook named `bronze_to_silver_order_details`.

7. Add the Lakehouse in the Data items section.

8. Add the following PySpark code:

   ```python
   # Load raw order details data from Bronze (via Shortcut)
   df_order_details_bronze = spark.read.format("csv") \
       .option("header", "true") \
       .option("inferSchema", "true") \
       .load("Files/bronze/northwind-orders/order_details.csv")
   
   # Note: This file is accessed via shortcut from Azure Blob Storage (fabricstoragezz)
   
   # Cleanse and validate
   from pyspark.sql.functions import col, round
   
   df_order_details_silver = df_order_details_bronze \
       .withColumnRenamed("orderID", "OrderID") \
       .withColumnRenamed("productID", "ProductID") \
       .withColumnRenamed("unitPrice", "UnitPrice") \
       .withColumnRenamed("quantity", "Quantity") \
       .withColumnRenamed("discount", "Discount") \
       .withColumn("UnitPrice", round(col("UnitPrice"), 2)) \
       .withColumn("Discount", round(col("Discount"), 2)) \
       .filter(col("OrderID").isNotNull()) \
       .filter(col("ProductID").isNotNull()) \
       .filter(col("Quantity") > 0)
   
   # Write to Silver layer as Delta table
   df_order_details_silver.write.format("delta") \
       .mode("overwrite") \
       .option("overwriteSchema", "true") \
       .saveAsTable("silver_order_details")
   
   print("Silver order_details table created successfully.")
   ```

9. Run the notebook. Verify the **silver_order_details** table appears in the Lakehouse.

10. Stop the session.

---

## Task 4.5: Implement Data Quality Checks in the Silver Layer

Data quality validation is critical for building reliable analytics pipelines. In this task, you'll implement quality checks to validate data integrity, completeness, and referential relationships before promoting data to the Gold layer.

> **Why Data Quality Matters:** Poor data quality leads to incorrect business insights, broken reports, and loss of trust in analytics. Implementing quality checks early in the pipeline prevents bad data from propagating downstream.

### Create a Data Quality Validation Notebook

1. Create a new notebook named `silver_data_quality_checks`.

2. Add the Lakehouse in the Data items section.

3. In a new cell, paste the following PySpark code:

   ```python
   # Silver Layer Data Quality Validation
   # This notebook performs comprehensive data quality checks on Silver tables
   
   from pyspark.sql.functions import col, count, when, isnan, isnull, sum as spark_sum
   from datetime import datetime
   
   # Initialize quality check results
   quality_results = []
   
   print("=" * 80)
   print("SILVER LAYER DATA QUALITY VALIDATION")
   print(f"Execution Time: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")
   print("=" * 80)
   ```

4. Add a new cell for **Completeness Checks**:

   ```python
   # ========================================
   # CHECK 1: Completeness - Check for NULL values in critical columns
   # ========================================
   
   print("\n1. COMPLETENESS CHECKS")
   print("-" * 80)
   
   # Check customers table
   df_customers = spark.read.table("silver_customers")
   total_customers = df_customers.count()
   
   null_checks_customers = df_customers.select([
       spark_sum(when(col("CustomerID").isNull(), 1).otherwise(0)).alias("CustomerID_nulls"),
       spark_sum(when(col("CompanyName").isNull(), 1).otherwise(0)).alias("CompanyName_nulls"),
       spark_sum(when(col("Country").isNull(), 1).otherwise(0)).alias("Country_nulls")
   ]).collect()[0]
   
   print(f"\n📊 Customers Table ({total_customers} rows):")
   print(f"   CustomerID NULLs: {null_checks_customers['CustomerID_nulls']}")
   print(f"   CompanyName NULLs: {null_checks_customers['CompanyName_nulls']}")
   print(f"   Country NULLs: {null_checks_customers['Country_nulls']}")
   
   # Check orders table
   df_orders = spark.read.table("silver_orders")
   total_orders = df_orders.count()
   
   null_checks_orders = df_orders.select([
       spark_sum(when(col("OrderID").isNull(), 1).otherwise(0)).alias("OrderID_nulls"),
       spark_sum(when(col("CustomerID").isNull(), 1).otherwise(0)).alias("CustomerID_nulls"),
       spark_sum(when(col("OrderDate").isNull(), 1).otherwise(0)).alias("OrderDate_nulls")
   ]).collect()[0]
   
   print(f"\n📊 Orders Table ({total_orders} rows):")
   print(f"   OrderID NULLs: {null_checks_orders['OrderID_nulls']}")
   print(f"   CustomerID NULLs: {null_checks_orders['CustomerID_nulls']}")
   print(f"   OrderDate NULLs: {null_checks_orders['OrderDate_nulls']}")
   
   # Record results
   quality_results.append({
       "check_category": "Completeness",
       "check_name": "Critical NULL checks",
       "status": "PASS" if (null_checks_customers['CustomerID_nulls'] == 0 and 
                           null_checks_orders['OrderID_nulls'] == 0) else "FAIL"
   })
   ```

5. Add a new cell for **Referential Integrity Checks**:

   ```python
   # ========================================
   # CHECK 2: Referential Integrity - Orphaned records
   # ========================================
   
   print("\n2. REFERENTIAL INTEGRITY CHECKS")
   print("-" * 80)
   
   # Check for orders with invalid CustomerIDs (orphaned orders)
   df_orders = spark.read.table("silver_orders")
   df_customers = spark.read.table("silver_customers")
   
   orphaned_orders = df_orders.join(
       df_customers, 
       df_orders.CustomerID == df_customers.CustomerID, 
       "left_anti"
   )
   
   orphan_count = orphaned_orders.count()
   
   print(f"\n🔗 Orders → Customers Referential Integrity:")
   print(f"   Total Orders: {df_orders.count()}")
   print(f"   Orphaned Orders (no matching customer): {orphan_count}")
   
   if orphan_count > 0:
       print("\n   ⚠️  WARNING: Found orphaned orders!")
       print("   Sample orphaned CustomerIDs:")
       orphaned_orders.select("OrderID", "CustomerID").show(5, truncate=False)
   else:
       print("   ✓ All orders have valid customer references")
   
   # Check for order details with invalid OrderIDs
   df_order_details = spark.read.table("silver_order_details")
   
   orphaned_details = df_order_details.join(
       df_orders,
       df_order_details.OrderID == df_orders.OrderID,
       "left_anti"
   )
   
   orphan_details_count = orphaned_details.count()
   
   print(f"\n🔗 Order Details → Orders Referential Integrity:")
   print(f"   Total Order Details: {df_order_details.count()}")
   print(f"   Orphaned Details (no matching order): {orphan_details_count}")
   
   if orphan_details_count > 0:
       print("   ⚠️  WARNING: Found orphaned order details!")
   else:
       print("   ✓ All order details have valid order references")
   
   # Check for order details with invalid ProductIDs
   df_products = spark.read.table("silver_products")
   
   orphaned_product_refs = df_order_details.join(
       df_products,
       df_order_details.ProductID == df_products.ProductID,
       "left_anti"
   )
   
   orphan_products_count = orphaned_product_refs.count()
   
   print(f"\n🔗 Order Details → Products Referential Integrity:")
   print(f"   Total Order Details: {df_order_details.count()}")
   print(f"   Invalid Product References: {orphan_products_count}")
   
   if orphan_products_count > 0:
       print("   ⚠️  WARNING: Found invalid product references!")
   else:
       print("   ✓ All order details have valid product references")
   
   # Record results
   quality_results.append({
       "check_category": "Referential Integrity",
       "check_name": "Orders → Customers",
       "status": "PASS" if orphan_count == 0 else "FAIL",
       "issue_count": orphan_count
   })
   
   quality_results.append({
       "check_category": "Referential Integrity",
       "check_name": "Order Details → Orders",
       "status": "PASS" if orphan_details_count == 0 else "FAIL",
       "issue_count": orphan_details_count
   })
   
   quality_results.append({
       "check_category": "Referential Integrity",
       "check_name": "Order Details → Products",
       "status": "PASS" if orphan_products_count == 0 else "FAIL",
       "issue_count": orphan_products_count
   })
   ```

6. Add a new cell for **Business Rule Validation**:

   ```python
   # ========================================
   # CHECK 3: Business Rules - Data validity
   # ========================================
   
   print("\n3. BUSINESS RULE VALIDATION")
   print("-" * 80)
   
   # Check for negative quantities
   negative_quantities = df_order_details.filter(col("Quantity") < 0).count()
   print(f"\n📋 Order Details Validation:")
   print(f"   Negative Quantities: {negative_quantities}")
   
   # Check for negative or zero prices
   invalid_prices = df_order_details.filter((col("UnitPrice") <= 0)).count()
   print(f"   Invalid Unit Prices (≤ 0): {invalid_prices}")
   
   # Check for invalid discount values (should be between 0 and 1)
   invalid_discounts = df_order_details.filter(
       (col("Discount") < 0) | (col("Discount") > 1)
   ).count()
   print(f"   Invalid Discounts (outside 0-1 range): {invalid_discounts}")
   
   # Check for future order dates
   from pyspark.sql.functions import current_date
   future_orders = df_orders.filter(col("OrderDate") > current_date()).count()
   print(f"\n📅 Orders Date Validation:")
   print(f"   Future Order Dates: {future_orders}")
   
   # Check for orders where ShippedDate < OrderDate
   invalid_ship_dates = df_orders.filter(
       (col("ShippedDate").isNotNull()) & 
       (col("ShippedDate") < col("OrderDate"))
   ).count()
   print(f"   Invalid Ship Dates (before order date): {invalid_ship_dates}")
   
   # Record results
   business_rule_pass = (negative_quantities == 0 and invalid_prices == 0 and 
                         invalid_discounts == 0 and future_orders == 0 and 
                         invalid_ship_dates == 0)
   
   quality_results.append({
       "check_category": "Business Rules",
       "check_name": "Data validity checks",
       "status": "PASS" if business_rule_pass else "FAIL"
   })
   ```

7. Add a new cell for **Duplicate Detection**:

   ```python
   # ========================================
   # CHECK 4: Uniqueness - Duplicate detection
   # ========================================
   
   print("\n4. UNIQUENESS CHECKS")
   print("-" * 80)
   
   # Check for duplicate OrderIDs
   total_orders = df_orders.count()
   distinct_orders = df_orders.select("OrderID").distinct().count()
   duplicate_orders = total_orders - distinct_orders
   
   print(f"\n🔑 Primary Key Uniqueness:")
   print(f"   Orders - Total: {total_orders}, Distinct: {distinct_orders}, Duplicates: {duplicate_orders}")
   
   # Check for duplicate CustomerIDs
   total_customers = df_customers.count()
   distinct_customers = df_customers.select("CustomerID").distinct().count()
   duplicate_customers = total_customers - distinct_customers
   
   print(f"   Customers - Total: {total_customers}, Distinct: {distinct_customers}, Duplicates: {duplicate_customers}")
   
   # Record results
   quality_results.append({
       "check_category": "Uniqueness",
       "check_name": "Primary key duplicates",
       "status": "PASS" if (duplicate_orders == 0 and duplicate_customers == 0) else "FAIL"
   })
   ```

8. Add a final cell for **Summary Report**:

   ```python
   # ========================================
   # SUMMARY REPORT
   # ========================================

   print("\n" + "=" * 80)
   print("DATA QUALITY SUMMARY REPORT")
   print("=" * 80)

   # Convert results to DataFrame for easy viewing
   from pyspark.sql import Row

   # Ensure all dicts have the same set of keys before creating the DataFrame
   # This avoids schema mismatch errors when some entries miss optional fields like 'issue_count'.
   if quality_results:
      # Collect all keys that ever appear in any result
      all_keys = set().union(*[r.keys() for r in quality_results])

      # Normalize each record to include all keys (missing values become None)
      normalized_results = [
         {k: r.get(k, None) for k in all_keys}
         for r in quality_results
      ]

      results_df = spark.createDataFrame([Row(**r) for r in normalized_results])

      # Show the summary table
      results_df.show(truncate=False)

      # Count passes and failures
      total_checks = len(quality_results)
      passed_checks = sum(1 for r in quality_results if r["status"] == "PASS")
      failed_checks = total_checks - passed_checks

      print(f"\n📈 Overall Quality Score: {passed_checks}/{total_checks} checks passed ({(passed_checks/total_checks*100):.1f}%)")

      if failed_checks == 0:
         print("\n✅ ALL QUALITY CHECKS PASSED - Data is ready for Gold layer")
      else:
         print(f"\n⚠️  {failed_checks} QUALITY CHECK(S) FAILED - Review issues before promoting to Gold")
         print("   Action Required: Fix data quality issues in source systems or Bronze→Silver transformations")

      # Save quality report to Delta table for tracking
      results_df.write.format("delta") \
         .mode("append") \
         .option("mergeSchema", "true") \
         .saveAsTable("data_quality_reports")

      print("\n📝 Quality report saved to 'data_quality_reports' table")
   else:
      print("No quality results were generated; nothing to summarize or save.")


   ```

9. Select **Run all** to execute the quality validation.

10. Review the output in the notebook. You should see detailed results for each quality check category, along with an overall quality score.

11. Stop the session.

### Understanding the Quality Checks

This notebook implements four categories of data quality checks:

| Check Category | Purpose | Example |
| --- | --- | --- |
| **Completeness** | Ensures critical fields are populated | NULL checks on CustomerID, OrderID |
| **Referential Integrity** | Validates foreign key relationships | All orders have valid CustomerIDs |
| **Business Rules** | Enforces business logic constraints | Quantities > 0, dates in valid range |
| **Uniqueness** | Detects duplicate records | Primary key duplicates |

### Best Practices for Production Pipelines

**When to run quality checks:**
- ✅ After Bronze → Silver transformations (this lab)
- ✅ Before Silver → Gold transformations
- ✅ After data loads from external sources
- ✅ On a scheduled basis for data drift detection

**How to handle quality failures:**
1. **Log issues**: Save failed records to a quarantine table
2. **Alert stakeholders**: Send notifications for critical failures
3. **Block downstream processing**: Don't promote bad data to Gold
4. **Track metrics over time**: Monitor quality trends using the `data_quality_reports` table

**Example quarantine pattern:**
```python
# Save failed records to quarantine table for investigation
orphaned_orders.write.format("delta") \
    .mode("append") \
    .option("mergeSchema", "true") \
    .saveAsTable("quarantine_orphaned_orders")
```

> **Lab Note:** In this lab, the sample data is clean and should pass all checks. In production, you would implement automated remediation or manual review processes for failed checks.

---

## Task 5: Transform Silver to Gold Using PySpark and Implement Dimensional Modeling

The Gold layer contains business-level aggregates and dimensional models optimized
for analytics. In this task you create fact and dimension tables following a star schema.

### Create a Notebook for Gold Dimension: Customers

1. Create a new notebook named `silver_to_gold_dim_customers`.

2. Add the Lakehouse in the Data items section.

3. Add the following PySpark code:

   ```python
   # Load Silver customers
   df_silver_customers = spark.read.table("silver_customers")
   
   # Add surrogate key and transform to dimension
   from pyspark.sql.functions import monotonically_increasing_id, col
   
   df_dim_customers = df_silver_customers \
       .withColumn("CustomerKey", monotonically_increasing_id()) \
       .select("CustomerKey", "CustomerID", "CompanyName", "ContactName", "ContactTitle", "City", "Country")
   
   # Write to Gold layer as Delta table
   df_dim_customers.write.format("delta") \
       .mode("overwrite") \
       .option("overwriteSchema", "true") \
       .saveAsTable("gold_dim_customers")
   
   print("Gold DimCustomers table created successfully.")
   ```

4. Run the notebook. Verify the **gold_dim_customers** table appears in the Lakehouse.

5. Stop the session.

### Create a Notebook for Gold Dimension: Products

1. Create a new notebook named `silver_to_gold_dim_products`.

2. Add the Lakehouse in the Data items section.

3. Add the following PySpark code:

   ```python
   # Load Silver products
   df_silver_products = spark.read.table("silver_products")
   
   # Add surrogate key and transform to dimension
   from pyspark.sql.functions import monotonically_increasing_id, col
   
   df_dim_products = df_silver_products \
       .withColumn("ProductKey", monotonically_increasing_id()) \
       .select("ProductKey", "ProductID", "ProductName", "QuantityPerUnit", "UnitPrice", "CategoryID", "UnitCost", "Discontinued")
   
   # Write to Gold layer as Delta table
   df_dim_products.write.format("delta") \
       .mode("overwrite") \
       .option("overwriteSchema", "true") \
       .saveAsTable("gold_dim_products")
   
   print("Gold DimProducts table created successfully.")
   ```

4. Run the notebook. Verify the **gold_dim_products** table appears in the Lakehouse.

5. Stop the session.

### Create a Notebook for Gold Dimension: Date

1. Create a new notebook named `gold_dim_date`.

2. Add the Lakehouse in the Data items section.

3. Add the following PySpark code to generate a date dimension:

   ```python
   from pyspark.sql.functions import col, year, month, dayofmonth, dayofweek, quarter, date_format
   from datetime import datetime, timedelta
   
   # Generate date range (2013-01-01 to 2014-12-31)
   start_date = datetime(2013, 1, 1)
   end_date = datetime(2014, 12, 31)
   date_range = [start_date + timedelta(days=x) for x in range((end_date - start_date).days + 1)]
   
   df_dates = spark.createDataFrame([(d,) for d in date_range], ["Date"])
   
   # Add date attributes
   df_dim_date = df_dates \
       .withColumn("DateKey", date_format(col("Date"), "yyyyMMdd").cast("int")) \
       .withColumn("Year", year(col("Date"))) \
       .withColumn("Month", month(col("Date"))) \
       .withColumn("Day", dayofmonth(col("Date"))) \
       .withColumn("Quarter", quarter(col("Date"))) \
       .withColumn("DayOfWeek", dayofweek(col("Date"))) \
       .select("DateKey", "Date", "Year", "Month", "Day", "Quarter", "DayOfWeek")
   
   # Write to Gold layer
   df_dim_date.write.format("delta") \
       .mode("overwrite") \
       .option("overwriteSchema", "true") \
       .saveAsTable("gold_dim_date")
   
   print("Gold DimDate table created successfully.")
   ```

4. Run the notebook. Verify the **gold_dim_date** table appears in the Lakehouse.

5. Stop the session.

### Create a Notebook for Gold Fact: Sales

1. Create a new notebook named `silver_to_gold_fact_sales`.

2. Add the Lakehouse in the Data items section.

3. Add the following PySpark code:

   ```python
   # Load Silver orders and order details
   df_silver_orders = spark.read.table("silver_orders")
   df_silver_order_details = spark.read.table("silver_order_details")
   
   # Join orders with order details to create fact table
   df_fact_base = df_silver_order_details \
       .join(df_silver_orders, "OrderID", "inner")
   
   # Load dimensions to get surrogate keys
   df_dim_customers = spark.read.table("gold_dim_customers")
   df_dim_products = spark.read.table("gold_dim_products")
   
   # Join with dimensions
   from pyspark.sql.functions import col, date_format, round
   
   df_fact_sales = df_fact_base \
       .join(df_dim_customers, df_fact_base.CustomerID == df_dim_customers.CustomerID, "left") \
       .join(df_dim_products, df_fact_base.ProductID == df_dim_products.ProductID, "left") \
       .withColumn("DateKey", date_format(col("OrderDate"), "yyyyMMdd").cast("int")) \
       .withColumn("LineTotal", round((col("UnitPrice") * col("Quantity")) * (1 - col("Discount")), 2)) \
       .select(
           col("OrderID"),
           col("DateKey"),
           col("CustomerKey"),
           col("ProductKey"),
           col("UnitPrice"),
           col("Quantity"),
           col("Discount"),
           col("LineTotal")
       )
   
   # Write to Gold layer as Delta table
   df_fact_sales.write.format("delta") \
       .mode("overwrite") \
       .option("overwriteSchema", "true") \
       .saveAsTable("gold_fact_sales")
   
   print("Gold FactSales table created successfully.")
   ```

4. Run the notebook. Verify the **gold_fact_sales** table appears in the Lakehouse.

5. Stop the session.

6. Your Gold layer now contains a star schema:
   - **gold_fact_sales** (fact table)
   - **gold_dim_customers** (dimension)
   - **gold_dim_products** (dimension)
   - **gold_dim_date** (dimension)

---

## Task 6: Query the Lakehouse Using the SQL Analytics Endpoint

The SQL analytics endpoint provides read-only T-SQL access to Lakehouse tables without
any ETL. This is ideal for ad-hoc analysis and BI tool connections.

### Query Gold Tables with T-SQL

1. In the Lakehouse, switch to the **SQL analytics endpoint** view (tab on the left).

2. You will see all your tables listed under the default schema.

3. Select **New SQL query** from the toolbar.

4. Run the following query to validate your Gold layer:

   ```sql
   -- Count records in each table
   SELECT 'FactSales' AS TableName, COUNT(*) AS RowCount FROM gold_fact_sales
   UNION ALL
   SELECT 'DimCustomers', COUNT(*) FROM gold_dim_customers
   UNION ALL
   SELECT 'DimProducts', COUNT(*) FROM gold_dim_products
   UNION ALL
   SELECT 'DimDate', COUNT(*) FROM gold_dim_date;
   ```

5. Run a business query:

   ```sql
   -- Total sales by customer
   SELECT 
       c.CompanyName,
       c.Country,
       SUM(f.LineTotal) AS TotalRevenue,
       COUNT(DISTINCT f.OrderID) AS OrderCount
   FROM gold_fact_sales f
   INNER JOIN gold_dim_customers c ON f.CustomerKey = c.CustomerKey
   GROUP BY c.CompanyName, c.Country
   ORDER BY TotalRevenue DESC;
   ```

6. The SQL analytics endpoint uses the same Delta tables as PySpark without duplicating
   data. This is the power of OneLake — one copy, multiple engines.

---

## Task 7: Create a Fabric Data Warehouse and Load Data with T-SQL COPY INTO

Fabric Data Warehouse is a dedicated T-SQL engine optimized for dimensional modeling,
stored procedures, and traditional warehouse workloads. In this task you create a warehouse
and load data from the Lakehouse Gold layer using COPY INTO.

### Create a Data Warehouse

1. In your workspace, select **+ New** → **More options**.

2. Under **Data Warehouse**, select **Warehouse**.

3. Name the warehouse `WarehouseIntermediate` and select **Create**.

4. The warehouse opens with an empty schema.

### Create Schema and Tables

1. Select **New SQL query** in the warehouse.

2. Run the following DDL to create a schema and tables:

   ```sql
   -- Create Sales schema
   CREATE SCHEMA Sales;
   GO
   
   -- Create DimCustomers
   CREATE TABLE Sales.DimCustomers (
       CustomerKey BIGINT NOT NULL,
       CustomerID NVARCHAR(10),
       CompanyName NVARCHAR(255),
       ContactName NVARCHAR(255),
       ContactTitle NVARCHAR(100),
       City NVARCHAR(100),
       Country NVARCHAR(100),
       CONSTRAINT PK_DimCustomers PRIMARY KEY NONCLUSTERED (CustomerKey) NOT ENFORCED
   );
   GO
   
   -- Create DimProducts
   CREATE TABLE Sales.DimProducts (
       ProductKey BIGINT NOT NULL,
       ProductID INT,
       ProductName NVARCHAR(255),
       QuantityPerUnit NVARCHAR(100),
       UnitPrice DECIMAL(18,2),
       CategoryID INT,
       UnitCost DECIMAL(18,2),
       Discontinued INT,
       CONSTRAINT PK_DimProducts PRIMARY KEY NONCLUSTERED (ProductKey) NOT ENFORCED
   );
   GO
   
   -- Create DimDate
   CREATE TABLE Sales.DimDate (
       DateKey INT NOT NULL,
       Date DATE,
       Year INT,
       Month INT,
       Day INT,
       Quarter INT,
       DayOfWeek INT,
       CONSTRAINT PK_DimDate PRIMARY KEY NONCLUSTERED (DateKey) NOT ENFORCED
   );
   GO
   
   -- Create FactSales
   CREATE TABLE Sales.FactSales (
       OrderID INT NOT NULL,
       DateKey INT,
       CustomerKey BIGINT,
       ProductKey BIGINT,
       UnitPrice DECIMAL(18,2),
       Quantity INT,
       Discount DECIMAL(18,2),
       LineTotal DECIMAL(18,2),
       CONSTRAINT PK_FactSales PRIMARY KEY NONCLUSTERED (OrderID, ProductKey) NOT ENFORCED
   );
   GO
   ```

3. Run the query to create the tables.

### Load Data Using COPY INTO

1. Create a new SQL query.

2. Use COPY INTO to load data from the Lakehouse Gold tables. You need the OneLake path
   to your Lakehouse tables. The path format is:
   ```
   https://onelake.dfs.fabric.microsoft.com/<workspace-id>/<lakehouse-id>/Tables/<table-name>
   ```

   > **Note:** For simplicity, you can also use a shortcut or direct table reference if
   > your warehouse and lakehouse are in the same workspace.

3. Example COPY INTO syntax (adjust paths as needed):

   ```sql
   -- Load DimCustomers
   COPY INTO Sales.DimCustomers
   FROM 'https://onelake.dfs.fabric.microsoft.com/.../Tables/gold_dim_customers/'
   WITH (FILE_TYPE = 'DELTA');
   GO
   
   -- Load DimProducts
   COPY INTO Sales.DimProducts
   FROM 'https://onelake.dfs.fabric.microsoft.com/.../Tables/gold_dim_products/'
   WITH (FILE_TYPE = 'DELTA');
   GO
   
   -- Load DimDate
   COPY INTO Sales.DimDate
   FROM 'https://onelake.dfs.fabric.microsoft.com/.../Tables/gold_dim_date/'
   WITH (FILE_TYPE = 'DELTA');
   GO
   
   -- Load FactSales
   COPY INTO Sales.FactSales
   FROM 'https://onelake.dfs.fabric.microsoft.com/.../Tables/gold_fact_sales/'
   WITH (FILE_TYPE = 'DELTA');
   GO
   ```

4. If you cannot determine the exact OneLake path, an alternative approach is to use
   **Cross-database queries** in Fabric. You can query the Lakehouse tables directly
   from the Warehouse:

   ```sql
   -- Insert from Lakehouse to Warehouse (cross-database query)
   INSERT INTO Sales.DimCustomers
   SELECT * FROM [LakehouseIntermediate].[dbo].[gold_dim_customers];
   
   INSERT INTO Sales.DimProducts
   SELECT * FROM [LakehouseIntermediate].[dbo].[gold_dim_products];
   
   INSERT INTO Sales.DimDate
   SELECT * FROM [LakehouseIntermediate].[dbo].[gold_dim_date];
   
   INSERT INTO Sales.FactSales
   SELECT * FROM [LakehouseIntermediate].[dbo].[gold_fact_sales];
   ```

5. Verify the data loaded successfully:

   ```sql
   SELECT 'DimCustomers' AS TableName, COUNT(*) AS RowCount FROM Sales.DimCustomers
   UNION ALL
   SELECT 'DimProducts', COUNT(*) FROM Sales.DimProducts
   UNION ALL
   SELECT 'DimDate', COUNT(*) FROM Sales.DimDate
   UNION ALL
   SELECT 'FactSales', COUNT(*) FROM Sales.FactSales;
   ```

---

## Task 8: Build Stored Procedures for Warehouse Data Loads

Stored procedures encapsulate ETL logic for repeatability and maintainability. In this
task you create stored procedures to manage warehouse data loads.

### Create a Stored Procedure to Load FactSales

1. In the warehouse, create a new SQL query.

2. Add the following stored procedure:

   ```sql
   CREATE OR ALTER PROCEDURE Sales.sp_Load_FactSales
   AS
   BEGIN
       -- Truncate and reload (full refresh pattern)
       TRUNCATE TABLE Sales.FactSales;
       
       INSERT INTO Sales.FactSales
       SELECT * FROM [LakehouseIntermediate].[dbo].[gold_fact_sales];
       
       PRINT 'FactSales loaded successfully.';
   END;
   GO
   ```

3. Run the procedure to test:

   ```sql
   EXEC Sales.sp_Load_FactSales;
   ```

### Create a Stored Procedure to Load Dimensions

1. Add the following stored procedure for dimensions:

   ```sql
   CREATE OR ALTER PROCEDURE Sales.sp_Load_Dimensions
   AS
   BEGIN
       -- Truncate and reload dimensions (SCD Type 1)
       TRUNCATE TABLE Sales.DimCustomers;
       TRUNCATE TABLE Sales.DimProducts;
       TRUNCATE TABLE Sales.DimDate;
       
       INSERT INTO Sales.DimCustomers
       SELECT * FROM [LakehouseIntermediate].[dbo].[gold_dim_customers];
       
       INSERT INTO Sales.DimProducts
       SELECT * FROM [LakehouseIntermediate].[dbo].[gold_dim_products];
       
       INSERT INTO Sales.DimDate
       SELECT * FROM [LakehouseIntermediate].[dbo].[gold_dim_date];
       
       PRINT 'Dimensions loaded successfully.';
   END;
   GO
   ```

2. Run the procedure:

   ```sql
   EXEC Sales.sp_Load_Dimensions;
   ```

3. These stored procedures can be scheduled using Fabric Data Pipelines or called from
   orchestration tools.

> **Production Note:** The stored procedures above use a **full refresh pattern** 
> (TRUNCATE + INSERT), which is simple and appropriate for small to medium datasets. 
> For large production fact tables, consider implementing incremental load patterns with 
> watermark columns and MERGE statements to load only new or changed records.

---

## Task 9: Design a Direct Lake Semantic Model on Gold Layer Delta Tables

Direct Lake is a new storage mode in Fabric that allows Power BI to query Delta tables
in OneLake directly without importing data or using DirectQuery aggregations. This
provides near-real-time performance with zero data duplication.

### Create a Direct Lake Semantic Model

1. In your workspace, select **+ New** → **More options**.

2. Under **Power BI**, select **Semantic model**.

3. Configure the semantic model:

   | Setting | Value |
   | --- | --- |
   | Name | `SemanticModel_SalesAnalytics` |
   | Lakehouse | **LakehouseIntermediate** |
   | Tables | Select **gold_fact_sales**, **gold_dim_customers**, **gold_dim_products**, **gold_dim_date** |

4. Select **Create**.

5. The semantic model opens in the modeling view.

### Configure Relationships

1. In the **Model view**, create relationships between the fact and dimension tables:

   | From (Fact) | To (Dimension) | Cardinality |
   | --- | --- | --- |
   | gold_fact_sales[DateKey] | gold_dim_date[DateKey] | Many-to-One |
   | gold_fact_sales[CustomerKey] | gold_dim_customers[CustomerKey] | Many-to-One |
   | gold_fact_sales[ProductKey] | gold_dim_products[ProductKey] | Many-to-One |

2. Ensure all relationships are active and have the correct cardinality.

### Create Measures

1. Select the **gold_fact_sales** table.

2. In the **Modeling** ribbon, select **New measure** and add the following measures:

   ```dax
   Total Revenue = SUM(gold_fact_sales[LineTotal])
   ```

   ```dax
   Order Count = DISTINCTCOUNT(gold_fact_sales[OrderID])
   ```

   ```dax
   Avg Order Value = DIVIDE([Total Revenue], [Order Count], 0)
   ```

   ```dax
   Total Quantity = SUM(gold_fact_sales[Quantity])
   ```

3. Save the semantic model.

### Verify Direct Lake Mode

1. In the semantic model settings, confirm that the storage mode for all tables is
   **Direct Lake**.

2. Direct Lake mode ensures that queries run directly against the Delta tables in OneLake
   without importing data into Power BI's VertiPaq engine. This provides:
   - Real-time data access
   - No refresh schedules required
   - Lower storage costs
   - Automatic schema updates

---

## Task 10: Create Power BI Reports Connected to the Direct Lake Semantic Model

Now you build Power BI reports that consume the Direct Lake semantic model for live,
low-latency analytics.

### Create a Power BI Report

1. In your workspace, select **+ New** → **Report**.

2. Select the data source: **SemanticModel_SalesAnalytics**.

3. The report canvas opens with access to all tables and measures.

### Build an Executive Summary Page

1. Rename the page to **Executive Summary**.

2. Add the following visuals:

   - **Card**: Display **Total Revenue** measure.
   - **Card**: Display **Order Count** measure.
   - **Card**: Display **Avg Order Value** measure.
   - **Line chart**: X-axis = **gold_dim_date[Date]**, Y-axis = **Total Revenue**, to show revenue trend over time.

3. Format the visuals with your preferred theme.

### Build a Customer Analysis Page

1. Add a new page and name it **Customer Analysis**.

2. Add the following visuals:

   - **Table**: Display **CompanyName**, **Country**, **Total Revenue**, and **Order Count**.
   - **Bar chart**: X-axis = **Country**, Y-axis = **Total Revenue**, to show revenue by country.
   - **Donut chart**: Values = **Total Revenue**, Legend = **Country**, to show revenue distribution.

### Build a Product Performance Page

1. Add a new page and name it **Product Performance**.

2. Add the following visuals:

   - **Table**: Display **ProductName**, **QuantityPerUnit**, **Total Revenue**, and **Total Quantity**.
   - **Column chart**: X-axis = **CategoryID**, Y-axis = **Total Revenue**, to show revenue by product category.
   - **Treemap**: Group = **CategoryID**, Values = **Total Revenue**.

### Publish the Report

1. Select **File** → **Save** and name the report `Sales Dashboard – Intermediate`.

2. The report is saved in your workspace and available for sharing.

3. Because the report uses Direct Lake mode, it always reflects the latest data in the
   Lakehouse without requiring scheduled refreshes.

---

## Task 11: Validate the Medallion Architecture and Review Data Lineage

In this final task you document the medallion architecture (Bronze / Silver / Gold)
and validate data lineage across layers.

### Review the Medallion Architecture

1. Open the Lakehouse and review the folder structure:

   - **Bronze**: Raw, unprocessed data (CSV files uploaded and shortcuts)
     - bronze/customers/
     - bronze/products/
     - bronze/orders/
     - bronze/order_details/
   - **Silver**: Cleansed, validated Delta tables (silver_customers, silver_products, silver_orders, silver_order_details)
   - **Gold**: Business-level dimensional model (gold_fact_sales, gold_dim_customers, gold_dim_products, gold_dim_date)

2. Document the purpose of each layer:

   | Layer | Purpose | Format | Quality |
   | --- | --- | --- | --- |
   | Bronze | Raw ingestion from source systems | CSV, Parquet, JSON | Unvalidated |
   | Silver | Cleansed, deduplicated, conformed | Delta tables | Validated |
   | Gold | Business-ready aggregates and dimensions | Delta tables (star schema) | Curated |

### Validate Data Lineage

1. Open the **Lineage view** in your workspace:
   - Select the workspace name → **View** → **Lineage view**.

2. You should see the following flow:

   ```
   Lakehouse (Bronze Files) → PySpark Notebooks (Silver) → PySpark Notebooks (Gold)
                                                              ↓
                                               Warehouse (Sales Schema)
                                                              ↓
                                               Semantic Model (Direct Lake)
                                                              ↓
                                               Power BI Report
   ```

3. Lineage view automatically tracks dependencies between Fabric items and highlights
   the data flow.

### Verify End-to-End Refresh

1. Add a new row to the **bronze/order_details/order_details.csv** file (upload a modified version).

2. Rerun the following notebooks in sequence:
   - `bronze_to_silver_order_details`
   - `silver_to_gold_fact_sales`

3. In the Warehouse, run the stored procedure:

   ```sql
   EXEC Sales.sp_Load_FactSales;
   ```

4. Open the Power BI report. The new data should appear automatically (Direct Lake mode).

5. You have now validated the full medallion architecture with end-to-end lineage.

---

## Lab Cleanup (Optional)

To avoid ongoing charges, consider deleting resources after completing the lab:

1. In the Fabric portal, navigate to your workspace **FabricWorkspace-Intermediate**.

2. Select the workspace settings (gear icon) → **Workspace settings**.

3. Under **Other**, select **Remove this workspace**.

4. Confirm the deletion.

> **Note:** This will delete all Fabric items in the workspace but will not delete or
> stop your F2 capacity. To manage or delete the capacity, navigate to the Capacity
> settings in the Admin portal.

---

## Review

In this lab you have:

✓ Provisioned a Microsoft Fabric workspace backed by Fabric capacity and understood the SaaS-first analytics model  
✓ Built a Lakehouse on OneLake and ingested data using shortcuts, file uploads, and Dataflows Gen2  
✓ Transformed raw data with PySpark notebooks and the SQL analytics endpoint over a single OneLake copy  
✓ Designed a Fabric Data Warehouse with T-SQL COPY INTO, stored procedures, and dimensional modeling  
✓ Built a Direct Lake semantic model that reads Delta tables natively without import or DirectQuery overhead  
✓ Created Power BI reports layered directly on Fabric semantic models for live, low-latency insights  
✓ Applied medallion architecture (Bronze / Silver / Gold) to organize raw, refined, and curated data layers  

You now have hands-on experience with OneLake-first lakehouse design, PySpark transformations on Delta tables, T-SQL data warehousing in Fabric, Direct Lake semantic modeling, Power BI reporting on Fabric, and medallion architecture patterns. These skills are foundational for building modern, unified analytics platforms with Microsoft Fabric.

---

## Additional Resources

### Microsoft Learn Documentation

To deepen your understanding of the topics covered in this lab, explore these official Microsoft Learn resources:

**Dataflows Gen2:**
- [Quickstart: Create your first dataflow to get and transform data](https://learn.microsoft.com/en-us/fabric/data-factory/create-first-dataflow-gen2)
- [Ingest Data with Dataflows in Microsoft Fabric - Training Module](https://learn.microsoft.com/en-us/training/modules/use-dataflow-gen-2-fabric/)

**Lakehouse & OneLake:**
- [What is a Lakehouse?](https://learn.microsoft.com/en-us/fabric/data-engineering/lakehouse-overview)
- [OneLake, the OneDrive for data](https://learn.microsoft.com/en-us/fabric/onelake/onelake-overview)

**Data Warehouse:**
- [What is data warehousing in Microsoft Fabric?](https://learn.microsoft.com/en-us/fabric/data-warehouse/data-warehousing)
- [Ingest data into the Warehouse using COPY INTO](https://learn.microsoft.com/en-us/fabric/data-warehouse/ingest-data-copy)

**Direct Lake:**
- [Direct Lake mode in Power BI](https://learn.microsoft.com/en-us/power-bi/enterprise/directlake-overview)
- [Create a Direct Lake semantic model](https://learn.microsoft.com/en-us/fabric/data-engineering/lakehouse-power-bi-reporting)

**Medallion Architecture:**
- [Implement medallion architecture in a Lakehouse](https://learn.microsoft.com/en-us/fabric/onelake/onelake-medallion-lakehouse-architecture)

### Certification Path

Consider pursuing the **Microsoft Certified: Fabric Data Engineer Associate** certification to validate your Fabric skills. This certification covers:
- Data ingestion and transformation
- Data storage and processing
- Data monitoring and optimization
- Data security and governance

[Learn more about Fabric Data Engineer Associate certification](https://learn.microsoft.com/en-us/credentials/certifications/fabric-data-engineer-associate/)
