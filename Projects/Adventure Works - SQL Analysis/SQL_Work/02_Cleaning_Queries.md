# 🧼 SQL Data Cleaning and Enhancement Script

## 1. Missing Data Check

```sql
SELECT * FROM Sales_2020
WHERE Territorykey IS NULL 
   OR OrderDate IS NULL 
   OR OrderNumber IS NULL 
   OR ProductKey IS NULL 
   OR Customerkey IS NULL 
   OR OrderLineItem IS NULL 
   OR OrderQuantity IS NULL;
```

## 2. Delete Null Customers

```sql
DELETE FROM Customers
WHERE FirstName IS NULL;
```

## 3. Drop Prefix Column

```sql
ALTER TABLE Customers DROP COLUMN Prefix;
```

## 4. Check Data Types

```sql
SELECT 
    TABLE_NAME,
    COLUMN_NAME,
    DATA_TYPE
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME IN (
    'Sales_2020', 'Sales_2021', 'Sales_2022', 'Returns', 'Product',
    'Customers', 'Territory', 'Product_Subcategories', 'Product_Categories'
)
ORDER BY TABLE_NAME, COLUMN_NAME;
```

## 5. Customers Table: Convert BirthDate to DOB (Date Only)

```sql
ALTER TABLE Customers ADD DOB DATE;
UPDATE Customers SET DOB = CAST(BirthDate AS DATE);
ALTER TABLE Customers DROP COLUMN BirthDate;
```

## 6. Customers Table: Capitalize First Letter of FirstName and LastName

```sql
UPDATE Customers
SET FirstName = UPPER(LEFT(FirstName,1)) + LOWER(SUBSTRING(FirstName,2,LEN(FirstName)));

UPDATE Customers
SET LastName = UPPER(LEFT(LastName,1)) + LOWER(SUBSTRING(LastName,2,LEN(LastName)));
```

## 7. Product Table: Add GrossMargin

```sql
ALTER TABLE Product ADD GrossMargin INT;
UPDATE Product
SET GrossMargin = ProductPrice - ProductCost;
```

## 8. Product Table: Add Margin (%)

```sql
ALTER TABLE Product ADD Margin FLOAT;
UPDATE Product
SET Margin = ROUND((GrossMargin / ProductPrice) * 100, 1);
```

## 9. Sales Table: Shorten OrderNumber and Change Data Type

```sql
UPDATE Sales
SET OrderNumber = RIGHT(OrderNumber, 5);

ALTER TABLE Sales
ALTER COLUMN OrderNumber INT;
```

## 10. Customers Table: Add Age Column (Assuming Current Year = 2022)

```sql
ALTER TABLE Customers ADD Age INT;
UPDATE Customers
SET Age = 2022 - YEAR(DOB);
```

## 11. Returns Table: Add ProductCategory, ProductSubcategory, Territory

```sql
ALTER TABLE Returns
ADD Territory VARCHAR(50),
    ProductCategory VARCHAR(50),
    ProductSubcategory VARCHAR(50);

UPDATE Returns
SET Territory = sa.Region,
    ProductCategory = sa.CategoryName,
    ProductSubcategory = sa.SubCategoryName
FROM Returns r
LEFT JOIN Sales_Analysis sa ON r.ProductKey = sa.Product_ProductKey;
```

## 12. Returns Table: Add Date and YearMonth Columns

```sql
ALTER TABLE Returns ADD Date DATE;
UPDATE Returns SET Date = CAST(ReturnDate AS DATE);

ALTER TABLE Returns ADD YearMonth VARCHAR(7);
UPDATE Returns SET YearMonth = FORMAT(ReturnDate, 'yyyy-MM');
```

## 13. Returns Table: Add ProductPrice from Product Table

```sql
ALTER TABLE Returns ADD ProductPrice FLOAT;
UPDATE Returns
SET ProductPrice = p.ProductPrice
FROM Returns r
LEFT JOIN Product p ON r.ProductKey = p.ProductKey;
```
