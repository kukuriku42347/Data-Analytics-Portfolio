# 💸 Profit Margin by Country

**File Origin:**  
`Projects/First Project/SQL/03_Analysis/02_profit_margin_by_country.md`

**Image Folder:**  
`Projects/First Project/SQL/..Images/`

---

## 🧠 Business Question

Which countries generate the most profit **relative to their sales**?

---

## 📊 SQL Query Used

```sql
SELECT 
  country,
  ROUND(SUM(totalprofit) / SUM(productprice * orderquantity), 2) AS margin_ratio
FROM Sales s
LEFT JOIN Product p ON s.productkey = p.productkey
LEFT JOIN territory t ON s.territorykey = t.salesterritorykey
GROUP BY country
ORDER BY margin_ratio DESC;
