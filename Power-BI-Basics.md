---
  title: 'Lab Power BI Basics: Reports and Dashboards'
  module: 'Power BI Fundamentals Lab'
---

# Lab Power BI Basics – Reports and Dashboards

## Lab Introduction

In this introductory lab, you will learn the fundamentals of Power BI by building a complete end-to-end analytics solution. You will work with Power BI Desktop to connect to data, create a data model, build interactive reports, and then publish your work to the Power BI Service where you'll create and share a dashboard.

This lab will guide you through the following tasks:

- Understand the basics of Power BI Desktop and the Power BI Service
- Connect to and import data from multiple sources (OData and CSV files)
- Build a data model using star schema design with relationships
- Create interactive reports with various visualizations, filters, and slicers
- Publish your report to the Power BI Service
- Create a dashboard by pinning visuals from your report
- Share your dashboard with colleagues

## Pre-requisites

- **Power BI Desktop installed**: Download and install the free Power BI Desktop application from [Microsoft's website](https://powerbi.microsoft.com/desktop)
- **Power BI Pro license**: Both you (the creator) and anyone you share content with must have a Power BI Pro license
- **Power BI Service account**: Sign up for a free account at [https://app.powerbi.com](https://app.powerbi.com)
- **Workspace access**: Access to a workspace in the Power BI Service (you'll create or use "My Workspace" in this lab)
- **Sample data access**: Download the CSV files (products.csv and customers.csv) from the sample-data folder
- **Internet connection**: Required for accessing the Northwind OData service

## Estimated Timing: 90 minutes

## Lab Scenario

Your organization wants to analyze sales data to gain insights into product performance, customer behavior, and overall sales trends. The data currently exists in multiple locations:

- **Order transactions** and **order line items** are available through a Northwind OData service (a public demo service)
- **Product catalog** data is maintained in a CSV file
- **Customer information** is stored in a separate CSV file

Your task is to:

- Connect to these disparate data sources and bring the data into Power BI Desktop
- Create a unified data model that relates orders to customers and products
- Build interactive reports that help answer key business questions:
  - What are our total sales and how many orders have we processed?
  - Which customers are generating the most revenue?
  - Which products are our top sellers?
  - How are sales distributed geographically?
- Publish the report to the Power BI Service
- Create a dashboard that highlights the most important KPIs
- Share the dashboard with your team for collaborative decision-making

## Architecture Overview

```
Power BI Lab Architecture
========================

Data Sources
├── Northwind OData Service (https://services.odata.org/v4/northwind/northwind.svc/)
│   ├── Orders table (order header data)
│   └── Order_Details table (order line item data)
│
└── CSV Files (sample-data folder)
    ├── products.csv (product catalog)
    └── customers.csv (customer information)

                    ↓ (Get Data / Import Mode)

Power BI Desktop
├── Data Model (Star Schema)
│   ├── Fact Tables
│   │   ├── Orders (center of star - many relationships)
│   │   └── Order_Details (transactional grain)
│   │
│   ├── Dimension Tables
│   │   ├── Customers (one-to-many → Orders)
│   │   ├── Products (one-to-many → Order_Details)
│   │   └── Products (one-to-many → Orders via Order_Details)
│   │
│   └── Relationships (one-to-many, single direction filter)
│       ├── Customers[CustomerID] → Orders[CustomerID]
│       ├── Products[ProductID] → Order_Details[ProductID]
│       └── Orders[OrderID] → Order_Details[OrderID]
│
├── Report Canvas (3 pages)
│   ├── Page 1: Sales Overview
│   │   ├── KPI Cards (Total Sales, Order Count, Avg Order Value)
│   │   ├── Sales Trend Line Chart
│   │   └── Orders by Country Bar Chart
│   │
│   ├── Page 2: Customer Analysis
│   │   ├── Top Customers Table
│   │   ├── Customer Map (by Country/City)
│   │   └── Orders by Customer Bar Chart
│   │
│   └── Page 3: Product Performance
│       ├── Product Sales Table
│       ├── Top Products Bar Chart
│       └── Category Slicer
│
└── Visualizations & Interactivity
    ├── Filters (report, page, visual level)
    ├── Slicers (date, category, country)
    └── Cross-filtering between visuals

                    ↓ (File → Publish to Power BI)

Power BI Service (https://app.powerbi.com)
├── Workspace
│   └── Published Report (interactive, shareable)
│
├── Dashboard
│   ├── Pinned Visuals from Report
│   │   ├── Total Sales Card
│   │   ├── Orders by Country Chart
│   │   ├── Top Customers Table
│   │   └── Top Products Chart
│   │
│   └── Real-time Interaction
│       └── Click tiles to navigate to source report
│
└── Sharing & Collaboration
    └── Share Dashboard with Pro users → Email notification with link

Data Flow Summary:
1. Connect: OData Service + CSV Files → Power BI Desktop
2. Model: Build star schema with relationships
3. Visualize: Create 3 report pages with interactive visuals
4. Publish: Upload report to Power BI Service workspace
5. Dashboard: Pin key visuals to a new dashboard
6. Share: Distribute dashboard to team members (requires Pro licenses)
```

## Job Skills

By completing this lab, you will gain hands-on experience with the following Power BI skills:

- Task 1: Introduction to Power BI Desktop and initial setup (10 minutes)
- Task 2: Connect to and import data from multiple sources (20 minutes)
- Task 3: Build a data model with relationships using star schema design (15 minutes)
- Task 4: Create interactive reports with visualizations, filters, and slicers (25 minutes)
- Task 5: Publish your report to the Power BI Service (10 minutes)
- Task 6: Create and share a dashboard in the Power BI Service (10 minutes)

---

## Task 1: Introduction to Power BI Desktop and Initial Setup

Power BI Desktop is a free Windows application that allows you to connect to data, transform it, and create rich interactive reports. The Power BI Service (the online component) is where you publish, share, and collaborate on your reports and dashboards.

> **What is Power BI?** Power BI is a business analytics service by Microsoft that provides interactive visualizations and business intelligence capabilities. It consists of:
> - **Power BI Desktop**: A free Windows application for creating reports (this is where you'll do most of your work)
> - **Power BI Service**: An online SaaS service for publishing, sharing, and collaborating on reports
> - **Power BI Mobile**: Mobile apps for iOS, Android, and Windows devices

### Launch Power BI Desktop

1. Open **Power BI Desktop** from your Start menu or desktop shortcut. Select **Blank report**

2. If you see a welcome screen or tutorial, you can close it or follow along briefly to get familiar with the interface.

3. You should now see a blank canvas with the following main components:

   | Component | Location | Purpose |
   | --- | --- | --- |
   | **Ribbon** | Top | Contains tabs (Home, Insert, Modeling, View) with tools and commands |
   | **Report canvas** | Center | The main workspace where you build and arrange visualizations |
   | **Visualizations pane** | Right side | Shows visualization types and formatting options |
   | **Fields pane** | Far right | Lists all tables and fields from your data model |
   | **Filters pane** | Right side (may be collapsed) | Apply filters at report, page, or visual level |
   | **Pages tabs** | Bottom | Navigate between report pages |

### Sign In to Power BI Service

Before you can publish reports, you need to sign in to the Power BI Service at least once.

1. In Power BI Desktop, select **File** → **Sign in**.

2. Enter your organizational account credentials (the account with your Power BI Pro license).

3. Complete the sign-in process. You should see your account name in the upper-right corner of Power BI Desktop.

> **Important:** Both the creator (you) and anyone you share content with must have a Power BI Pro license. The Power BI Service will prompt recipients to upgrade if they don't have the appropriate license.

### Verify Workspace Access

1. Open a web browser and navigate to [https://app.powerbi.com](https://app.powerbi.com).

2. Sign in with the same credentials you used in Power BI Desktop.

3. On the left navigation pane, you should see **Workspaces**.

4. Expand **Workspaces** and verify you can see **My workspace**. This is your personal workspace where you can publish content.

   > **What is a workspace?** A workspace is a collaborative environment where you and your team can create and share dashboards, reports, and datasets. "My workspace" is your personal workspace, while team workspaces allow multiple users to collaborate.

5. You can close the browser for now. We'll return to the Power BI Service in Task 5.

### Save Your Power BI Desktop File

1. In Power BI Desktop, select **File** → **Save As**.

2. Select **Browse this device** and choose a location on your computer and name the file **SalesAnalysis.pbix**.

3. Select **Save**.

> **Tip:** Save your work frequently as you progress through this lab using **Ctrl + S** or **File** → **Save**.

---

## Task 2: Connect to and Import Data from Multiple Sources

Power BI Desktop can connect to a wide variety of data sources including databases, cloud services, files, and web services. In this task, you'll connect to both an OData service and CSV files to bring sales data into Power BI.

> **Import Mode vs. DirectQuery:** When connecting to data, Power BI offers different connection modes:
> - **Import Mode** (default): Data is copied into Power BI and stored in the .pbix file. Best for smaller datasets and faster performance.
> - **DirectQuery**: Power BI queries the data source directly each time a visual is refreshed. Best for very large datasets or real-time data.
> 
> For this lab, we'll use **Import Mode** which is ideal for learning and for datasets of this size.

### Connect to the Northwind OData Service for Orders

The Northwind OData service is a publicly available demo service that provides sample order data.

1. In Power BI Desktop, on the **Home** ribbon, select **Get data**.

2. In the **Get Data** dialog, search for **OData** or scroll to find **OData Feed** in the list.

3. Select **OData Feed** and then select **Connect**.

4. In the **OData Feed** dialog, enter the following URL in the **URL** box:
   ```
   https://services.odata.org/v4/northwind/northwind.svc/
   ```

5. Select **OK**.

6. If prompted for authentication, select **Anonymous** and then **Connect** (this is a public service that doesn't require credentials).

7. The **Navigator** window appears showing all available tables from the Northwind service. You should see tables like Customers, Employees, Orders, Order_Details, Products, and more.

8. In the Navigator window, select the checkbox next to **Orders** to see a preview of the data.

   The Orders table contains order header information including:
   - OrderID (unique identifier for each order)
   - CustomerID (links to customer)
   - OrderDate (when the order was placed)
   - Freight (shipping cost)
   - ShipCountry, ShipCity (delivery location)

9. Also select the checkbox next to **Order_Details** (note the underscore).

   The Order_Details table contains order line items with:
   - OrderID (links back to Orders table)
   - ProductID (links to product)
   - UnitPrice (price per unit)
   - Quantity (number of units ordered)
   - Discount (discount percentage applied)

10. With both tables selected, select **Transform Data** (not Load).

    > **Why Transform Data?** The Transform Data button opens Power Query Editor where you can clean, shape, and prepare your data before loading it into the data model. This is a best practice for ensuring data quality.

### Clean and Prepare Orders Data in Power Query Editor

The Power Query Editor opens with your two queries listed on the left.

1. In the **Queries** pane on the left, select the **Orders** query.

2. Review the columns in the preview pane. Notice that some columns like RequiredDate and ShippedDate are visible.

3. Let's remove some unnecessary columns to simplify our model. Select the **Home** tab, then select **Choose Columns**.

4. In the **Choose Columns** dialog, **uncheck** the Select All Columns option, then check only the following columns to keep:
   - OrderID
   - CustomerID
   - OrderDate
   - Freight
   - ShipCity
   - ShipCountry

5. Select **OK** to apply the column selection.

6. Select the ShipCountry column, then go to the **Transform** tab and select **Format** → **UPPERCASE** to convert all text to uppercase.

### Clean and Prepare Order Details Data

1. In the **Queries** pane, select the **Order_Details** query.

2. Review the columns in the preview pane. You may notice columns named **Order** and **Product**. These are navigation properties from the OData service that contain nested table data.

   > **Why remove these columns?** The OData service automatically creates navigation columns when tables have relationships. However, you already have the foreign keys (OrderID and ProductID) needed to build proper relationships in Power BI's data model. Keeping these nested columns would make your data model unnecessarily complex and could cause performance issues.

3. Remove the **Order** column:
   - Right-click the **Order** column header
   - Select **Remove**

4. Remove the **Product** column:
   - Right-click the **Product** column header
   - Select **Remove**

5. Your **Order_Details** query should now contain only these five essential fields:
   - OrderID
   - ProductID
   - UnitPrice
   - Quantity
   - Discount

### Connect to CSV File for Products Data

Now let's add the product catalog data from a CSV file.

1. Download the **products.csv** file from the sample-data folder of this repository.

2. Head back to the Power BI Desktop. In Power Query Editor, select **Home** → **New Source** → **More**.

3. In the **Get Data** dialog, search for **CSV** or select **Text/CSV** from the list.

4. Select **Text/CSV** and then select **Connect**.

5. Select the **products.csv** file and then select **Open**.

6. Power BI automatically detects the file structure. You should see a preview showing:
   - productID
   - productName
   - quantityPerUnit
   - unitPrice
   - discontinued
   - categoryID
   - unitCost

7. Verify that the data looks correct (headers are properly detected, data types look appropriate).

8. Select **OK** to add this table to your Power Query Editor.

9. The **products** query now appears in your Queries pane on the left.

10. In the **products** query, let's remove the **unitCost** column since we won't need it for this analysis:
    - Right-click the **unitCost** column header and select **Remove**.

### Connect to CSV File for Customers Data

1. Download the **customers.csv** file from the sample-data folder of this repository.

2. Head back to Power BI Desktop. In Power Query Editor, select **Home** → **New Source** → **Text/CSV**.

3. Select **customers.csv** and select **Open**.

4. Review the preview showing customer information:
   - customerID
   - companyName
   - contactName
   - contactTitle
   - city
   - country

5. Select **OK** to add this to Power Query Editor.

6. The **customers** query now appears in your Queries pane.

7. Verify if the headers of the table is correct. If not, select the **Use First Row as Headers** button in the ribbon found in the **Home** tab.

8. Let's clean up the customer names. Select the **contactName** column.

9. On the **Transform** tab, select **Format** → **Trim** to remove any leading or trailing spaces.

10. Also select **Format** → **Clean** to remove any non-printable characters.


### Rename Queries for Consistency

Let's rename our queries to follow a consistent naming convention (PascalCase without underscores).

1. In the **Queries** pane, right-click the **Orders** query and select **Rename**.

2. The name is already good (Orders), so just press **Enter**.

3. Right-click the **Order_Details** query and select **Rename**.

4. Change the name to **OrderDetails** (remove the underscore) and press **Enter**.

5. Right-click the **products** query and select **Rename**.

6. Change the name to **Products** (capitalize the P) and press **Enter**.

7. Right-click the **customers** query and select **Rename**.

8. Change the name to **Customers** (capitalize the C) and press **Enter**.

### Load Data into Power BI Desktop

1. Review your four queries in the Queries pane:
   - Orders
   - OrderDetails
   - Products
   - Customers

2. On the **Home** tab in Power Query Editor, select **Close & Apply**.

3. Power BI will now load all four tables into your data model. You'll see a progress dialog as the data is imported.

4. Once loading is complete, you're returned to the Power BI Desktop report canvas.

5. In the **Fields** pane on the right, you should now see all four tables listed with their columns:
   - Customers
   - Orders
   - OrderDetails
   - Products

> **Congratulations!** You've successfully connected to and imported data from multiple sources. Your data is now ready to be modeled and analyzed.

---

## Task 3: Build a Data Model with Relationships

A well-designed data model is the foundation of effective reporting in Power BI. In this task, you'll create relationships between your tables using star schema design principles, where fact tables (Orders, OrderDetails) sit at the center and dimension tables (Customers, Products) provide context.

> **What is a Star Schema?** A star schema is a database design pattern optimized for reporting and analytics. It consists of:
> - **Fact Tables**: Contain measurements and metrics (Orders, OrderDetails)
> - **Dimension Tables**: Contain descriptive attributes (Customers, Products)
> - **Relationships**: One-to-many relationships from dimensions to facts
> 
> This design makes queries fast and reports easy to build.

### Switch to Model View

1. On the left side of Power BI Desktop, select the **Model** view icon (it looks like three connected boxes).

2. You should now see a diagram showing your four tables as boxes with their columns listed inside.

   > **Auto-detected Relationships:** Power BI often automatically detects and creates relationships based on matching column names. You may already see lines connecting your tables. If so, that's great! The following steps will help you verify and understand these relationships. If you don't see any relationships yet, we'll create them manually.

3. Arrange the tables by dragging them around the canvas so you can see them clearly:
   - Place **Customers** on the left
   - Place **Orders** in the center
   - Place **OrderDetails** to the right of Orders
   - Place **Products** on the far right

### Create Relationship: Customers to Orders

Let's create the first relationship connecting customers to their orders.

> **Note:** Power BI may have already auto-detected and created these relationships for you based on matching column names (CustomerID, OrderID, ProductID). If you see relationship lines already connecting your tables, you can skip the manual creation steps and proceed directly to step 4 to verify the relationship properties. The following steps show you how to manually create relationships if needed.

1. In the **Customers** table, locate the **customerID** column.

2. **If a relationship line doesn't already exist**, click and drag the **customerID** column from the **Customers** table to the **CustomerID** column in the **Orders** table. If a line already exists, proceed to step 4.

3. A line appears between the two tables, indicating a relationship has been created.

4. Double-click the relationship line to view its properties.

5. The **New relationship** dialog appears. Verify the following settings:

   | Setting | Value |
   | --- | --- |
   | From: Table (Column) | Orders (CustomerID) |
   | To: Table (Column) | Customers (CustomerID) |
   | Cardinality | Many to one (\*:1) |
   | Cross filter direction | Single |
   | Make this relationship active | ✓ Checked |

   > **Understanding Cardinality:** 
   > - **Many to one (\*:1)**: Many orders can belong to one customer
   > - The "many" side is indicated by an asterisk (\*)
   > - The "one" side is indicated by a 1
   > - This is the most common relationship type in Power BI

6. Select **OK** to close the dialog.

7. You should now see a relationship line with **1** on the Customers side and **\*** on the Orders side.

### Create Relationship: Orders to OrderDetails

Now connect the Orders table to the OrderDetails table.

1. **If a relationship line doesn't already exist**, drag the **OrderID** column from the **Orders** table to the **OrderID** column in the **OrderDetails** table. If a line already exists, proceed to step 3.

2. A relationship line appears.

3. Double-click the relationship line to verify the settings:

   | Setting | Value |
   | --- | --- |
   | From: Table (Column) | Orders (OrderID) |
   | To: Table (Column) | OrderDetails (OrderID) |
   | Cardinality | One to many (1:\*) |
   | Cross filter direction | Single |
   | Make this relationship active | ✓ Checked |

4. Select **OK**.

### Create Relationship: Products to OrderDetails

Finally, connect products to order details.

1. **If a relationship line doesn't already exist**, drag the **productID** column from the **Products** table to the **ProductID** column in the **OrderDetails** table. If a line already exists, proceed to step 3.

2. A relationship line appears.

3. Double-click the relationship line to verify:

   | Setting | Value |
   | --- | --- |
   | From: Table (Column) | Products (productID) |
   | To: Table (Column) | OrderDetails (ProductID) |
   | Cardinality | One to many (1:\*) |
   | Cross filter direction | Single |
   | Make this relationship active | ✓ Checked |

4. Select **OK**.

### Verify Your Data Model

Your completed data model should now look like this:

```
Customers (1) ──────────> (*) Orders (1) ──────────> (*) OrderDetails (*) <────────── (1) Products
   [Dimension]              [Fact]                        [Fact]                       [Dimension]
```

The relationships allow filters to flow from dimension tables to fact tables:
- Selecting a customer will filter all their orders and order details
- Selecting a product will filter all order details for that product
- This is called **filter propagation** and is fundamental to how Power BI reports work

### Review the Relationships in the Relationships Pane

1. On the **Home** tab in Model view, select **Manage relationships**.

2. The **Manage relationships** dialog shows all three relationships you created:
   - Customers (customerID) to Orders (CustomerID) - Active
   - Orders (OrderID) to OrderDetails (OrderID) - Active
   - Products (productID) to OrderDetails (ProductID) - Active

3. All relationships should show **Active** status.

4. Select **Close**.

### Test the Model with a Quick Visual

Let's verify that the relationships work correctly by creating a quick test visual.

1. Select the **Report** view icon on the left (it looks like a bar chart).

2. In the **Fields** pane, expand the **Customers** table and check the box next to **country**.

3. Expand the **OrderDetails** table and check the box next to **Quantity**.

4. Select the Table visual from the Visualizations pane. A table appears showing countries and the total quantity of products ordered.

5. This confirms that your relationships are working correctly - the filter is propagating from Customers → Orders → OrderDetails!

6. You can delete this test visual (select it and press **Delete**) or keep it for reference.

> **Congratulations!** You've successfully built a star schema data model with proper relationships. This model is now optimized for creating reports and analyzing your sales data.

---

## Task 4: Create Interactive Reports in Power BI Desktop

Now that your data model is ready, it's time to create interactive visualizations that answer business questions. You'll build three report pages, each focusing on different aspects of the sales data.

> **Best Practice:** Organize your reports into logical pages that focus on specific themes or audiences. This makes it easier for users to find the information they need.

### Page 1: Sales Overview

Let's create the first page showing high-level sales KPIs and trends.

#### Create KPI Cards

1. Ensure you're in **Report** view (select the bar chart icon on the left if needed).

2. You should see a blank report page labeled "Page 1" at the bottom.

3. Let's create a card showing total revenue. Select the Card icon in the Visualizations pane.

4. A card visual appears on your canvas.

5. From the **Fields** pane, expand **OrderDetails** and drag **UnitPrice** to the card visual.

6. The card now shows a sum of all unit prices, but we need to calculate the actual revenue. Click on the card to select it.

7. In the **Visualizations** pane, under **Fields**, you'll see **Sum of UnitPrice** in the Values well.

8. Let's create a measure instead. Right-click on the **OrderDetails** table in the Fields pane and select **New measure**.

9. In the formula bar at the top, type the following DAX formula:
   ```
   Total Revenue = SUMX(OrderDetails, OrderDetails[UnitPrice] * OrderDetails[Quantity] * (1 - OrderDetails[Discount]))
   ```

10. Press **Enter** to create the measure.

11. Now select your card visual again, remove the **UnitPrice** field, and drag the new **Total Revenue** measure to the card.

12. The card displays the total revenue across all orders.

13. Format the card:
    - With the card selected, go to the **Format** pane (paint roller icon)
    - Select the Visual tab
    - Expand **Callout**
    - Expand **Apply settings to**
    - Select **Total Revenue** in the **Cards** dropdown
    - Expand **Value**
    - Set **Display units** to **Thousands**
    - Set **Value decimal places** to **0**
    - Turn **Label** to **Off** (optional, hides the "Total Revenue" label on the card)

14. Resize the card to make it smaller and position it in the upper-left corner of the canvas.

15. Add a title to the card:
    - In the Format pane, expand **General** → **Title**
    - Turn **Title** to **On**
    - Expand **Title**
    - Change **Text** to "Total Revenue"

#### Create Order Count Card

1. Under the Visualizations pane, select Add data to your visual tab. Add another card visual to the canvas.

2. Right-click the **Orders** table in the Fields pane and select **New measure**.

3. Type the following DAX formula:
   ```
   Total Orders = COUNTROWS(Orders)
   ```

4. Press **Enter**.

5. Drag the **Total Orders** measure to your new card visual.

6. Format the card similarly (display units: Auto, no decimal places).

7. Add title: "Total Orders"

8. Position this card to the right of the Total Revenue card.

#### Create Average Order Value Card

1. Add a third card visual.

2. Right-click the **Orders** table and select **New measure**.

3. Type the following DAX formula:
   ```
   Average Order Value = DIVIDE([Total Revenue], [Total Orders], 0)
   ```

4. Press **Enter**.

5. Drag the **Average Order Value** measure to the card.

6. Format: Display units None, 2 decimal places.

7. Add title: "Average Order Value"

8. Position this card to the right of Total Orders.

#### Create Sales Trend Line Chart

1. Below the KPI cards, add a **Line chart** visual from the Visualizations pane.

2. Configure the visual:
   - **X-axis**: Drag **OrderDate** from the Orders table 
   - **Y-axis**: Drag the **Total Revenue** measure
   - **Legend**: Leave empty for now

3. Format the line chart:
   - Select the visual
   - Go to Format pane → **X-axis**
   - Turn **Title** to **On**
   - Change **Title text** to "Order Date"
   - Go to **Y-axis**
   - Turn **Title** to **On**
   - Change **Title text** to "Revenue"

4. Resize the chart to take up about 40% of the page width.

#### Create Orders by Country Bar Chart

1. Add a **Clustered bar chart** visual (horizontal bars).

2. Configure:
   - **Y-axis**: Drag **ShipCountry** from Orders table
   - **X-axis**: Drag **Total Orders** measure

3. Sort the chart by Total Orders descending:
   - Select the visual
   - Click the **More options** (...) at the top-right of the visual
   - Select **Sort by** → **Sort descending**

4. Add title: "Orders by Country"
   - In Format pane, go to the General Tab
   - Turn **Title** to **On**
   - Change **Text** to "Orders by Country"

5. Position this chart to the right of the line chart.

#### Add a Date Slicer

1. Add a **Slicer** visual from the Visualizations pane.

2. Drag **OrderDate** from the Orders table to the **Field** well.

3. Change the slicer style:
   - Select the slicer
   - Go to Format pane → Visual → Slicer settings
   - Expand **Options**
   - Select **Between** (shows a slider with two handles for date range selection)

4. Position the slicer at the bottom of the page, spanning the full width.

5. Add title: "Select Date Range"
   - Select the General tab in the Format pane
   - Turn **Title** to **On**
   - Change **Text** to "Select Date Range"

#### Rename Page 1

1. At the bottom of the screen, right-click **Page 1** and select **Rename**.

2. Type **Sales Overview** and press **Enter**.

### Page 2: Customer Analysis

Let's create a second page focused on customer insights.

1. At the bottom of the screen, select the **+** icon to add a new page.

2. A new blank page (**Page 2**) is created.

#### Create Top Customers Table

1. Add a **Table** visual from the Visualizations pane.

2. Configure the table with these fields from the Customers table:
   - companyName
   - country
   - city

3. Add the **Total Revenue** measure to the table.

4. Add the **Total Orders** measure to the table.

5. Sort the table by Total Revenue descending:
   - Click the **More options** (...) on the visual
   - Select **Sort by** → **Total Revenue**
   - Select **Sort descending**

6. Resize the table to take up the left half of the page.

7. Add title: "Top Customers by Revenue"
   - In Format pane, go to General Tab
   - Turn **Title** to **On**
   - Change **Text** to "Top Customers by Revenue"

#### Create Customer Treemap

1. Add a **Treemap** visual from the Visualizations pane.

2. Configure:
   - **Category**: Drag **country** from Customers table
   - **Values**: Drag **Total Revenue** measure

3. The treemap displays rectangles showing revenue by country, with larger rectangles representing higher revenue.

4. Position the treemap on the right side of the page, taking up the remaining space.

5. Add title: "Revenue by Customer Location"
   - In Format pane, go to General Tab
   - Turn **Title** to **On**
   - Change **Text** to "Revenue by Customer Location"

#### Add Country Slicer

1. Add a **Slicer** visual.

2. Drag **country** from Customers table to the Field well.

3. Format:
   - Go to Format pane → Visual → Slicer settings → Selection
   - Enable **Show "Select all" option**
   
4. Position the slicer at the bottom of the page.

5. Add title: "Filter by Country"
   - In Format pane, go to General Tab
   - Turn **Title** to **On**
   - Change **Text** to "Filter by Country"

#### Rename Page 2

1. Right-click **Page 2** at the bottom and select **Rename**.

2. Type **Customer Analysis** and press **Enter**.

### Page 3: Product Performance

Create a third page focusing on product-level insights.

1. Add a new page by clicking the **+** icon at the bottom.

#### Create Product Sales Table

1. Add a **Table** visual.

2. Configure with these fields:
   - **productName** from Products table
   - **categoryID** from Products table (shows product category)
   - **Total Revenue** measure
   - **Quantity** from OrderDetails table (Power BI will sum it automatically)

3. Sort by Total Revenue descending.
   - Click the **More options** (...) on the visual
   - Select **Sort by** → **Total Revenue**
   - Select **Sort descending**

4. Resize to take up about 60% of the page width on the left side.
   
5. Add title: "Product Sales Performance"
   - In Format pane, go to General Tab
   - Turn **Title** to **On**
   - Change **Text** to "Product Sales Performance"

#### Create Top Products Column Chart

1. Add a **Clustered column chart** visual (vertical bars).

2. Configure:
   - **X-axis**: Drag **productName** from Products table
   - **Y-axis**: Drag **Total Revenue** measure

3. Show only top products:
   - Select the visual
   - Go to **Filters** pane on the right
   - Expand the **productName** filter
   - Change **Filter type** to **Top N**
   - Set **Show items** to **Top 10**
   - Set **By value** to **Total Revenue**
   - Select **Apply filter**

4. Format the chart:
   - Rotate X-axis labels for better readability
   - Go to Format pane → **Visual** → **X-axis**
   - Set **Text size** to 8

5. Position on the right side of the page.

6. Add title: "Top 10 Products by Revenue"
   - In Format pane, go to General Tab
   - Turn **Title** to **On**
   - Change **Text** to "Top 10 Products by Revenue"

#### Add Category Slicer

1. Add a **Slicer** visual.

2. Drag **categoryID** from Products table to the Field well.

3. Format as a vertical list.
   - Go to Format pane → Visual → Slicer settings → Options
   - Select **Vertical list** (shows all categories in a list)

4. Position at the bottom of the page.

5. Add title: "Filter by Category"
   - In Format pane, go to General Tab
   - Turn **Title** to **On**
   - Change **Text** to "Filter by Category"

#### Rename Page 3

1. Right-click **Page 3** and select **Rename**.

2. Type **Product Performance** and press **Enter**.

### Test Interactivity

Let's verify that your visualizations interact with each other correctly.

1. Go back to the **Sales Overview** page.

2. Click on one of the bars in the "Orders by Country" chart (e.g., USA).

3. Notice how all other visuals on the page automatically filter to show only data for that country. This is called **cross-filtering**.

4. Click the same bar again to deselect and reset the filters.

5. Go to the **Customer Analysis** page.

6. Select a country in the country slicer.

7. The table and treemap update to show only customers and data from that country.

8. Test the **Product Performance** page by selecting a category in the slicer.

9. The table and column chart update to show only products from that category.

> **Cross-filtering:** When you click on a visual element (bar, slice, data point), Power BI automatically filters related visuals on the same page. You can control this behavior using Format → Edit interactions.

### Save Your Work

1. Navigate to the Sales Overview Page. Press **Ctrl + S** to save your Power BI Desktop file.

> **Congratulations!** You've created a complete three-page interactive report with KPIs, trends, tables, charts, treemaps, and slicers. Your report is now ready to publish to the Power BI Service!

---

## Task 5: Publish Your Report to the Power BI Service

The Power BI Service is the online platform where you share, collaborate, and consume Power BI content. Publishing your report makes it accessible from anywhere via a web browser.

> **What Happens When You Publish?** Publishing uploads your .pbix file to the Power BI Service, including:
> - Your data model (tables, relationships, measures)
> - All report pages and visualizations
> - Any bookmarks or page-level filters
> 
> The published report will be refreshed with the data as it exists at the time of publishing.

### Publish the Report

1. In Power BI Desktop, ensure your file is saved (**Ctrl + S**).

2. On the **Home** ribbon, select **Publish**.

3. If prompted to save changes, select **Save**.

4. The **Publish to Power BI** dialog appears showing your available workspaces.

5. Select **My workspace** (or another workspace you have access to).

   > **Note:** "My workspace" is your personal workspace. In a team environment, you would typically publish to a shared workspace where colleagues can access the content.

6. Select **Select** to begin the publishing process.

7. Power BI uploads your report and data to the service. You'll see a progress dialog.

8. When publishing completes, you'll see a **Success!** message with a link to your report.

9. Select the **Open 'SalesAnalysis.pbix' in Power BI** link to open your report in the browser.

   > **Tip:** You can also navigate to [https://app.powerbi.com](https://app.powerbi.com) manually and find your report in "My workspace."

### Explore Your Report in the Power BI Service

Your report opens in the Power BI Service in your web browser.

1. You should see the same report you created in Power BI Desktop, starting with the **Sales Overview** page.

2. Test the interactivity:
   - Click on different elements in the visuals
   - Use the slicers to filter data
   - Navigate between pages using the page tabs at the bottom

3. Notice the toolbar at the top with options like:
   - **Edit**: Opens the report in editing mode (browser-based editing)
   - **Share**: Share the report with others
   - **Export**: Export the report to PowerPoint or PDF
   
### Explore the Workspace

1. In the left navigation pane, select **My workspace**.

2. You should see two items:
   - **SalesAnalysis** (Type Report) - The report you just published
   - **SalesAnalysis** (Type Semantic model) - The data model

   > **Report vs. Semantic model:** 
   > - The **Semantic model** contains your data model (tables, relationships, measures)
   > - The **Report** contains your visualizations and pages
   
3. Get Quick Insights:
   - Hover over the SalesAnalysis report
   - Click the **More options** (...) that appears
   - Select **Quick insights**
   - Power BI runs automated analysis on your data and generates insights like trends, outliers, and correlations.

### Understanding Refresh

1. Go back to your workspace and select the **SalesAnalysis** Semantic model.

2. Select **Refresh now** to manually refresh the data from your original sources.

   > **Note:** For this lab, the data sources are static (CSV files and a demo OData service), so refreshing won't change the data. In a production environment with live data sources, you would schedule automatic refreshes to keep your reports up to date.

3. You can also select **Schedule refresh** to set up automatic daily or weekly refreshes.

4. For now, we'll skip scheduling refresh since our data sources are for learning purposes.

> **Congratulations!** Your report is now published to the Power BI Service and accessible from any device with a web browser. Next, you'll create a dashboard to highlight the most important insights.

---

## Task 6: Create and Share a Dashboard in the Power BI Service

Dashboards in Power BI provide a single-page view of your most important metrics and insights. Unlike reports (which can have multiple pages), a dashboard is always a single page composed of tiles pinned from one or more reports.

> **Report vs. Dashboard:**
> - **Report**: Multi-page, interactive, created in Power BI Desktop
> - **Dashboard**: Single-page, tiles pinned from reports, created only in Power BI Service
> - **Use case**: Dashboards are great for executives and stakeholders who want quick access to KPIs without navigating through multiple report pages

### Create a New Dashboard

1. In the Power BI Service, open your **SalesAnalysis** report (if it's not already open).

2. Navigate to the **Sales Overview** page.

3. Let's pin the Total Revenue card to a new dashboard:
   - Hover over the **Total Revenue** card visual
   - Click the **Pin** icon (looks like a pushpin) that appears in the top-right corner of the visual

4. The **Pin to dashboard** dialog appears.

5. Select **New dashboard**.

6. Enter the name: **Sales Performance Dashboard**

7. Select **Pin**.

8. A confirmation message appears saying "Pinned to dashboard" with a link to view it.

### Pin Additional Visuals

Let's add more tiles to our dashboard from different report pages.

1. Remain on the **Sales Overview** page.

2. Pin the **Total Orders** card:
   - Hover over the card
   - Click the **Pin** icon
   - Select **Existing dashboard**
   - Choose **Sales Performance Dashboard**
   - Select **Pin**

3. Pin the **Average Order Value** card using the same process.

4. Pin the **Orders by Country** bar chart.

5. Navigate to the **Customer Analysis** page.

6. Pin the **Top Customers by Revenue** table.

7. Pin the **Revenue by Customer Location** treemap.

8. Navigate to the **Product Performance** page.

9. Pin the **Top 10 Products by Revenue** chart.

### View and Arrange Your Dashboard

1. In the left navigation pane, select **My workspace**.

2. You should now see your **Sales Performance Dashboard** in the list.

3. Click on the dashboard name to open it.

4. Your dashboard displays all the tiles you pinned from your report, arranged in a default layout.

5. Let's rearrange the tiles for a better layout:
   - Drag tiles to rearrange them
   - Resize tiles by dragging the corner handles
   - Suggested layout:
     - Row 1: Three KPI cards (Total Revenue, Total Orders, Avg Order Value) side by side
     - Row 2: Orders by Country chart (left half) and Revenue by Customer Location treemap (right half)
     - Row 3: Top Customers by Revenue table (left half) and Top Products by Revenue table (right half)


### Interact with Dashboard Tiles

Dashboard tiles are interactive:

1. Click on any tile on your dashboard (e.g., click the Total Revenue card).

2. Power BI navigates you back to the source report and the specific page where that visual exists.

3. This allows dashboard viewers to quickly drill into details when they see something interesting.

4. Use the browser's back button  to return to the dashboard.

### Share Your Dashboard

Now let's share the dashboard with colleagues.

1. On the dashboard, select **Share** at the top of the screen.

2. The **Share dashboard** dialog appears.

3. In the **Enter a name or email address** field, enter the email address of someone you want to share with.

   > **Important:** The recipient must have a Power BI Pro license to view shared content and must be in the same organization (same email domain). If they don't have access, they'll receive an error when trying to view the dashboard.

4. Review the sharing options:
   - ☑ **Allow recipients to share your dashboard** - Check this if you want recipients to be able to share with others
   - ☑ **Allow recipients to build content with the data associated with this dashboard** - Check this if you want recipients to create new reports using your dataset
   - ☑ **Send an email notification** - Check this to send an email with a link to the dashboard

5. Select **Grant access**.

6. The recipient receives an email notification with a link to access the dashboard.

### Manage Dashboard Access

1. Select **Manage permissions** on the dashboard (or **More options** → **Manage permissions**).

2. This shows everyone who has access to the dashboard.

3. You can:
   - Add more people
   - Remove access for specific users
   - Change permissions (like revoking share or build rights)

4. Select **Close** when done reviewing.

### Set Up Dashboard Alerts (Optional)

You can configure alerts to notify you when data reaches certain thresholds.

1. Hover over the **Total Revenue** tile.

2. Click the **More options** (...) at the top-right of the tile.

3. Select **Manage alerts**.

4. Select **+ Add alert rule**.

5. Configure the alert:
   - **Condition**: Above
   - **Threshold**: Enter a value (e.g., 1,000,000)
   - **Notification**: Email
   - **Maximum**: Every 24 hours

6. Select **Save and close**.

7. You'll receive an email notification when the total revenue exceeds your threshold.

> **Note:** Alerts only work with certain visual types (cards, gauges, KPIs) and only for data that changes (not static data like in this lab).

### Subscribe to the Dashboard (Optional)

Subscriptions send regular email snapshots of your dashboard.

1. On the dashboard, select **Subscribe** (or **More options** → **Subscribe**).

2. Configure the subscription:
   - **Frequency**: Daily or Weekly
   - **Time**: Select a specific time
   - **Subject**: Customize the email subject line
   - **Message**: Add a custom message

3. Select **Save and close**.

4. You (and anyone you add) will receive email snapshots of the dashboard at the specified frequency.

> **Congratulations!** You've successfully created a dashboard in the Power BI Service, arranged tiles for optimal viewing, and shared it with colleagues. Your team can now access real-time sales insights from anywhere!

---

## Lab Summary

In this lab, you've learned the fundamentals of Power BI by completing a comprehensive end-to-end workflow:

### What You Accomplished

✅ **Task 1**: Set up Power BI Desktop and signed into the Power BI Service
✅ **Task 2**: Connected to multiple data sources (OData service and CSV files) and imported data
✅ **Task 3**: Built a star schema data model with relationships between customers, orders, and products
✅ **Task 4**: Created three interactive report pages with various visualizations:
   - Sales Overview: KPIs, trends, and geographic analysis
   - Customer Analysis: Top customers and location-based insights
   - Product Performance: Product sales and category filtering
✅ **Task 5**: Published your report to the Power BI Service
✅ **Task 6**: Created and shared a dashboard highlighting key metrics

### Key Concepts You Learned

- **Power BI Desktop vs. Service**: Desktop for creating, Service for sharing
- **Data connections**: OData feeds and CSV file imports
- **Import mode**: Data is copied into Power BI for fast performance
- **Star schema design**: Fact and dimension tables with one-to-many relationships
- **Data modeling**: Creating relationships with drag-and-drop
- **DAX measures**: Calculated values like Total Revenue and Total Orders
- **Interactive visualizations**: Cards, charts, tables, maps, and slicers
- **Cross-filtering**: How visuals interact and filter each other
- **Publishing workflow**: From Desktop to Service
- **Dashboards**: Single-page views with pinned tiles from reports
- **Sharing**: Collaborating with colleagues (requires Power BI Pro for all users)

### Next Steps

Now that you've completed the basics, consider exploring these advanced topics:

- **Advanced DAX**: Create more sophisticated calculations with time intelligence, ranking, and statistical functions
- **Custom visuals**: Download additional visualizations from the AppSource marketplace
- **Q&A**: Ask natural language questions about your data
- **Power BI Mobile**: Access your reports and dashboards on mobile devices
- **Dataflows**: Create reusable data preparation logic in the Power BI Service
- **Paginated reports**: Create pixel-perfect reports for printing and PDF export
- **Row-level security**: Restrict data access based on user roles

### Additional Resources

- [Microsoft Power BI Documentation](https://learn.microsoft.com/power-bi/)
- [Power BI Community Forums](https://community.powerbi.com/)
- [Power BI Guided Learning](https://learn.microsoft.com/training/powerplatform/power-bi)
- [DAX Reference Guide](https://dax.guide/)

**Thank you for completing this lab!** You now have practical experience with Power BI and are ready to create your own reports and dashboards using real-world data.
