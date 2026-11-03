# Superstore Sales Analysis – Power BI Dashboard

## 1. Project overview

This project is an interactive **Superstore Sales Analysis** report created in Microsoft Power BI. It uses the `orders_data.xlsx` workbook as its source and presents sales, profit, quantity, order volume, product-category performance, regional performance, and geographic distribution in one report page.

The project follows a typical Power BI workflow:

1. Load the Excel dataset into Power BI.
2. Inspect and clean/prepare the data in **Power Query Editor**.
3. Apply the prepared data to the Power BI model.
4. Create a calculated measure in **Data/Modeling view**.
5. Build report visuals, KPI cards, and slicers.
6. Arrange and format the visuals to create the final dashboard.

**Project files**
- `orders_data.xlsx` – source dataset.
- `dashboard.pbix` – Power BI report and data model.

## 2. Dataset description

The workbook contains one worksheet named `orders`, with **9,994 rows and 21 columns**. Each row represents an order line (a product line within an order), rather than necessarily one entire order.

| Column | Meaning |
|---|---|
| `row_id` | Unique row identifier in the source data |
| `order_id` | Order reference; multiple rows can belong to one order |
| `order_date` | Date on which the order was placed |
| `ship_date` | Date on which the order was shipped |
| `ship_mode` | Shipping service/method |
| `customer_id` | Customer identifier |
| `customer_name` | Customer name |
| `segment` | Customer segment |
| `country` | Country |
| `city` | City |
| `state` | State |
| `postal_code` | Postal/ZIP code |
| `region` | Sales region |
| `product_id` | Product identifier |
| `category` | Main product category |
| `sub_category` | More detailed product category |
| `product_name` | Product description/name |
| `sales` | Sales amount for the order line |
| `quantity` | Number of units in the order line |
| `discount` | Discount applied to the order line |
| `profit` | Profit or loss for the order line |

### Dataset coverage and basic profile

- Rows: **9,994**
- Columns: **21**
- Distinct order IDs: **5,009**
- Order-date range in the supplied workbook: **2 January 2019 – 29 December 2022**
- Product categories: Furniture, Office Supplies, Technology
- Regions: West, East, Central, South
- Sales total in the supplied source: approximately **2,297,200.86**
- Profit total in the supplied source: approximately **286,397.02**
- Quantity total in the supplied source: **37,873**

These totals are source-data checks. They are not guaranteed to match every dashboard view when slicers or filters are active.

## 3. Importing the Excel file into Power BI

1. Open **Power BI Desktop**.
2. Select **Home → Get data → Excel workbook**.
3. Browse to `orders_data.xlsx`.
4. In the Navigator window, select the `orders` worksheet.
5. Choose **Transform Data**, rather than loading immediately. This opens Power Query Editor and allows the source data to be checked before it enters the model.

Power Query is the data preparation layer. It records transformation steps so that the same preparation can be applied again when the source is refreshed.

## 4. Preparing the data in Power Query Editor

Power Query Editor is used to inspect, clean, reshape, and type the source data. The preparation process should be documented as a sequence of repeatable steps.

### 4.1 Inspect the table and column names

- Confirm that the correct worksheet (`orders`) was selected.
- Check that the first row is being interpreted as the column-header row.
- Review all 21 fields and confirm that the column names are meaningful.
- Check for unexpected blank rows, error values, and columns that are not required for analysis.

### 4.2 Set appropriate data types

Correct data types are important because they control how Power BI sorts, filters, aggregates, and formats values.

| Field(s) | Appropriate type | Reason |
|---|---|---|
| `row_id`, `quantity` | Whole number | Identifiers/counts; quantity is additive |
| `order_id`, `customer_id`, `product_id` | Text | Codes should be treated as identifiers, not quantities |
| `order_date`, `ship_date` | Date | Enables date filtering and time-based analysis |
| `postal_code` | Text or whole number, depending on the analysis | It is an identifier, not a value to sum; text also preserves leading zeroes |
| `sales`, `profit`, `discount` | Decimal number | These fields contain fractional values |
| Descriptive fields such as city, state, region, category, segment, ship mode | Text | Used for grouping, filtering, and labels |

In Power Query, select a column and use the data-type icon in its header or **Transform → Data Type**. Confirm that date values are recognized as dates, not text.

### 4.3 Check missing values and errors

- Use column-quality indicators or filters to identify blank and error values.
- The supplied workbook has **11 blank values in `postal_code`** and no blanks in the other 20 columns.
- A missing postal code does not automatically mean the entire order line should be deleted. Keep the row if its sales and other analysis fields are valid; only replace or remove the missing postal code if there is a justified business rule.
- Check for conversion errors after changing data types.

### 4.4 Check duplicate records

- Check whether `row_id` is unique.
- Check for completely duplicated rows.
- Do not remove rows merely because `order_id` repeats: an order can contain multiple product lines, so repeated order IDs are expected.
- In the supplied source, `row_id` has no duplicates and there are no fully duplicated rows.

### 4.5 Review date and numeric fields

- Confirm that `order_date` and `ship_date` are valid dates.
- Check sales, profit, quantity, and discount for errors or unexpected text values.
- Negative profit values can be legitimate: they indicate that an order line made a loss. Do not remove them simply because they are negative.
- Keep discount as a numeric field so it can be summarized or used in calculations when needed.

### 4.6 Apply the query

After completing the required transformations, select **Home → Close & Apply**. Power BI loads the prepared table into the data model.

> **Important documentation note:** The supplied `.pbix` file exposes the report layout and fields used by its visuals, but it does not provide a reliable, human-readable audit of every Power Query Applied Step. Therefore, the sections above describe the correct preparation checks and workflow, not a claim that every listed operation was definitely performed in this particular query. To make the README an exact transformation log, open **Transform data → Query Settings → Applied Steps** in Power BI and replace/add the actual step names and actions (for example, Changed Type, Removed Columns, Replaced Values, or Filtered Rows) as applicable.

## 5. Data modelling and measures

After applying the query, the prepared `orders` table is available in the Power BI model. Data modelling is where fields and calculations are organized for analysis.

### 5.1 Column versus measure

A **column** stores a value for each row. For example, `sales` stores the sales amount for each order line.

A **measure** is a calculation evaluated when a visual is queried. Its result changes with the filter context—for example, selecting a region or date changes the total sales displayed by a card.

Measures are created using **DAX (Data Analysis Expressions)**:

1. Open **Data view** or **Model view**.
2. Select the `Orders` table.
3. Select **Modeling → New measure** (or right-click the table and select **New measure**).
4. Enter the DAX expression in the formula bar.
5. Press Enter and format the measure as appropriate.

### 5.2 Measure identified in the supplied report

The report contains a measure named **`No of Orders`**, which is used by a KPI card. Its exact DAX expression is not exposed in the report's visual definition, so it should be checked in the Power BI formula bar before being documented as the original formula.

A standard distinct-order measure for this dataset is:

```DAX
No of Orders = DISTINCTCOUNT(Orders[Order ID])
```

This counts each order once, even when an order has several product rows. If the actual measure in the PBIX uses a different expression, retain the actual formula and update this section accordingly.

### 5.3 Other KPI aggregations visible in the report

The report also uses the `Sales`, `Profit`, and `Quantity` fields as summed values in KPI cards and charts. The visual definition shows these as **Sum of Sales**, **Sum of Profit**, and **Sum of Quantity** aggregations. They may be implicit aggregations rather than separately created named measures.

For a more maintainable model, these can optionally be created as explicit measures:

```DAX
Total Sales = SUM(Orders[Sales])

Total Profit = SUM(Orders[Profit])

Total Quantity = SUM(Orders[Quantity])

No of Orders = DISTINCTCOUNT(Orders[Order ID])
```

Use the actual table and column names shown in your Fields pane. The PBIX visual metadata refers to the model table as `Orders` and displays friendly field names such as `Sales`, `Profit`, `Quantity`, and `Order ID`.

### 5.4 Why filter context matters

If a user selects **West** in the Region or State slicer, Power BI filters the relevant order rows and recalculates the measures. The Sales card, Profit card, Quantity card, order count, charts, and map can therefore update together.

This is the key difference between a static spreadsheet total and an interactive Power BI measure.

## 6. Dashboard layout and detailed visual explanation

The supplied PBIX contains one report page named **Dashboard**, with the heading **SUPERSTORE SALES ANALYSIS**. The page includes four KPI cards, product and regional charts, a geographic map, and slicers.

### 6.1 KPI cards

The cards provide a quick summary before the viewer studies the detailed charts.

**1. Total Sales**
- Displays the sum of the `Sales` column for the current filter selection.
- Answers: “How much revenue/sales value was generated?”
- Changes when the date or state selection changes.
- It is a sales-value metric, not the same as profit.

**2. Total Profit**
- Displays the sum of the `Profit` column.
- Answers: “How much profit or loss did the selected transactions generate?”
- Can be positive or negative depending on the selected records.
- Comparing Sales with Profit helps distinguish high sales volume from profitable sales.

**3. Total Quantity**
- Displays the sum of the `Quantity` column.
- Answers: “How many product units were sold?”
- It measures units, not the number of orders. One order can contain multiple units and multiple product lines.

**4. No of Orders**
- Displays the distinct order count using the `No of Orders` measure.
- Answers: “How many unique orders are represented in the current selection?”
- A distinct count is appropriate because the dataset has multiple rows per order.

### 6.2 Sales by Category – pie chart

- **Category field:** `Category`
- **Value:** Sum of `Sales`
- Compares the contribution of Furniture, Office Supplies, and Technology to total sales.
- A larger slice indicates a larger share of sales value in the current filter context.
- Use this chart to identify which main product category contributes most to sales.
- It describes sales contribution; it does not directly show which category has the highest profit.

### 6.3 Profit by Region – doughnut chart

- **Category/legend:** `Region`
- **Value:** Sum of `Profit`
- Compares profit across West, East, Central, and South.
- Helps identify the regions contributing to overall profit.
- Interpret this alongside the sales figures: a region with high sales is not necessarily the region with the highest profit.
- If a selected region has negative profit, the business should investigate its product mix, discounts, and costs.

### 6.4 Sales by Sub-Category – column charts

The report contains two column-chart visuals using:
- **Category axis:** `Sub-Category`
- **Value axis:** Sum of `Sales`
- **Sort:** Sales in descending order in the inspected chart definition.

These charts compare sales across the more detailed product groups, such as Bookcases, Chairs, Phones, and other sub-categories present in the data.

How to interpret them:
- Taller columns represent higher sales value.
- Shorter columns represent lower sales value.
- Descending order makes the strongest sales contributors easier to identify.
- Comparing sub-categories helps move from a broad category-level view to a more actionable product-group view.

The PBIX contains two sub-category column charts. Review the report canvas to confirm whether they are intentionally showing different filters or formatting; the visual metadata indicates that both use Sub-Category and Sum of Sales.

### 6.5 Geographic sales map

- **Location field:** `State`
- **Additional geographic grouping:** `City`
- **Tooltip/value:** Sum of `Sales`
- **Visual type:** Filled map

The map provides a geographic view of sales across states and cities. It helps answer:
- Which states contribute to sales?
- How is sales activity distributed geographically?
- Which locations may deserve additional sales or marketing attention?

Hover over a map area to inspect its available tooltip information. Map interpretation depends on Power BI correctly geocoding the State and City fields. If locations are ambiguous, set the appropriate **Data category** for geographic columns and verify the country context.

### 6.6 State slicer

- Uses the `State` field.
- Allows the viewer to filter the report to a selected state.
- The inspected slicer is configured for single selection and includes a Select All option.
- Selecting a state should update the KPI cards and other visuals that interact with the slicer.

### 6.7 Order Date slicer

- Uses `Order Date Converted`.
- Is configured as a **Between** date slicer.
- Allows the viewer to choose a start and end date.
- It supports time-period analysis, such as comparing the dashboard for a shorter period against the full dataset.
- The selected date range filters the visuals that use the same report filter context.

The model field is named `Order Date Converted` in the report definition. Check the Fields pane and Power Query steps to confirm how this field was created from the source `order_date` column.

## 7. How to interact with the dashboard

1. Open `dashboard.pbix` in Power BI Desktop.
2. Start with the four KPI cards to understand the overall result.
3. Use the Order Date slicer to select the analysis period.
4. Use the State slicer to focus on a particular state.
5. Observe how the cards and charts respond.
6. Compare category sales with regional profit.
7. Review the sub-category columns to identify the product groups driving sales.
8. Use the map to explore where sales are occurring.
9. Clear slicer selections to return to the broader view.

**Tip:** If selecting one visual does not filter another visual as expected, select the source visual and open **Format → Edit interactions**. Confirm whether the target visual should be filtered, highlighted, or left unaffected.

## 8. Business questions this dashboard can help answer

- What is the total sales value for the selected period and state?
- How much profit did the selected transactions generate?
- How many units were sold?
- How many unique orders were placed?
- Which product category contributes the most sales?
- Which region contributes the most profit?
- Which sub-categories have the highest and lowest sales?
- How does sales activity vary across states and cities?
- Does a change in date range or state materially change the business results?

The dashboard is descriptive: it summarizes historical transaction data. It does not, by itself, establish why a result occurred or predict future performance.

## 9. Refreshing the dashboard

If the source workbook is updated:

1. Keep the expected column names and data structure, or update the Power Query steps if the structure changes.
2. Open the PBIX in Power BI Desktop.
3. Select **Home → Refresh**.
4. Check for source-path, data-type, or query errors.
5. Verify the KPI totals and a few visuals after refresh.
6. Save the PBIX.

If the Excel file has moved, update the source path through **Transform data → Data source settings** or the relevant Source step in Power Query.

## 10. Validation and quality checks

Before sharing the report, check the following:

- The imported row count is consistent with the intended source.
- `row_id` is unique and repeated `order_id` values have not been incorrectly removed.
- Date fields are typed as Date.
- Sales, Profit, and Quantity are numeric.
- Missing postal codes are handled deliberately rather than deleting otherwise valid transactions.
- Negative profit values remain available for analysis.
- The `No of Orders` measure counts distinct order IDs.
- Slicer selections affect the intended visuals.
- The map resolves states and cities correctly.
- Totals reconcile with the source workbook when no filters are applied.

Source workbook baseline checks:
- Sales: approximately **2,297,200.86**
- Profit: approximately **286,397.02**
- Quantity: **37,873**
- Distinct orders: **5,009**

Small display differences can occur because of rounding and formatting. A larger difference should prompt a review of filters, data types, query steps, relationships, and DAX.

## 11. Limitations and recommended improvements

1. **Add a dedicated Date table.** A marked Date table provides a stronger foundation for year, quarter, month, and time-intelligence calculations.
2. **Create explicit KPI measures.** Named measures such as Total Sales and Total Profit improve consistency and make the model easier to maintain.
3. **Add profit margin.** For example, `DIVIDE([Total Profit], [Total Sales])` provides profit as a proportion of sales. Format it as a percentage.
4. **Add year-over-year analysis.** With a proper Date table, compare sales and profit with the previous year.
5. **Add a discount analysis.** Investigate whether high discounts are associated with lower profit.
6. **Improve visual titles and tooltips.** Dynamic titles can show the selected state and date range; tooltips can include sales, profit, quantity, and order count.
7. **Review the two sub-category charts.** Ensure that both visuals have a distinct analytical purpose or remove an unnecessary duplicate.
8. **Document exact Power Query steps and DAX.** Capture the actual Applied Steps and formula-bar expressions from the PBIX so the project is fully reproducible.

## 12. Tools and technologies

- **Microsoft Power BI Desktop** – data modelling, DAX, report design, and interactive visualization.
- **Power Query Editor** – data import, profiling, cleaning, and transformation.
- **DAX** – measures and filter-aware calculations.
- **Microsoft Excel (`.xlsx`)** – source data storage.

## 13. Project outcome

The completed report turns a transaction-level sales dataset into an interactive sales-analysis dashboard. It combines high-level KPIs with product, regional, sub-category, and geographic analysis. The date and state slicers allow users to investigate different subsets of the business without manually changing the underlying data.

The most important analytical distinction is between **sales, profit, quantity, and orders**: they measure different aspects of performance and should not be treated as interchangeable. Together, they provide a more complete view of business activity.
