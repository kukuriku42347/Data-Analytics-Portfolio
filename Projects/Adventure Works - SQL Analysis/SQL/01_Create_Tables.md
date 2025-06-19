```sql
-- Action: Combined 3 sales data tables into one Sales table:
select * into Sales from(
  select * from Sales_2020
  union all 
  select * from sales_2021
  union all 
  select * from sales_2022
) as sub;

-- Creating one large table for quick analysis - some columns might be missing:
select  
  -- From Sales
  s.OrderNumber, s.OrderDate, s.StockDate, s.OrderQuantity,
  s.ProductKey as Sales_ProductKey,
  -- From Product
  p.ProductKey as Product_ProductKey, p.ProductName, p.ModelName, p.ProductColor, p.ProductSize,
  p.ProductStyle, p.ProductPrice, p.ProductCost, p.GrossMargin,
  -- From Customer
  c.CustomerKey, c.FirstName, c.LastName, c.Gender, c.AnnualIncome, c.TotalChildren, c.Occupation,
  c.Homeowner, c.DOB, c.Age,
  -- From Subcategory & Category
  su.ProductSubcategoryKey, su.SubcategoryName, ca.ProductCategoryKey, ca.CategoryName,
  -- From Territory
  t.SalesTerritoryKey, t.Region, t.Country, t.Continent
into sales_analysis
from sales s
left join product p on s.ProductKey = p.ProductKey
left join customers c on s.CustomerKey = c.CustomerKey
left join product_subcategories su on p.ProductSubcategoryKey = su.ProductSubcategoryKey
left join product_categories ca on su.ProductCategoryKey = ca.ProductCategoryKey
left join territory t on s.TerritoryKey = t.SalesTerritoryKey;

-- Action: Created an aggregated sales table, because the sales tables don’t show the order size:
CREATE TABLE agg_sales (
    orderdate DATE,
    ordernumber INT,
    tot_ordercost DECIMAL(18, 2),
    tot_productcost DECIMAL(18, 2),
    tot_grossmargin DECIMAL(18, 2),
    customerkey INT,
    territorykey INT
);

WITH joined_table AS (
    SELECT s.orderdate, s.ordernumber, s.customerkey, productcost, 
           productprice, orderquantity, grossmargin, s.productkey,
           p.ProductSubcategoryKey, s.territorykey
    FROM sales s
    LEFT JOIN product p ON s.productkey = p.productkey
)
INSERT INTO agg_sales
SELECT orderdate, ordernumber, 
       SUM(productprice * orderquantity) AS tot_ordercost,
       SUM(productcost * orderquantity) AS tot_productcost, 
       SUM(grossmargin * orderquantity) AS tot_grossmargin, 
       customerkey, territorykey
FROM joined_table
GROUP BY orderdate, ordernumber, customerkey, territorykey;
