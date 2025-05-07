# Analyzing Data in a Microsoft Fabric Data Warehouse

This guide walks you through the process of analyzing data within a Microsoft Fabric data warehouse, leveraging its full SQL capabilities and integration with Power BI for visualization.

**Note:** This lab requires a Microsoft Fabric trial or a Fabric-enabled license (Premium or Fabric).

## Prerequisites

* A Microsoft Fabric account with a trial or paid license.
* A web browser to access the Microsoft Fabric portal.

## Steps

### 1. Create a Workspace

1.  Navigate to the [Microsoft Fabric home page](https://app.fabric.microsoft.com/home?experience=fabric) and sign in with your Fabric credentials.
2.  In the left-hand menu, click on **Workspaces** (🗇 icon).
3.  Click **New workspace**.
4.  Provide a name for your workspace.
5.  Ensure that the licensing mode includes Fabric capacity (Trial, Premium, or Fabric).
6.  Click **Create**.

### 2. Create a Data Warehouse

1.  Within your new workspace, click on **Create** in the left-hand menu.
2.  Under the **Data Warehouse** section, select **Warehouse**.
3.  Give your new data warehouse a unique name.
4.  Click **Create**.
5.  Wait for a few minutes for the data warehouse to be provisioned.

### 3. Create Tables and Insert Data

1.  In your newly created data warehouse, select the **T-SQL** tile.
2.  Paste and run the following `CREATE TABLE` statement to create the `DimProduct` table:

    ```sql
    CREATE TABLE dbo.DimProduct
    (
        ProductKey INTEGER NOT NULL,
        ProductAltKey VARCHAR(25) NULL,
        ProductName VARCHAR(50) NOT NULL,
        Category VARCHAR(50) NULL,
        ListPrice DECIMAL(5,2) NULL
    );
    GO
    ```

    Click the **▷ Run** button to execute the script.
3.  Click the **Refresh** button on the toolbar. In the **Explorer** pane, expand **Schemas** > **dbo** > **Tables** to verify the `DimProduct` table.
4.  On the **Home** menu tab, click **New SQL Query**.
5.  Paste and run the following `INSERT INTO` statement to add data to the `DimProduct` table:

    ```sql
    INSERT INTO dbo.DimProductVALUES
    (1, 'RING1', 'Bicycle bell', 'Accessories', 5.99),
    (2, 'BRITE1', 'Front light', 'Accessories', 15.49),
    (3, 'BRITE2', 'Rear light', 'Accessories', 15.49);
    GO
    ```

    Click the **▷ Run** button.
6.  In the **Explorer** pane, select the `DimProduct` table to confirm the inserted rows.
7.  On the **Home** menu tab, click **New SQL Query**.
8.  Copy and paste the Transact-SQL code from [https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/create-dw.txt](https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/create-dw.txt) into the new query pane.
9.  Click the **▷ Run** button to create additional tables and load data. This might take around 30 seconds.
10. Click the **Refresh** button. In the **Explorer** pane, verify that the **dbo** schema now contains the following tables:
    * `DimCustomer`
    * `DimDate`
    * `DimProduct`
    * `FactSalesOrder`

### 4. Define a Data Model

1.  In the toolbar, click the **Model layouts** button.
2.  Rearrange the tables in the model pane so that `FactSalesOrder` is in the center.
3.  Drag the `ProductKey` field from `FactSalesOrder` and drop it onto the `ProductKey` field in `DimProduct`. Confirm the following relationship details:
    * **From table:** `FactSalesOrder`
    * **Column:** `ProductKey`
    * **To table:** `DimProduct`
    * **Column:** `ProductKey`
    * **Cardinality:** Many to one (\*:1)
    * **Cross filter direction:** Single
    * **Make this relationship active:** Selected
    * **Assume referential integrity:** Unselected
4.  Repeat the process to create many-to-one relationships for:
    * `FactSalesOrder.CustomerKey` → `DimCustomer.CustomerKey`
    * `FactSalesOrder.SalesOrderDateKey` → `DimDate.DateKey`
5.  Ensure your model looks like the example provided in the original instructions.

### 5. Query Data Warehouse Tables

#### Query fact and dimension tables

1.  Click **New SQL Query**.
2.  Run the following query to analyze sales revenue by year and month:

    ```sql
    SELECT  d.[Year] AS CalendarYear,
             d.[Month] AS MonthOfYear,
             d.MonthName AS MonthName,
            SUM(so.SalesTotal) AS SalesRevenue
    FROM FactSalesOrder AS so
    JOIN DimDate AS d ON so.SalesOrderDateKey = d.DateKey
    GROUP BY d.[Year], d.[Month], d.MonthName
    ORDER BY CalendarYear, MonthOfYear;
    ```

3.  Modify the query to include sales region:

    ```sql
    SELECT  d.[Year] AS CalendarYear,
            d.[Month] AS MonthOfYear,
            d.MonthName AS MonthName,
            c.CountryRegion AS SalesRegion,
           SUM(so.SalesTotal) AS SalesRevenue
    FROM FactSalesOrder AS so
    JOIN DimDate AS d ON so.SalesOrderDateKey = d.DateKey
    JOIN DimCustomer AS c ON so.CustomerKey = c.CustomerKey
    GROUP BY d.[Year], d.[Month], d.MonthName, c.CountryRegion
    ORDER BY CalendarYear, MonthOfYear, SalesRegion;
    ```

    Run the modified query and review the results.

#### Create a view

1.  Modify the previous query to create a view named `vSalesByRegion`:

    ```sql
    CREATE VIEW vSalesByRegion
    AS
    SELECT  d.[Year] AS CalendarYear,
            d.[Month] AS MonthOfYear,
            d.MonthName AS MonthName,
            c.CountryRegion AS SalesRegion,
           SUM(so.SalesTotal) AS SalesRevenue
    FROM FactSalesOrder AS so
    JOIN DimDate AS d ON so.SalesOrderDateKey = d.DateKey
    JOIN DimCustomer AS c ON so.CustomerKey = c.CustomerKey
    GROUP BY d.[Year], d.[Month], d.MonthName, c.CountryRegion;
    ```

    Click the **▷ Run** button.
2.  Refresh the data warehouse schema in the **Explorer** pane to see the new view.
3.  Create a **New SQL Query** and run the following `SELECT` statement to query the view:

    ```sql
    SELECT CalendarYear, MonthName, SalesRegion, SalesRevenue
    FROM vSalesByRegion
    ORDER BY CalendarYear, MonthOfYear, SalesRegion;
    ```

### 6. Create a Visual Query

1.  On the **Home** menu, expand the options under **New SQL query** and select **New visual query**.
2.  Drag the `FactSalesOrder` table onto the canvas.
3.  Drag the `DimProduct` table onto the canvas.
4.  Click the **(+)** button on the `FactSalesOrder` table and select **Merge queries**.
5.  In the **Merge queries** window:
    * Select `DimProduct` as the right table.
    * Select `ProductKey` in both tables.
    * Leave the **Join kind** as **Left outer**.
    * Click **OK**.
6.  Expand the new `DimProduct` column in the preview and select **ProductName**. Click **OK**.
7.  To filter data for a specific product, click the filter icon on the `ProductName` column and select **Cable Lock**.
8.  You can now click **Visualize results** or **Download Excel file** to analyze the filtered data.

### 7. Visualize Your Data

1.  Click the **Model layouts** button.
2.  Hide the following unnecessary columns by right-clicking on them and selecting **Hide in report view**:
    * **FactSalesOrder:** `SalesOrderDateKey`, `CustomerKey`, `ProductKey`
    * **DimCustomer:** `CustomerKey`, `CustomerAltKey`
    * **DimDate:** `DateKey`, `DateAltKey`
    * **DimProduct:** `ProductKey`, `ProductAltKey`
3.  On the **Reporting** menu, select **New report**.
4.  In the **Data** pane, expand `FactSalesOrder` and select `SalesTotal`.
5.  Ensure the column chart is active and select `Category` from the `DimProduct` table.
6.  In the **Visualizations** pane, change the chart type to a **Clustered bar chart**. Resize as needed.
7.  In the **Format your visual** tab, under **General** > **Title**, change the **Text** to "Total Sales by Category".
8.  In the **File** menu, select **Save** and save the report as "Sales Report" in your workspace.
9.  Navigate back to your workspace to see the data warehouse, its semantic model, and the new report.

### 8. Clean Up Resources

1.  In the left-hand menu, click on your workspace name.
2.  Click the **...** menu on the toolbar and select **Workspace settings**.
3.  In the **General** section, click **Remove this workspace**.
4.  Confirm the deletion.

Congratulations! You have successfully analyzed data in a Microsoft Fabric data warehouse.
