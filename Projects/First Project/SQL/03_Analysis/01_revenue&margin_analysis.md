# 📊 Sales & Margin Analysis Report

## 🎯 Aim
Start with analysis of **revenue** — by **territory**, **region**, **category**, and **products**. Then analyze **margin** per **category** and **subcategory**.

---

## 🌍 Sales Revenue by Territory

```sql
SELECT 
  ROUND(SUM(productprice * orderquantity), 0) AS totalsale,
  country
FROM Sales s
LEFT JOIN Product p ON s.productkey = p.productkey
LEFT JOIN territory t ON s.territorykey = t.salesterritorykey
GROUP BY country
ORDER BY ROUND(SUM(productprice * orderquantity), 0) DESC;
```

📸 Image: `Projects/First Project/SQL/Images/Sales_Revenue_by_Territory1.png`  
💡 **Insight:** US brings in the most revenue, followed closely by Australia. Combined revenue from France, Germany, and the UK is comparable to top performers. Canada performs weakest.  
✅ **Recommendation:** Explore sales/marketing saturation in underperforming countries to find expansion opportunities.

---

## 🇺🇸 Sales Revenue by Region (All & US-specific)

```sql
-- All regions
SELECT 
  ROUND(SUM(productprice * orderquantity), 0) AS totalsale,
  region
FROM Sales s
LEFT JOIN Product p ON s.productkey = p.productkey
LEFT JOIN territory t ON s.territorykey = t.salesterritorykey
GROUP BY region
ORDER BY ROUND(SUM(productprice * orderquantity), 0) DESC;
```

```sql
-- United States only
SELECT 
  ROUND(SUM(productprice * orderquantity), 0) AS totalsale,
  region
FROM Sales s
LEFT JOIN Product p ON s.productkey = p.productkey
LEFT JOIN territory t ON s.territorykey = t.salesterritorykey
WHERE country = 'United States'
GROUP BY region
ORDER BY ROUND(SUM(productprice * orderquantity), 0) DESC;
```

📸 Images:  
- `Projects/First Project/SQL/Images/Sales_Revenue_by_All_Regions.png`  
- `Projects/First Project/SQL/Images/Sales_Revenue_by_US_Region.png`  

💡 **Insight:** Southwest and Northwest regions dominate revenue, even outperforming most countries. The other 3 US regions have minimal sales.  
✅ **Recommendation:** Investigate store presence and marketing spend in low-performing US regions.

---

## 🛍 Revenue by Product Category

```sql
SELECT 
  ROUND(SUM(orderquantity * productprice), 0) AS total_sales,
  productcategorykey,
  CategoryName
FROM sales_analysis
GROUP BY productcategorykey, CategoryName
ORDER BY SUM(orderquantity * productprice) DESC;
```

📸 Image: `Projects/First Project/SQL/Images/Revenue_by_Category.png`  
💡 **Insight:** Bikes dominate, making up 95% of total sales.

```sql
SELECT ROUND(
  (SELECT SUM(orderquantity * productprice)
   FROM sales_analysis
   WHERE CategoryName = 'Bikes') 
   / 
  (SELECT SUM(orderquantity * productprice)
   FROM sales_analysis) * 100, 
2);
```

✅ **Recommendation:** Focus sales strategy around the Bikes category. Explore potential in smaller categories only if margins are strong.

---

## 🚲 Top 10 Products by Revenue

```sql
SELECT TOP 10 
  ProductName,
  subcategoryname,
  categoryname,
  ROUND(SUM(orderquantity * ProductPrice), 0) AS product_sales_revenue,
  SUM(orderquantity) AS total_order_no,
  ROUND(ProductPrice, 0) AS ProductPrice
FROM sales_analysis
GROUP BY ProductName, subcategoryname, categoryname, ProductPrice
ORDER BY SUM(orderquantity * ProductPrice) DESC;
```

📸 Image: `Projects/First Project/SQL/Images/Top_10_Products.png`  
💡 **Insight:** All top 10 products are either mountain bikes or road bikes — bikes dominate both quantity and revenue.

---

## 🧢 Top 10 Non-Bike Products

```sql
SELECT TOP 10 
  ProductName,
  subcategoryname,
  categoryname,
  ROUND(SUM(orderquantity * ProductPrice), 0) AS product_sales_revenue,
  SUM(orderquantity) AS total_order_no,
  ROUND(ProductPrice, 0) AS ProductPrice
FROM sales_analysis
WHERE categoryname <> 'Bikes'
GROUP BY ProductName, subcategoryname, categoryname, ProductPrice
ORDER BY SUM(orderquantity * ProductPrice) DESC;
```

📸 Image: `Projects/First Project/SQL/Images/Top_10_non-bike_Products.png`  
💡 **Insight:** Revenue from non-bike products is much lower. Top performers include fenders, helmets, and tires.

---

## 💰 Margin Analysis by Category

```sql
SELECT 
  categoryname,
  ROUND(SUM(orderquantity * grossmargin), 0) AS total_profit,
  SUM(orderquantity) AS total_order_no,
  ROUND(SUM(orderquantity * grossmargin) / SUM(orderquantity * productprice), 2) * 100 AS 'margin%'
FROM sales_analysis
GROUP BY categoryname
ORDER BY SUM(orderquantity * grossmargin) DESC;
```

📸 Image: `Projects/First Project/SQL/Images/Margin_by_Category.png`  
💡 **Insight:** Bikes generate the most profit, but Accessories have a significantly better margin. Clothing offers the lowest return.  
✅ **Recommendation:** Investigate how to raise bike margins; consider reducing focus on clothing.

---

## 🧩 Margin by Subcategory (Revenue, Profit & Margins)

```sql
SELECT 
  SubcategoryName,
  categoryname,
  ROUND(SUM(orderquantity * ProductPrice), 0) AS product_sale_rev,
  ROUND(SUM(orderquantity * grossmargin), 0) AS total_profit,
  SUM(orderquantity) AS total_order_no,
  ROUND(SUM(orderquantity * grossmargin) / SUM(orderquantity * productprice), 2) * 100 AS 'margin%'
FROM sales_analysis
GROUP BY subcategoryname, categoryname
ORDER BY SUM(orderquantity * grossmargin) DESC;
```

📸 Image: `Projects/First Project/SQL/Images/Returns_by_Subcategory.png`  
💡 **Insight:** Mountain bikes are second in profit and have a much better margin than Road/Touring bikes. Accessories subcategories also perform well in terms of margin.  
✅ **Recommendation:** Focus more on growing Mountain Bike sales. Evaluate how to improve Road/Touring bike margins.

---

## 📆 Monthly Revenue by Territory

```sql
SELECT YearMonth,
       [Southwest] AS Southwest_Revenue,
       [Northwest] AS Northwest_Revenue,
       [Southeast] AS Southeast_Revenue,
       [Northeast] AS Northeast_Revenue,
       [Australia] AS Australia_Revenue,
       [Canada] AS Canada_Revenue,
       [United Kingdom] AS UnitedKingdom_Revenue,
       [France] AS France_Revenue,
       [Germany] AS Germany_Revenue
FROM (
    SELECT 
        FORMAT(orderdate, 'yyyy-MM') AS YearMonth,
        region,
        ROUND(productprice * orderquantity, 0) AS revenue
    FROM sales_analysis
) AS SourceTable
PIVOT (
    SUM(revenue)
    FOR region IN (
        [Southwest], [Northwest], [Southeast], [Northeast],
        [Australia], [Canada], [United Kingdom], [France], [Germany]
    )
) AS PivotTable
ORDER BY YearMonth;
```

📸 Image: `Projects/First Project/SQL/Images/Revenue_by_month_by_category.png`  
💡 **Insight:** Revenue trends upward over time for all regions, despite temporary dips for Southwest US and Australia, both of which rebounded strongly.  
✅ **Recommendation:** Compare trends to marketing spend and store expansion data for better understanding of growth drivers.

---
