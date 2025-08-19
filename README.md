# Customer Cohort Analysis in SQL with Retail Data

## What is a Cohort?

A cohort is a group of customers who share a common characteristic, typically defined by their first interaction with a business, such as their initial purchase date. In this analysis, customers are grouped by the month of their first transaction, allowing us to track their behavior over time.

## Why We Need Cohort Analysis

Cohort analysis helps businesses understand customer behavior by grouping users into cohorts based on shared attributes, such as their first purchase month. It enables tracking of retention, churn, and purchasing patterns over time, providing insights into how different customer groups engage with the business.

## Database Setup

The analysis uses a retail dataset stored in a MySQL database. Below is the SQL code to create the database and table:

```sql
CREATE DATABASE RETAIL_SALES;
CREATE TABLE Retail (
    InvoiceNo VARCHAR(20),
    StockCode VARCHAR(20),
    Descriptions TEXT,
    Quantity INT,
    InvoiceDate VARCHAR(50), 
    UnitPrice DECIMAL(10,2),
    CustomerID INT,
    Country VARCHAR(50)
);
```

## Exploring Retail Data

The dataset contains 541,909 records with details about retail transactions, including invoice numbers, product descriptions, quantities, invoice dates, unit prices, customer IDs, and countries. Below are some exploratory queries and their outputs:

### Sample Data
```sql
SELECT * FROM Retail LIMIT 10;
```

**Output**:
| InvoiceNo | StockCode | Descriptions                           | Quantity | InvoiceDate       | UnitPrice | CustomerID | Country        |
|-----------|-----------|----------------------------------------|----------|-------------------|-----------|------------|----------------|
| 536365    | 85123A    | WHITE HANGING HEART T-LIGHT HOLDER     | 6        | 12/01/2010 08:26  | 2.55      | 17850      | United Kingdom |
| 536365    | 71053     | WHITE METAL LANTERN                    | 6        | 12/01/2010 08:26  | 3.39      | 17850      | United Kingdom |
| 536365    | 84406B    | CREAM CUPID HEARTS COAT HANGER         | 8        | 12/01/2010 08:26  | 2.75      | 17850      | United Kingdom |
| 536365    | 84029G    | KNITTED UNION FLAG HOT WATER BOTTLE    | 6        | 12/01/2010 08:26  | 3.39      | 17850      | United Kingdom |
| 536365    | 84029E    | RED WOOLLY HOTTIE WHITE HEART.         | 6        | 12/01/2010 08:26  | 3.39      | 17850      | United Kingdom |
| 536365    | 22752     | SET 7 BABUSHKA NESTING BOXES           | 2        | 12/01/2010 08:26  | 7.65      | 17850      | United Kingdom |
| 536365    | 21730     | GLASS STAR FROSTED T-LIGHT HOLDER      | 6        | 12/01/2010 08:26  | 4.25      | 17850      | United Kingdom |
| 536366    | 22633     | HAND WARMER UNION JACK                 | 6        | 12/01/2010 08:28  | 1.85      | 17850      | United Kingdom |
| 536366    | 22632     | HAND WARMER RED POLKA DOT              | 6        | 12/01/2010 08:28  | 1.85      | 17850      | United Kingdom |
| 536367    | 84879     | ASSORTED COLOUR BIRD ORNAMENT          | 32       | 12/01/2010 08:34  | 1.69      | 13047      | United Kingdom |

### Total Records
```sql
SELECT COUNT(*) FROM Retail; -- 541909
```

**Output**: 541,909

### Unique Countries
```sql
SELECT DISTINCT Country FROM Retail;
```

**Output**:
- United Kingdom
- France
- Australia
- Netherlands
- Germany
- Norway
- EIRE
- Switzerland
- Spain
- Poland
- Portugal
- Italy
- Belgium
- Lithuania
- Japan
- Iceland
- Channel Islands
- Denmark
- Cyprus
- Sweden
- Austria
- Israel
- Finland
- Bahrain
- Greece
- Hong Kong
- Singapore
- Lebanon
- United Arab Emirates
- Saudi Arabia
- Czech Republic
- Canada
- Unspecified
- Brazil
- USA
- European Community
- Malta
- RSA

### Minimum Quantity
```sql
SELECT MIN(Quantity) FROM Retail; -- -80995
```

**Output**: -80,995

### Negative Quantity Records
```sql
SELECT COUNT(*) FROM Retail WHERE Quantity <= 0; -- 10624
```

**Output**: 10,624

### Canceled Orders
```sql
SELECT COUNT(*) FROM Retail WHERE InvoiceNo LIKE 'C%'; -- 9288
```

**Output**: 9,288

### Missing Customer IDs
```sql
SELECT COUNT(*) FROM Retail WHERE CustomerID IS NULL; -- 135080
```

**Output**: 135,080

### Empty Customer IDs
```sql
SELECT COUNT(*) FROM Retail WHERE CustomerID = ''; -- 0
```

**Output**: 0

## Cohort Analysis

### Customer Retention
This query tracks the number of unique customers who return to make purchases in subsequent months after their first transaction.

```sql
WITH CTE1 AS (
  SELECT
    CustomerID,
    STR_TO_DATE(InvoiceDate, '%m/%d/%Y %H:%i') AS Formatted_Date
  FROM Retail
  WHERE CustomerID IS NOT NULL AND CustomerID != ''
    AND InvoiceNo NOT LIKE 'C%' AND Quantity > 0 AND UnitPrice > 0
),
CTE2 AS (
  SELECT
    CustomerID,
    Formatted_Date AS Purchase_Date,
    MIN(Formatted_Date) OVER (PARTITION BY CustomerID) AS First_Transaction_Date
  FROM CTE1
),
CTE3 AS (
  SELECT
    CustomerID,
    DATE_FORMAT(First_Transaction_Date, '%Y-%m-01') AS First_Transaction_Month,
    TIMESTAMPDIFF(MONTH, First_Transaction_Date, Purchase_Date) AS Cohort_Month_Num
  FROM CTE2
)
SELECT
  First_Transaction_Month AS Cohort,
  COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 0 THEN CustomerID END) AS Month_0,
  COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 1 THEN CustomerID END) AS Month_1,
  COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 2 THEN CustomerID END) AS Month_2,
  COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 3 THEN CustomerID END) AS Month_3,
  COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 4 THEN CustomerID END) AS Month_4,
  COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 5 THEN CustomerID END) AS Month_5,
  COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 6 THEN CustomerID END) AS Month_6,
  COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 7 THEN CustomerID END) AS Month_7,
  COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 8 THEN CustomerID END) AS Month_8,
  COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 9 THEN CustomerID END) AS Month_9,
  COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 10 THEN CustomerID END) AS Month_10,
  COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 11 THEN CustomerID END) AS Month_11,
  COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 12 THEN CustomerID END) AS Month_12
FROM CTE3
GROUP BY First_Transaction_Month
ORDER BY First_Transaction_Month;
```

**Output**:
| Cohort      | Month_0 | Month_1 | Month_2 | Month_3 | Month_4 | Month_5 | Month_6 | Month_7 | Month_8 | Month_9 | Month_10 | Month_11 | Month_12 |
|-------------|---------|---------|---------|---------|---------|---------|---------|---------|---------|---------|----------|----------|----------|
| 01/12/2010 | 885     | 333     | 290     | 356     | 293     | 342     | 321     | 305     | 319     | 339     | 372      | 448      | 89       |
| 01/01/2011 | 417     | 97      | 117     | 117     | 117     | 113     | 104     | 110     | 134     | 144     | 107      | 3        | 0        |
| 01/02/2011 | 380     | 85      | 92      | 108     | 81      | 101     | 98      | 95      | 111     | 87      | 3        | 0        | 0        |
| 01/03/2011 | 452     | 87      | 103     | 100     | 82      | 95      | 112     | 118     | 95      | 8       | 0        | 0        | 0        |
| 01/04/2011 | 300     | 77      | 51      | 68      | 52      | 65      | 70      | 66      | 6       | 0       | 0        | 0        | 0        |
| 01/05/2011 | 284     | 54      | 44      | 56      | 70      | 67      | 68      | 3       | 0       | 0       | 0        | 0        | 0        |
| 01/06/2011 | 242     | 38      | 47      | 56      | 69      | 66      | 4       | 0       | 0       | 0       | 0        | 0        | 0        |
| 01/07/2011 | 188     | 35      | 40      | 45      | 40      | 0       | 0       | 0       | 0       | 0       | 0        | 0        | 0        |
| 01/08/2011 | 169     | 37      | 50      | 36      | 1       | 0       | 0       | 0       | 0       | 0       | 0        | 0        | 0        |
| 01/09/2011 | 299     | 88      | 72      | 4       | 0       | 0       | 0       | 0       | 0       | 0       | 0        | 0        | 0        |
| 01/10/2011 | 358     | 71      | 5       | 0       | 0       | 0       | 0       | 0       | 0       | 0       | 0        | 0        | 0        |
| 01/11/2011 | 323     | 10      | 0       | 0       | 0       | 0       | 0       | 0       | 0       | 0       | 0        | 0        | 0        |
| 01/12/2011 | 41      | 0       | 0       | 0       | 0       | 0       | 0       | 0       | 0       | 0       | 0        | 0        | 0        |

### Customer Retention Rate (%)
This query calculates the percentage of customers retained each month relative to the initial cohort size.

```sql
WITH CTE1 AS (
  SELECT
    CustomerID,
    STR_TO_DATE(InvoiceDate, '%m/%d/%Y %H:%i') AS Formatted_Date
  FROM Retail
  WHERE CustomerID IS NOT NULL AND CustomerID != ''
    AND InvoiceNo NOT LIKE 'C%' AND Quantity > 0 AND UnitPrice > 0
),
CTE2 AS (
  SELECT
    CustomerID,
    Formatted_Date AS Purchase_Date,
    MIN(Formatted_Date) OVER (PARTITION BY CustomerID) AS First_Transaction_Date
  FROM CTE1
),
CTE3 AS (
  SELECT
    CustomerID,
    DATE_FORMAT(First_Transaction_Date, '%Y-%m-01') AS First_Transaction_Month,
    TIMESTAMPDIFF(MONTH, First_Transaction_Date, Purchase_Date) AS Cohort_Month_Num
  FROM CTE2
),
Customer_Retention AS (
  SELECT
    First_Transaction_Month AS Cohort,
    COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 0 THEN CustomerID END) AS Month_0,
    COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 1 THEN CustomerID END) AS Month_1,
    COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 2 THEN CustomerID END) AS Month_2,
    COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 3 THEN CustomerID END) AS Month_3,
    COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 4 THEN CustomerID END) AS Month_4,
    COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 5 THEN CustomerID END) AS Month_5,
    COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 6 THEN CustomerID END) AS Month_6,
    COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 7 THEN CustomerID END) AS Month_7,
    COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 8 THEN CustomerID END) AS Month_8,
    COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 9 THEN CustomerID END) AS Month_9,
    COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 10 THEN CustomerID END) AS Month_10,
    COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 11 THEN CustomerID END) AS Month_11,
    COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 12 THEN CustomerID END) AS Month_12
  FROM CTE3
  GROUP BY First_Transaction_Month
  ORDER BY First_Transaction_Month
)
SELECT
  Cohort,
  Month_0,
  ROUND(100.0 * Month_1 / Month_0, 2) AS Month_1,
  ROUND(100.0 * Month_2 / Month_0, 2) AS Month_2,
  ROUND(100.0 * Month_3 / Month_0, 2) AS Month_3,
  ROUND(100.0 * Month_4 / Month_0, 2) AS Month_4,
  ROUND(100.0 * Month_5 / Month_0, 2) AS Month_5,
  ROUND(100.0 * Month_6 / Month_0, 2) AS Month_6,
  ROUND(100.0 * Month_7 / Month_0, 2) AS Month_7,
  ROUND(100.0 * Month_8 / Month_0, 2) AS Month_8,
  ROUND(100.0 * Month_9 / Month_0, 2) AS Month_9,
  ROUND(100.0 * Month_10 / Month_0, 2) AS Month_10,
  ROUND(100.0 * Month_11 / Month_0, 2) AS Month_11,
  ROUND(100.0 * Month_12 / Month_0, 2) AS Month_12
FROM Customer_Retention
ORDER BY Cohort;
```

**Output**:
| Cohort      | Month_0 | Month_1 | Month_2 | Month_3 | Month_4 | Month_5 | Month_6 | Month_7 | Month_8 | Month_9 | Month_10 | Month_11 | Month_12 |
|-------------|---------|---------|---------|---------|---------|---------|---------|---------|---------|---------|----------|----------|----------|
| 01/12/2010 | 885     | 37.63   | 32.77   | 40.23   | 33.11   | 38.64   | 36.27   | 34.46   | 36.05   | 38.31   | 42.03    | 50.62    | 10.06    |
| 01/01/2011 | 417     | 23.26   | 28.06   | 28.06   | 28.06   | 27.1    | 24.94   | 26.38   | 32.13   | 34.53   | 25.66    | 0.72     | 0        |
| 01/02/2011 | 380     | 22.37   | 24.21   | 28.42   | 21.32   | 26.58   | 25.79   | 25      | 29.21   | 22.89   | 0.79     | 0        | 0        |
| 01/03/2011 | 452     | 19.25   | 22.79   | 22.12   | 18.14   | 21.02   | 24.78   | 26.11   | 21.02   | 1.77    | 0        | 0        | 0        |
| 01/04/2011 | 300     | 25.67   | 17      | 22.67   | 17.33   | 21.67   | 23.33   | 22      | 2       | 0       | 0        | 0        | 0        |
| 01/05/2011 | 284     | 19.01   | 15.49   | 19.72   | 24.65   | 23.59   | 23.94   | 1.06    | 0       | 0       | 0        | 0        | 0        |
| 01/06/2011 | 242     | 15.7    | 19.42   | 23.14   | 28.51   | 27.27   | 1.65    | 0       | 0       | 0       | 0        | 0        | 0        |
| 01/07/2011 | 188     | 18.62   | 21.28   | 23.94   | 21.28   | 0       | 0       | 0       | 0       | 0       | 0        | 0        | 0        |
| 01/08/2011 | 169     | 21.89   | 29.59   | 21.3    | 0.59    | 0       | 0       | 0       | 0       | 0       | 0        | 0        | 0        |
| 01/09/2011 | 299     | 29.43   | 24.08   | 1.34    | 0       | 0       | 0       | 0       | 0       | 0       | 0        | 0        | 0        |
| 01/10/2011 | 358     | 19.83   | 1.4     | 0       | 0       | 0       | 0       | 0       | 0       | 0       | 0        | 0        | 0        |
| 01/11/2011 | 323     | 3.1     | 0       | 0       | 0       | 0       | 0       | 0       | 0       | 0       | 0        | 0        | 0        |
| 01/12/2011 | 41      | 0       | 0       | 0       | 0       | 0       | 0       | 0       | 0       | 0       | 0        | 0        | 0        |

### Customer Churn Rate
This query calculates the percentage of customers who do not return each month (churn rate) relative to the initial cohort size.

```sql
WITH CTE1 AS (
  SELECT
    CustomerID,
    STR_TO_DATE(InvoiceDate, '%m/%d/%Y %H:%i') AS Formatted_Date
  FROM Retail
  WHERE CustomerID IS NOT NULL AND CustomerID != ''
    AND InvoiceNo NOT LIKE 'C%' AND Quantity > 0 AND UnitPrice > 0
),
CTE2 AS (
  SELECT
    CustomerID,
    Formatted_Date AS Purchase_Date,
    MIN(Formatted_Date) OVER (PARTITION BY CustomerID) AS First_Transaction_Date
  FROM CTE1
),
CTE3 AS (
  SELECT
    CustomerID,
    DATE_FORMAT(First_Transaction_Date, '%Y-%m-01') AS First_Transaction_Month,
    TIMESTAMPDIFF(MONTH, First_Transaction_Date, Purchase_Date) AS Cohort_Month_Num
  FROM CTE2
),
Customer_Retention AS (
  SELECT
    First_Transaction_Month AS Cohort,
    COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 0 THEN CustomerID END) AS Month_0,
    COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 1 THEN CustomerID END) AS Month_1,
    COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 2 THEN CustomerID END) AS Month_2,
    COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 3 THEN CustomerID END) AS Month_3,
    COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 4 THEN CustomerID END) AS Month_4,
    COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 5 THEN CustomerID END) AS Month_5,
    COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 6 THEN CustomerID END) AS Month_6,
    COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 7 THEN CustomerID END) AS Month_7,
    COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 8 THEN CustomerID END) AS Month_8,
    COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 9 THEN CustomerID END) AS Month_9,
    COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 10 THEN CustomerID END) AS Month_10,
    COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 11 THEN CustomerID END) AS Month_11,
    COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 12 THEN CustomerID END) AS Month_12
  FROM CTE3
  GROUP BY First_Transaction_Month
  ORDER BY First_Transaction_Month
)
SELECT
  Cohort,
  Month_0,
  ROUND(100 - (100.0 * Month_1 / Month_0), 2) AS Month_1,
  ROUND(100 - (100.0 * Month_2 / Month_0), 2) AS Month_2,
  ROUND(100 - (100.0 * Month_3 / Month_0), 2) AS Month_3,
  ROUND(100 - (100.0 * Month_4 / Month_0), 2) AS Month_4,
  ROUND(100 - (100.0 * Month_5 / Month_0), 2) AS Month_5,
  ROUND(100 - (100.0 * Month_6 / Month_0), 2) AS Month_6,
  ROUND(100 - (100.0 * Month_7 / Month_0), 2) AS Month_7,
  ROUND(100 - (100.0 * Month_8 / Month_0), 2) AS Month_8,
  ROUND(100 - (100.0 * Month_9 / Month_0), 2) AS Month_9,
  ROUND(100 - (100.0 * Month_10 / Month_0), 2) AS Month_10,
  ROUND(100 - (100.0 * Month_11 / Month_0), 2) AS Month_11,
  ROUND(100 - (100.0 * Month_12 / Month_0), 2) AS Month_12
FROM Customer_Retention
ORDER BY Cohort;
```

**Output**:
| Cohort      | Month_0 | Month_1 | Month_2 | Month_3 | Month_4 | Month_5 | Month_6 | Month_7 | Month_8 | Month_9 | Month_10 | Month_11 | Month_12 |
|-------------|---------|---------|---------|---------|---------|---------|---------|---------|---------|---------|----------|----------|----------|
| 01/12/2010 | 885     | 62.37   | 67.23   | 59.77   | 66.89   | 61.36   | 63.73   | 65.54   | 63.95   | 61.69   | 57.97    | 49.38    | 89.94    |
| 01/01/2011 | 417     | 76.74   | 71.94   | 71.94   | 71.94   | 72.9    | 75.06   | 73.62   | 67.87   | 65.47   | 74.34    | 99.28    | 100      |
| 01/02/2011 | 380     | 77.63   | 75.79   | 71.58   | 78.68   | 73.42   | 74.21   | 75      | 70.79   | 77.11   | 99.21    | 100      | 100      |
| 01/03/2011 | 452     | 80.75   | 77.21   | 77.88   | 81.86   | 78.98   | 75.22   | 73.89   | 78.98   | 98.23   | 100      | 100      | 100      |
| 01/04/2011 | 300     | 74.33   | 83      | 77.33   | 82.67   | 78.33   | 76.67   | 78      | 98      | 100     | 100      | 100      | 100      |
| 01/05/2011 | 284     | 80.99   | 84.51   | 80.28   | 75.35   | 76.41   | 76.06   | 98.94   | 100     | 100     | 100      | 100      | 100      |
| 01/06/2011 | 242     | 84.3    | 80.58   | 76.86   | 71.49   | 72.73   | 98.35   | 100     | 100     | 100     | 100      | 100      | 100      |
| 01/07/2011 | 188     | 81.38   | 78.72   | 76.06   | 78.72   | 100     | 100     | 100     | 100     | 100     | 100      | 100      | 100      |
| 01/08/2011 | 169     | 78.11   | 70.41   | 78.7    | 99.41   | 100     | 100     | 100     | 100     | 100     | 100      | 100      | 100      |
| 01/09/2011 | 299     | 70.57   | 75.92   | 98.66   | 100     | 100     | 100     | 100     | 100     | 100     | 100      | 100      | 100      |
| 01/10/2011 | 358     | 80.17   | 98.6    | 100     | 100     | 100     | 100     | 100     | 100     | 100     | 100      | 100      | 100      |
| 01/11/2011 | 323     | 96.9    | 100     | 100     | 100     | 100     | 100     | 100     | 100     | 100     | 100      | 100      | 100      |
| 01/12/2011 | 41      | 100     | 100     | 100     | 100     | 100     | 100     | 100     | 100     | 100     | 100      | 100      | 100      |

### Customer Lifetime Value (CLV)
This query calculates the total revenue generated by each cohort over time.

```sql
WITH CTE1 AS (
  SELECT
    CustomerID,
    STR_TO_DATE(InvoiceDate, '%m/%d/%Y %H:%i') AS Formatted_Date,
    ROUND(Quantity * UnitPrice, 2) AS Sale_Value
  FROM Retail
  WHERE 
    CustomerID IS NOT NULL
    AND CustomerID != ''
    AND InvoiceNo NOT LIKE 'C%'
    AND Quantity > 0
    AND UnitPrice > 0
),
CTE2 AS (
  SELECT
    CustomerID,
    Formatted_Date AS Purchase_Date,
    MIN(Formatted_Date) OVER (PARTITION BY CustomerID) AS First_Transaction_Date,
    Sale_Value
  FROM CTE1
),
CTE3 AS (
  SELECT 
    CustomerID,
    First_Transaction_Date,
    Purchase_Date,
    Sale_Value,
    TIMESTAMPDIFF(MONTH, First_Transaction_Date, Purchase_Date) AS Cohort_Month_Num,
    DATE_FORMAT(First_Transaction_Date, '%Y-%m-01') AS First_Transaction_Month
  FROM CTE2
)
SELECT
  First_Transaction_Month AS Cohort,
  ROUND(SUM(CASE WHEN Cohort_Month_Num = 0 THEN Sale_Value ELSE 0 END), 0) AS Month_0,
  ROUND(SUM(CASE WHEN Cohort_Month_Num = 1 THEN Sale_Value ELSE 0 END), 0) AS Month_1,
  ROUND(SUM(CASE WHEN Cohort_Month_Num = 2 THEN Sale_Value ELSE 0 END), 0) AS Month_2,
  ROUND(SUM(CASE WHEN Cohort_Month_Num = 3 THEN Sale_Value ELSE 0 END), 0) AS Month_3,
  ROUND(SUM(CASE WHEN Cohort_Month_Num = 4 THEN Sale_Value ELSE 0 END), 0) AS Month_4,
  ROUND(SUM(CASE WHEN Cohort_Month_Num = 5 THEN Sale_Value ELSE 0 END), 0) AS Month_5,
  ROUND(SUM(CASE WHEN Cohort_Month_Num = 6 THEN Sale_Value ELSE 0 END), 0) AS Month_6,
  ROUND(SUM(CASE WHEN Cohort_Month_Num = 7 THEN Sale_Value ELSE 0 END), 0) AS Month_7,
  ROUND(SUM(CASE WHEN Cohort_Month_Num = 8 THEN Sale_Value ELSE 0 END), 0) AS Month_8,
  ROUND(SUM(CASE WHEN Cohort_Month_Num = 9 THEN Sale_Value ELSE 0 END), 0) AS Month_9,
  ROUND(SUM(CASE WHEN Cohort_Month_Num = 10 THEN Sale_Value ELSE 0 END), 0) AS Month_10,
  ROUND(SUM(CASE WHEN Cohort_Month_Num = 11 THEN Sale_Value ELSE 0 END), 0) AS Month_11,
  ROUND(SUM(CASE WHEN Cohort_Month_Num = 12 THEN Sale_Value ELSE 0 END), 0) AS Month_12
FROM CTE3
GROUP BY First_Transaction_Month
ORDER BY First_Transaction_Month;
```

**Output**:
| Cohort      | Month_0 | Month_1 | Month_2 | Month_3 | Month_4 | Month_5 | Month_6 | Month_7 | Month_8 | Month_9 | Month_10 | Month_11 | Month_12 |
|-------------|---------|---------|---------|---------|---------|---------|---------|---------|---------|---------|----------|----------|----------|
| 01/12/2010 | 623420  | 285886  | 214598  | 323449  | 204023  | 340244  | 300676  | 358902  | 331313  | 472351  | 483960   | 490040   | 83284    |
| 01/01/2011 | 311841  | 72668   | 57800   | 82443   | 76822   | 82763   | 63327   | 86372   | 106335  | 97834   | 86395    | 1282     | 0        |
| 01/02/2011 | 164131  | 43743   | 42765   | 46782   | 31476   | 42673   | 56802   | 52572   | 64642   | 46934   | 1355     | 0        | 0        |
| 01/03/2011 | 216664  | 38348   | 54527   | 43064   | 48057   | 50194   | 63282   | 81799   | 45724   | 2100    | 0        | 0        | 0        |
| 01/04/2011 | 129511  | 32843   | 22084   | 27607   | 24593   | 28533   | 34320   | 25392   | 1738    | 0       | 0        | 0        | 0        |
| 01/05/2011 | 131736  | 20801   | 19761   | 20815   | 34099   | 25389   | 201997  | 947     | 0       | 0       | 0        | 0        | 0        |
| 01/06/2011 | 143171  | 14389   | 16440   | 30483   | 32710   | 35140   | 1122    | 0       | 0       | 0       | 0        | 0        | 0        |
| 01/07/2011 | 81478   | 12751   | 17543   | 17417   | 15279   | 0       | 0       | 0       | 0       | 0       | 0        | 0        | 0        |
| 01/08/2011 | 92114   | 29639   | 54421   | 19519   | 357     | 0       | 0       | 0       | 0       | 0       | 0        | 0        | 0        |
| 01/09/2011 | 169358  | 34888   | 27769   | 1281    | 0       | 0       | 0       | 0       | 0       | 0       | 0        | 0        | 0        |
| 01/10/2011 | 195859  | 30454   | 1056    | 0       | 0       | 0       | 0       | 0       | 0       | 0       | 0        | 0        | 0        |
| 01/11/2011 | 147358  | 4519    | 0       | 0       | 0       | 0       | 0       | 0       | 0       | 0       | 0        | 0        | 0        |
| 01/12/2011 | 27059   | 0       | 0       | 0       | 0       | 0       | 0       | 0       | 0       | 0       | 0        | 0        | 0        |

### Customer Average Spend
This query calculates the average spend per customer in each cohort over time.

```sql
WITH CTE1 AS (
  SELECT
    CustomerID,
    STR_TO_DATE(InvoiceDate, '%m/%d/%Y %H:%i') AS Formatted_Date,
    ROUND(Quantity * UnitPrice, 2) AS Sale_Value
  FROM Retail
  WHERE 
    CustomerID IS NOT NULL
    AND CustomerID != ''
    AND InvoiceNo NOT LIKE 'C%'
    AND Quantity > 0
    AND UnitPrice > 0
),
CTE2 AS (
  SELECT
    CustomerID,
    Formatted_Date AS Purchase_Date,
    MIN(Formatted_Date) OVER (PARTITION BY CustomerID) AS First_Transaction_Date,
    Sale_Value
  FROM CTE1
),
CTE3 AS (
  SELECT 
    CustomerID,
    First_Transaction_Date,
    Purchase_Date,
    Sale_Value,
    TIMESTAMPDIFF(MONTH, First_Transaction_Date, Purchase_Date) AS Cohort_Month_Num,
    DATE_FORMAT(First_Transaction_Month, '%Y-%m-01') AS First_Transaction_Month
  FROM CTE2
)
SELECT
  First_Transaction_Month AS Cohort,
  ROUND(
    SUM(CASE WHEN Cohort_Month_Num = 0 THEN Sale_Value ELSE 0 END)
    / NULLIF(COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 0 THEN CustomerID END), 0)
  ) AS Month_0,
  ROUND(
    SUM(CASE WHEN Cohort_Month_Num = 1 THEN Sale_Value ELSE 0 END)
    / NULLIF(COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 1 THEN CustomerID END), 0)
  ) AS Month_1,
  ROUND(
    SUM(CASE WHEN Cohort_Month_Num = 2 THEN Sale_Value ELSE 0 END)
    / NULLIF(COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 2 THEN CustomerID END), 0)
  ) AS Month_2,
  ROUND(
    SUM(CASE WHEN Cohort_Month_Num = 3 THEN Sale_Value ELSE 0 END)
    / NULLIF(COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 3 THEN CustomerID END), 0)
  ) AS Month_3,
  ROUND(
    SUM(CASE WHEN Cohort_Month_Num = 4 THEN Sale_Value ELSE 0 END)
    / NULLIF(COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 4 THEN CustomerID END), 0)
  ) AS Month_4,
  ROUND(
    SUM(CASE WHEN Cohort_Month_Num = 5 THEN Sale_Value ELSE 0 END)
    / NULLIF(COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 5 THEN CustomerID END), 0)
  ) AS Month_5,
  ROUND(
    SUM(CASE WHEN Cohort_Month_Num = 6 THEN Sale_Value ELSE 0 END)
    / NULLIF(COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 6 THEN CustomerID END), 0)
  ) AS Month_6,
  ROUND(
    SUM(CASE WHEN Cohort_Month_Num = 7 THEN Sale_Value ELSE 0 END)
    / NULLIF(COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 7 THEN CustomerID END), 0)
  ) AS Month_7,
  ROUND(
    SUM(CASE WHEN Cohort_Month_Num = 8 THEN Sale_Value ELSE 0 END)
    / NULLIF(COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 8 THEN CustomerID END), 0)
  ) AS Month_8,
  ROUND(
    SUM(CASE WHEN Cohort_Month_Num = 9 THEN Sale_Value ELSE 0 END)
    / NULLIF(COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 9 THEN CustomerID END), 0)
  ) AS Month_9,
  ROUND(
    SUM(CASE WHEN Cohort_Month_Num = 10 THEN Sale_Value ELSE 0 END)
    / NULLIF(COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 10 THEN CustomerID END), 0)
  ) AS Month_10,
  ROUND(
    SUM(CASE WHEN Cohort_Month_Num = 11 THEN Sale_Value ELSE 0 END)
    / NULLIF(COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 11 THEN CustomerID END), 0)
  ) AS Month_11,
  ROUND(
    SUM(CASE WHEN Cohort_Month_Num = 12 THEN Sale_Value ELSE 0 END)
    / NULLIF(COUNT(DISTINCT CASE WHEN Cohort_Month_Num = 12 THEN CustomerID END), 0)
  ) AS Month_12
FROM CTE3
GROUP BY First_Transaction_Month
ORDER BY First_Transaction_Month;
```

**Output**:
| Cohort      | Month_0 | Month_1 | Month_2 | Month_3 | Month_4 | Month_5 | Month_6 | Month_7 | Month_8 | Month_9 | Month_10 | Month_11 | Month_12 |
|-------------|---------|---------|---------|---------|---------|---------|---------|---------|---------|---------|----------|----------|----------|
| 01/12/2010 | 704     | 859     | 740     | 909     | 696     | 995     | 937     | 1177    | 1039    | 1393    | 1301     | 1094     | 936      |
| 01/01/2011 | 748     | 749     | 494     | 705     | 657     | 732     | 609     | 785     | 794     | 679     | 807      | 427      | NULL     |
| 01/02/2011 | 432     | 515     | 465     | 433     | 389     | 423     | 580     | 553     | 582     | 539     | 452      | NULL     | NULL     |
| 01/03/2011 | 479     | 441     | 529     | 431     | 586     | 528     | 565     | 693     | 481     | 262     | NULL     | NULL     | NULL     |
| 01/04/2011 | 432     | 427     | 433     | 406     | 473     | 439     | 490     | 385     | 290     | NULL    | NULL     | NULL     | NULL     |
| 01/05/2011 | 464     | 385     | 449     | 372     | 487     | 379     | 2971    | 316     | NULL    | NULL    | NULL     | NULL     | NULL     |
| 01/06/2011 | 592     | 379     | 350     | 544     | 474     | 532     | 280     | NULL    | NULL    | NULL    | NULL     | NULL     | NULL     |
| 01/07/2011 | 433     | 364     | 439     | 387     | 382     | NULL    | NULL    | NULL    | NULL    | NULL    | NULL     | NULL     | NULL     |
| 01/08/2011 | 545     | 801     | 1088    | 542     | 357     | NULL    | NULL    | NULL    | NULL    | NULL    | NULL     | NULL     | NULL     |
| 01/09/2011 | 566     | 396     | 386     | 320     | NULL    | NULL    | NULL    | NULL    | NULL    | NULL    | NULL     | NULL     | NULL     |
| 01/10/2011 | 547     | 429     | 211     | NULL    | NULL    | NULL    | NULL    | NULL    | NULL    | NULL    | NULL     | NULL     | NULL     |
| 01/11/2011 | 456     | 452     | NULL    | NULL    | NULL    | NULL    | NULL    | NULL    | NULL    | NULL    | NULL     | NULL     | NULL     |
| 01/12/2011 | 660     | NULL    | NULL    | NULL    | NULL    | NULL    | NULL    | NULL    | NULL    | NULL    | NULL     | NULL     | NULL     |
