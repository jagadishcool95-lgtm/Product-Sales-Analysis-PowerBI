# Retail Sales & Customer Performance Analysis — Power BI

## 1. Import the CSV

Power BI Desktop → Home → Get Data → Text/CSV → `retail_sales_analysis.csv`.

Select **Transform Data** before loading.

## 2. Power Query cleaning

In Power Query:

1. Rename the query to `Sales`.
2. Set `Order_Date` to **Date**.
3. Set `Order_ID`, `Customer_ID`, `Customer_Name`, `Gender`, `City`, `State`,
   `Category`, `Sub_Category`, `Product`, `Payment_Mode`, `Sales_Channel`,
   `Customer_Segment` to **Text**.
4. Set `Age`, `Quantity`, `Discount_Percent` to **Whole Number**.
5. Set `Unit_Price`, `Discount_Amount`, `Sales`, `Cost`, `Profit` to
   **Fixed Decimal Number / Currency** as appropriate.
6. Remove duplicates using `Order_ID` if any are found.
7. Check nulls in key columns.
8. Close & Apply.

Recommended query name: `Sales`.

## 3. Create a proper Date table

Modeling → New table:

```DAX
Date =
ADDCOLUMNS(
    CALENDAR(DATE(2025,1,1), DATE(2025,12,31)),
    "Year", YEAR([Date]),
    "Month Number", MONTH([Date]),
    "Month", FORMAT([Date], "MMM"),
    "Month Year", FORMAT([Date], "MMM yyyy"),
    "Quarter", "Q" & FORMAT([Date], "Q"),
    "Year Month Sort", YEAR([Date]) * 100 + MONTH([Date])
)
```

Then select `Date[Month]` → Sort by Column → `Month Number`.

Select `Date[Month Year]` → Sort by Column → `Year Month Sort`.

Model → Mark as date table → `Date[Date]`.

Create relationship:

`Date[Date]` (1) → `Sales[Order_Date]` (*)

Cross filter direction: Single.

## 4. Core DAX measures

```DAX
Total Sales =
SUM(Sales[Sales])
```

```DAX
Total Cost =
SUM(Sales[Cost])
```

```DAX
Total Profit =
SUM(Sales[Profit])
```

```DAX
Total Orders =
DISTINCTCOUNT(Sales[Order_ID])
```

```DAX
Total Customers =
DISTINCTCOUNT(Sales[Customer_ID])
```

```DAX
Total Quantity =
SUM(Sales[Quantity])
```

```DAX
Average Order Value =
DIVIDE([Total Sales], [Total Orders], 0)
```

```DAX
Profit Margin =
DIVIDE([Total Profit], [Total Sales], 0)
```

```DAX
Average Selling Price =
DIVIDE([Total Sales], [Total Quantity], 0)
```

```DAX
Average Discount =
AVERAGE(Sales[Discount_Percent])
```

## 5. Time-intelligence measures

```DAX
Sales Previous Month =
CALCULATE(
    [Total Sales],
    DATEADD('Date'[Date], -1, MONTH)
)
```

```DAX
Sales MoM % =
DIVIDE(
    [Total Sales] - [Sales Previous Month],
    [Sales Previous Month],
    0
)
```

```DAX
Profit Previous Month =
CALCULATE(
    [Total Profit],
    DATEADD('Date'[Date], -1, MONTH)
)
```

```DAX
Profit MoM % =
DIVIDE(
    [Total Profit] - [Profit Previous Month],
    [Profit Previous Month],
    0
)
```

```DAX
YTD Sales =
TOTALYTD(
    [Total Sales],
    'Date'[Date]
)
```

```DAX
YTD Profit =
TOTALYTD(
    [Total Profit],
    'Date'[Date]
)
```

## 6. Customer measures

```DAX
New Customers =
CALCULATE(
    DISTINCTCOUNT(Sales[Customer_ID]),
    Sales[Customer_Segment] = "New"
)
```

```DAX
Premium Customers =
CALCULATE(
    DISTINCTCOUNT(Sales[Customer_ID]),
    Sales[Customer_Segment] = "Premium"
)
```

```DAX
Regular Customers =
CALCULATE(
    DISTINCTCOUNT(Sales[Customer_ID]),
    Sales[Customer_Segment] = "Regular"
)
```

## 7. Ranking measures

```DAX
Product Sales Rank =
RANKX(
    ALL(Sales[Product]),
    [Total Sales],
    ,
    DESC,
    DENSE
)
```

```DAX
Customer Sales Rank =
RANKX(
    ALL(Sales[Customer_ID]),
    [Total Sales],
    ,
    DESC,
    DENSE
)
```

## 8. Dashboard Page 1 — Executive Overview

Page name: `01 Executive Overview`

Add these KPI cards:

- Total Sales
- Total Profit
- Profit Margin
- Total Orders
- Total Customers
- Average Order Value

Visuals:

1. Line chart
   - X-axis: `Date[Month Year]`
   - Y-axis: `[Total Sales]`

2. Clustered column chart
   - X-axis: `Sales[Category]`
   - Y-axis: `[Total Sales]`

3. Bar chart
   - Y-axis: `Sales[State]`
   - X-axis: `[Total Sales]`

4. Donut chart
   - Legend: `Sales[Sales_Channel]`
   - Values: `[Total Sales]`

5. Bar chart — Top 10 Products
   - Y-axis: `Sales[Product]`
   - X-axis: `[Total Sales]`
   - Visual filter: Top N = 10 by `[Total Sales]`

Slicers:

- Order Date
- State
- Category
- Sales Channel
- Customer Segment

## 9. Dashboard Page 2 — Product Analysis

Page name: `02 Product Analysis`

KPI cards:

- Total Sales
- Total Profit
- Total Quantity
- Average Discount

Visuals:

1. Bar chart: Category vs Sales
2. Bar chart: Sub-category vs Profit
3. Top 10 Products by Sales
4. Top 10 Products by Profit
5. Scatter chart:
   - X = Discount_Percent
   - Y = Profit
   - Size = Sales
   - Legend = Category
6. Matrix:
   - Rows = Category → Sub_Category → Product
   - Values = Sales, Cost, Profit, Quantity, Profit Margin

This page should answer:
- Which categories generate the most revenue?
- Which products generate the most profit?
- Does discounting reduce profitability?
- Which products have high sales but relatively low profit?

## 10. Dashboard Page 3 — Customer Analysis

Page name: `03 Customer Analysis`

KPI cards:

- Total Customers
- New Customers
- Premium Customers
- Average Order Value

Visuals:

1. Donut: Customer Segment vs Customers
2. Column chart: Age Group vs Sales
3. Donut: Gender vs Sales
4. Bar chart: Top 10 Customers by Sales
5. Bar chart: Top 10 Customers by Profit
6. Column chart: Customer Segment vs Profit

Create an Age Group calculated column:

```DAX
Age Group =
SWITCH(
    TRUE(),
    Sales[Age] < 25, "18-24",
    Sales[Age] < 35, "25-34",
    Sales[Age] < 45, "35-44",
    Sales[Age] < 55, "45-54",
    "55+"
)
```

## 11. Dashboard Page 4 — Regional & Channel Analysis

Page name: `04 Regional & Channel Analysis`

Visuals:

1. Map or Azure Maps:
   - Location = City
   - Size = Total Sales

2. Bar chart:
   - State vs Sales

3. Bar chart:
   - State vs Profit

4. Donut:
   - Payment Mode vs Sales

5. Column chart:
   - Online vs Offline Sales

6. Line chart:
   - Month Year vs Sales
   - Legend = Sales Channel

7. Matrix:
   - State
   - Sales
   - Profit
   - Orders
   - Profit Margin

## 12. Formatting

Use a professional corporate dashboard style.

Recommended:
- Page background: very light neutral
- KPI cards: white
- One consistent accent color
- Dark text
- Minimal borders
- Rounded cards where available
- Consistent font sizes

Suggested hierarchy:
- Page title: 20–24 pt
- KPI number: 20–28 pt
- Chart title: 12–14 pt
- Axis/labels: 9–11 pt

Avoid overcrowding each page.

## 13. Interactions

Use Format → Edit interactions.

Make slicers affect all relevant charts.

Add a Reset Filters button:

Insert → Buttons → Blank.

Add a bookmark named `Reset Filters`.

## 14. Tooltips

Create a tooltip page called:

`Tooltip - Product`

Include:
- Product
- Sales
- Profit
- Quantity
- Profit Margin
- Average Discount

Set Page Information → Tooltip = On.

## 15. Drill-through

Create a drill-through page:

`Product Details`

Drill-through field:
`Sales[Product]`

Add:
- Sales
- Profit
- Quantity
- Discount
- Monthly Sales
- Customer Segment
- Sales Channel

Users can right-click a product and open its detailed page.

## 16. Business insights to discuss

Use the dashboard to identify:

- Highest-revenue categories
- Highest-profit products
- States contributing the most sales
- Online vs offline contribution
- Most-used payment methods
- Customer segment contribution
- Monthly sales peaks and dips
- High-discount products with weak margins

Do not write predetermined findings until the report visuals confirm them.

## 17. Portfolio project description

**Retail Sales & Customer Performance Analysis — Power BI**

Built an interactive Power BI dashboard using 10,000 Indian retail transaction records. The project covers sales performance, profitability, product performance, customer segmentation, regional performance and sales-channel analysis.

Tools:
- Power BI Desktop
- Power Query
- DAX
- Data Modelling
- Interactive Visualizations

Key skills demonstrated:
- Data cleaning and transformation
- Star-schema-style date modelling
- DAX measures
- Time intelligence
- KPI design
- Top-N analysis
- Customer and product analysis
- Interactive dashboard design
