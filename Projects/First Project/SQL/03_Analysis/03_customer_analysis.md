Aims:  
Customer analysis - try to spot patterns of spending by analysing age, gender, occupation, number of children, home ownership etc.

First, aggregate customers into a table as a CTE  
```sql
with agg_cust as (
select round(sum(productprice),0) as total_spend,CustomerKey,SUM(orderquantity) as total_products_bought,
gender,annualincome,totalchildren,occupation,age,SalesTerritoryKey,Region,Country,Continent
from sales_analysis
group by CustomerKey,gender,annualincome,totalchildren,occupation,age,
SalesTerritoryKey,Region,Country,Continent)
```

Total spend/revenue by gender  
```sql
select sum(total_spend) as sum_total_spend,gender
from agg_cust
group by gender
```
Projects/First Project/SQL/Images/Customers_Male_Female_Spend.png  
INSIGHT: Male/Female fairly equal split with women slightly higher

Number of total_customers by gender  
Projects/First Project/SQL/Images/Customers_Male_Female_Spend.png  
INSIGHT: slightly fewer women, meaning average purchase of women is higher than men’s

Revenue and annual salary buckets  
```sql
select sum(total_spend) as total_spends,annualincome
from agg_cust
group by annualincome
order by total_spends desc
```
Projects/First Project/SQL/Images/Customers_Annual_Salary_Buckets.png  
INSIGHT: surprisingly most of the spend comes from customers with lower salaries. Perhaps it’s because there are more of the lower-salaries than higher-salaries in the population though.

Revenue by number of children and spend  
```sql
select sum(total_spend) as total_spends,totalchildren
from agg_cust
group by totalchildren
order by total_spends desc
```
Projects/First Project/SQL/Images/Customers_Spend_by_number_of_children.png  
INSIGHT: a very interesting insight: those that shop/buy more have fewer children, a direct correlation in fact (-0.975 correlation coefficient, using Excel)

Revenue by occupation  
```sql
select sum(total_spend) as total_spends,occupation
from agg_cust
group by occupation
order by total_spends desc
```
Projects/First Project/SQL/Images/Total_Spend_by_Occupation.png  
INSIGHT: worth targeting professionals, skilled manual and management, as they spend more than other occupation groups.

Revenue by occupation & gender - combining gender and occupation to see if we can discover something else.  
Projects/First Project/SQL/Images/Total_Spend_by_Occupation&Gender.png  
INSIGHT: same insight as before, both M and F follow the same pattern with spend according to occupation

Revenue by age group  
```sql
WITH agg_cust AS (
    SELECT 
        ROUND(SUM(productprice), 0) AS total_spend, CustomerKey, SUM(orderquantity) AS total_products_bought,
        gender, annualincome, totalchildren, occupation, age, SalesTerritoryKey, Region, Country, Continent
    FROM sales_analysis
    GROUP BY CustomerKey, gender, annualincome, totalchildren, occupation, age, SalesTerritoryKey, Region, Country, Continent
)

SELECT 
    CAST((age / 5) * 5 AS VARCHAR) + '-' + CAST((age / 5) * 5 + 4 AS VARCHAR) AS age_bucket,
    SUM(total_spend) AS total_spends
FROM agg_cust
GROUP BY (age / 5) * 5
ORDER BY total_spends DESC;
```
Projects/First Project/SQL/Images/Spend_by_Age_Buckets.png  
INSIGHT: There are customers until 110-114 age! This means the data is probably not real. Also strangely purchases begin from 42-year-olds!  
Biggest spenders are 45-69 ages, in particular 55-69. Worth-targeting with marketing.  
Recommendation: to consider if other age groups are worth targeting.
