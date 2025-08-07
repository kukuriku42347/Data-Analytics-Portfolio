# 📊 SQL Data Consolidation and Aggregation Script

## 1. Combine Sales Data from 2020–2022 into One Table

```sql
-- Action: Combined 3 sales data tables into one Sales table
SELECT * INTO Sales FROM (
    SELECT * FROM Sales_2020
    UNION ALL 
    SELECT * FROM Sales_2021
    UNION ALL 
    SELECT * FROM Sales_2022
) AS sub;
```

## 2. Create `sales_analysis` Table with Joined Data

```sql
-- Creating one large table for quick analysis - some columns might be missing
SELECT  
    -- From Sales
    s.OrderNumber, s.OrderDate, s.StockDate, s.OrderQuantity,
    s.ProductKey AS Sales_ProductKey,
    
    -- From Product
    p.ProductKey AS Product_ProductKey, p.ProductName, p.ModelName, 
    p.ProductColor, p.ProductSize, p.ProductStyle, p.ProductPrice, 
    p.ProductCost, p.GrossMargin,

    -- From Customer
    c.CustomerKey, c.FirstName, c.LastName, c.Gender, 
    c.AnnualIncome, c.TotalChildren, c.Occupation, 
    c.Homeowner, c.DOB, c.Age,

    -- From Subcategory & Category
    su.ProductSubcategoryKey, su.SubcategoryName, 
    ca.ProductCategoryKey, ca.CategoryName,

    -- From Territory
    t.SalesTerritoryKey, t.Region, t.Country, t.Continent

INTO Sales_Analysis
FROM Sales s
LEFT JOIN Product p ON s.ProductKey = p.ProductKey
LEFT JOIN Customers c ON s.CustomerKey = c.CustomerKey
LEFT JOIN Product_Subcategories su ON p.ProductSubcategoryKey = su.ProductSubcategoryKey
LEFT JOIN Product_Categories ca ON su.ProductCategoryKey = ca.ProductCategoryKey
LEFT JOIN Territory t ON s.TerritoryKey = t.SalesTerritoryKey;
```

## 3. Create Aggregated Sales Table

```sql
-- Action: Created an aggregated sales table because the sales tables don’t show the order size
CREATE TABLE agg_sales (
    orderdate DATE,
    ordernumber INT,
    tot_ordercost DECIMAL(18, 2),
    tot_productcost DECIMAL(18, 2),
    tot_grossmargin DECIMAL(18, 2),
    customerkey INT,
    territorykey INT
);
```

## 4. Populate `agg_sales` with Aggregated Metrics

```sql
WITH joined_table AS (
    SELECT 
        s.orderdate, s.ordernumber, s.customerkey, 
        productcost, productprice, orderquantity, grossmargin, 
        s.productkey, p.ProductSubcategoryKey, s.territorykey
    FROM Sales s
    LEFT JOIN Product p ON s.ProductKey = p.ProductKey
)

INSERT INTO agg_sales
SELECT 
    orderdate, ordernumber, 
    SUM(productprice * orderquantity) AS tot_ordercost,
    SUM(productcost * orderquantity) AS tot_productcost, 
    SUM(grossmargin * orderquantity) AS tot_grossmargin, 
    customerkey, territorykey
FROM joined_table
GROUP BY orderdate, ordernumber, customerkey, territorykey;
```
