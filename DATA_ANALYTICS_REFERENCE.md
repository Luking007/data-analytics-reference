# 📊 Data Analytics Reference Guide
### By Oyewo Lukman Segun | TS Academy 2026 | Aviation & Operations Analytics

> Personal reference toolkit covering Excel, SQL, and Power BI DAX.  
> Built alongside the TS Academy Data Analytics Programme (Feb – May 2026).

---

## 📌 Table of Contents

1. [The Analytical Framework](#1-the-analytical-framework)
2. [Data Cleaning Checklist](#2-data-cleaning-checklist)
3. [Excel Formulas](#3-excel-formulas)
4. [SQL — Joins & Merging Tables](#4-sql--joins--merging-tables)
5. [SQL — Core Commands](#5-sql--core-commands)
6. [SQL — Aggregation & Grouping](#6-sql--aggregation--grouping)
7. [SQL — Filtering](#7-sql--filtering)
8. [SQL — Advanced Queries](#8-sql--advanced-queries)
9. [Power BI — DAX Formulas](#9-power-bi--dax-formulas)
10. [Excel vs SQL vs DAX — Side by Side](#10-excel-vs-sql-vs-dax--side-by-side)
11. [KPI Formulas](#11-kpi-formulas)
12. [Portfolio Projects](#12-portfolio-projects)

---

## 1. The Analytical Framework

### 5 Questions to Ask Before Touching Any Data

| # | Question | How to Check |
|---|----------|--------------|
| 1 | **What is the grain?** What does one row represent? | Read column headers, check for duplicate IDs |
| 2 | **What period does it cover?** | `SELECT MIN(date), MAX(date) FROM table` |
| 3 | **Are there nulls and do they make sense?** | `SELECT COUNT(*) WHERE column IS NULL` |
| 4 | **What is the baseline?** Compare against last period / target / industry | Find benchmarks before reporting numbers |
| 5 | **What decision will this answer?** | Write your recommendation BEFORE building charts |

### Finding → Recommendation Framework

```
❌ WEAK:  "Apple is the most profitable brand"
✅ STRONG: "Apple generates 21% of total profit on 17% of sales volume —
            increasing Apple stock allocation by 15% in Q3 could add
            ₦42,000 to monthly profit without increasing customer base"
```

### The Full Data Journey

```
Raw Tables (Sales + Products + Customers)
         ↓
Data Cleaning (nulls, casing, formats, duplicates)
         ↓
Merge / JOIN tables using shared ID columns
         ↓
Add calculated columns (Revenue, Cost, Profit)
         ↓
Aggregate (GROUP BY brand, month, region...)
         ↓
Visualise (charts, KPI cards, dashboard)
         ↓
Insight + Recommendation → Business Value
```

---

## 2. Data Cleaning Checklist

Run through every column of every table before any analysis.

### Check 1 — Missing / NULL Values
```sql
-- SQL: count nulls per column
SELECT COUNT(*) AS Total_Rows,
       SUM(CASE WHEN Brand IS NULL THEN 1 ELSE 0 END) AS Brand_Nulls,
       SUM(CASE WHEN Color IS NULL THEN 1 ELSE 0 END) AS Color_Nulls
FROM Products;
```
```excel
-- Excel: count blank cells
=COUNTBLANK(A2:A5200)
```
**Action:** Decide — fill with default, flag as "Unknown", or exclude from analysis?

---

### Check 2 — Duplicate IDs
```sql
-- SQL: find duplicate IDs
SELECT ProductID, COUNT(*) AS Count
FROM Products
GROUP BY ProductID
HAVING COUNT(*) > 1;
```
```excel
-- Excel: highlight duplicates
Home → Conditional Formatting → Highlight Cell Rules → Duplicate Values
```
**Action:** Keep first occurrence, remove duplicates, or investigate source system?

---

### Check 3 — Inconsistent Casing
```sql
-- SQL: standardise to Title Case (SQL Server)
UPDATE Customers SET Gender = UPPER(LEFT(Gender,1)) + LOWER(SUBSTRING(Gender,2,LEN(Gender)))
UPDATE Customers SET Region = LTRIM(RTRIM(Region))   -- also removes spaces
```
```excel
-- Excel: fix casing
=PROPER(A2)    -- Title Case → "North", "Female"
=UPPER(A2)     -- ALL CAPS
=LOWER(A2)     -- all lowercase
```
**Common culprits:** Gender, Region, City, Category columns

---

### Check 4 — Extra Spaces
```sql
-- SQL
UPDATE Customers SET Region = TRIM(Region);
```
```excel
-- Excel
=TRIM(A2)      -- removes leading, trailing, and double spaces
=LEN(A2)       -- use to spot cells that look identical but aren't
```

---

### Check 5 — Mixed Date Formats
```sql
-- SQL Server: convert various formats to standard date
SELECT CONVERT(DATE, SaleDate, 103)   -- DD/MM/YYYY format
SELECT CONVERT(DATE, SaleDate, 120)   -- YYYY-MM-DD format
SELECT TRY_CAST(SaleDate AS DATE)     -- safe conversion, returns NULL if fails
```
```excel
-- Excel: replace slashes with dashes first, then format as date
=SUBSTITUTE(A2, "/", "-")
-- Then: Format Cells → Date → YYYY-MM-DD
```
**Rule:** Always standardise to **YYYY-MM-DD** — it sorts correctly and is universally recognised.

---

### Check 6 — Wrong Data Types
```sql
-- SQL: change column data type
ALTER TABLE Sales ALTER COLUMN Revenue DECIMAL(15,2);
ALTER TABLE Sales ALTER COLUMN SaleDate DATE;
```
```excel
-- Excel: numbers stored as text won't SUM — fix by:
-- Select column → Data → Text to Columns → Finish
-- Or multiply by 1: =A2*1
```

---

### Check 7 — Outliers
```sql
-- SQL: find extreme values
SELECT * FROM Sales
WHERE SalesAmount > (SELECT AVG(SalesAmount) + 3 * STDEV(SalesAmount) FROM Sales);

-- Or simply check min/max
SELECT MIN(SalesAmount), MAX(SalesAmount), AVG(SalesAmount) FROM Sales;
```
```excel
-- Excel: sort descending, inspect top and bottom 10 rows
-- Use: =LARGE(D2:D5200, 1)  -- largest value
--      =SMALL(D2:D5200, 1)  -- smallest value
```

---

### Check 8 — Referential Integrity (Orphan Records)
```sql
-- Find Sales rows whose ProductID doesn't exist in Products
SELECT s.*
FROM Sales s
LEFT JOIN Products p ON s.ProductID = p.ProductID
WHERE p.ProductID IS NULL;

-- Find Sales rows whose CustomerID doesn't exist in Customers
SELECT s.*
FROM Sales s
LEFT JOIN Customers c ON s.CustomerID = c.CustomerID
WHERE c.CustomerID IS NULL;
```
```excel
-- Excel: VLOOKUP returns #N/A for orphan IDs — use COUNTIF
=COUNTIF(Products!$A:$A, B2)   -- returns 0 if ProductID not found
```

---

## 3. Excel Formulas

### Lookup Formulas

```excel
-- VLOOKUP: find value in first column, return from another column
=VLOOKUP(lookup_value, table_array, col_index_num, FALSE)

-- Example: bring Brand from Products into Sales (ProductID is in B2)
=VLOOKUP(B2, Products!$A:$F, 4, FALSE)
-- B2         = the ID to look up
-- Products!$A:$F = the reference table ($ locks it when dragging down)
-- 4           = column 4 of Products = Brand
-- FALSE       = exact match only (always use FALSE)

-- Bring Color (column 5):
=VLOOKUP(B2, Products!$A:$F, 5, FALSE)

-- Bring IncomeLevel from Customers (CustomerID is in C2):
=VLOOKUP(C2, Customers!$A:$G, 6, FALSE)

-- Handle errors gracefully:
=IFERROR(VLOOKUP(B2, Products!$A:$F, 4, FALSE), "Not Found")


-- INDEX MATCH: more powerful than VLOOKUP, works in any direction
=INDEX(return_column, MATCH(lookup_value, search_column, 0))

-- Example: same as VLOOKUP above but more flexible
=INDEX(Products!$D:$D, MATCH(B2, Products!$A:$A, 0))
-- Products!$D:$D = column to return values from (Brand)
-- B2            = value to search for
-- Products!$A:$A = column to search in (ProductID)
-- 0             = exact match


-- XLOOKUP: modern replacement (Excel 2019+), cleanest syntax
=XLOOKUP(lookup_value, lookup_array, return_array, "Not Found")

-- Example:
=XLOOKUP(B2, Products!$A:$A, Products!$D:$D, "Not Found")
```

**Which to use?**
| Formula | Use When |
|---------|----------|
| VLOOKUP | Simple, quick, widely compatible |
| INDEX MATCH | ID not in first column, or looking left |
| XLOOKUP | Excel 2019+ available, cleanest option |

---

### Aggregation Formulas

```excel
=SUM(D2:D5200)                          -- total
=AVERAGE(D2:D5200)                      -- mean average
=COUNT(D2:D5200)                        -- count numbers only
=COUNTA(A2:A5200)                       -- count non-empty cells
=COUNTBLANK(A2:A5200)                   -- count empty cells
=MAX(D2:D5200)                          -- largest value
=MIN(D2:D5200)                          -- smallest value
=MEDIAN(D2:D5200)                       -- middle value
=LARGE(D2:D5200, 3)                     -- 3rd largest value
=SMALL(D2:D5200, 3)                     -- 3rd smallest value


-- Conditional aggregation:
=COUNTIF(E2:E5200, "Apple")             -- count where Brand = Apple
=SUMIF(E2:E5200, "Apple", H2:H5200)    -- sum Revenue where Brand = Apple
=AVERAGEIF(E2:E5200, "Apple", H2:H5200) -- average Revenue where Brand = Apple

-- Multiple conditions:
=COUNTIFS(E2:E5200, "Apple", F2:F5200, "White")
-- count where Brand=Apple AND Color=White

=SUMIFS(H2:H5200, E2:E5200, "Apple", F2:F5200, "White")
-- sum Revenue where Brand=Apple AND Color=White
```

---

### Text Cleaning Formulas

```excel
=TRIM(A2)                    -- remove leading, trailing, double spaces
=PROPER(A2)                  -- Title Case ("north" → "North")
=UPPER(A2)                   -- ALL CAPS
=LOWER(A2)                   -- all lowercase
=LEN(A2)                     -- character count
=LEFT(A2, 3)                 -- first 3 characters
=RIGHT(A2, 4)                -- last 4 characters
=MID(A2, 2, 5)               -- 5 characters starting at position 2
=SUBSTITUTE(A2, "/", "-")    -- replace "/" with "-" (fix dates)
=CONCATENATE(A2, " ", B2)    -- join text: "Oyewo" + " " + "Lukman"
=A2 & " " & B2               -- same as CONCATENATE, shorter syntax
=FIND("@", A2)               -- position of "@" in text
=TRIM(PROPER(SUBSTITUTE(A2, "/", "-")))  -- chain multiple cleaners
```

---

### Date Formulas

```excel
=TODAY()                         -- today's date
=NOW()                           -- current date and time
=YEAR(A2)                        -- extract year (2023)
=MONTH(A2)                       -- extract month number (1-12)
=DAY(A2)                         -- extract day number
=TEXT(A2, "MMM")                 -- month name short ("Jan")
=TEXT(A2, "MMMM")                -- month name full ("January")
=TEXT(A2, "YYYY-MM-DD")          -- format date as text
=DATEDIF(A2, B2, "D")            -- days between two dates
=DATEDIF(A2, B2, "M")            -- months between two dates
=DATEDIF(A2, B2, "Y")            -- years between two dates
=EOMONTH(A2, 0)                  -- last day of same month
=NETWORKDAYS(A2, B2)             -- working days between dates
=WEEKDAY(A2, 2)                  -- day of week (1=Mon, 7=Sun)
```

---

### Conditional Logic Formulas

```excel
=IF(H2 > 0, "Profit", "Loss")

=IF(H2 > 10000, "High",
   IF(H2 > 5000, "Medium", "Low"))     -- nested IF (max 7 levels)

=IFS(H2 > 10000, "High",              -- cleaner than nested IF
     H2 > 5000,  "Medium",
     H2 <= 5000, "Low")

=IFERROR(formula, "fallback_value")   -- catch any error

=IFNA(VLOOKUP(...), "Not Found")      -- catch only #N/A errors

=AND(A2 > 0, B2 = "Apple")           -- TRUE only if BOTH conditions met
=OR(A2 > 0, B2 = "Apple")            -- TRUE if EITHER condition met
=NOT(A2 = "Apple")                    -- reverses TRUE/FALSE
```

---

### Financial / KPI Formulas

```excel
-- Revenue
=SalesAmount * Quantity

-- Cost
=UnitCost * Quantity

-- Profit
=Revenue - Cost

-- Profit Margin %
=(Revenue - Cost) / Revenue

-- Format as percentage: Ctrl+Shift+%

-- Year-on-Year Growth
=(ThisYear - LastYear) / LastYear

-- Running Total
=SUM($D$2:D2)    -- drag down — expands range as you go

-- % of Total
=D2 / SUM($D$2:$D$5200)    -- this row as % of grand total
```

---

## 4. SQL — Joins & Merging Tables

### The 4 Join Types

```
TABLE A (Sales)          TABLE B (Products)
┌─────────────┐          ┌─────────────┐
│ SaleID      │          │ ProductID   │
│ ProductID ──┼──────────┼─ ProductID  │
│ Quantity    │          │ Brand       │
│ SalesAmount │          │ Color       │
└─────────────┘          └─────────────┘
```

---

#### INNER JOIN — Only matching rows from BOTH tables
```sql
SELECT s.SaleID, s.Quantity, p.Brand, p.Color
FROM Sales s
INNER JOIN Products p ON s.ProductID = p.ProductID;
-- Rows with no match in either table are DROPPED
-- Use when: you only want complete, matched records
```

---

#### LEFT JOIN — All rows from LEFT table + matches from RIGHT
```sql
SELECT s.SaleID, s.Quantity, p.Brand, p.Color
FROM Sales s
LEFT JOIN Products p ON s.ProductID = p.ProductID;
-- ALL Sales rows kept
-- Unmatched rows show NULL for Brand and Color
-- Use when: you must preserve your main transaction table (most common in analytics)
```

---

#### RIGHT JOIN — All rows from RIGHT table + matches from LEFT
```sql
SELECT s.SaleID, p.ProductName, p.Brand
FROM Sales s
RIGHT JOIN Products p ON s.ProductID = p.ProductID;
-- ALL Products rows kept even if never sold
-- Unmatched Products show NULL for SaleID
-- Use when: finding products/customers with zero activity
```

---

#### FULL OUTER JOIN — All rows from BOTH tables
```sql
SELECT s.SaleID, p.ProductName
FROM Sales s
FULL OUTER JOIN Products p ON s.ProductID = p.ProductID;
-- Everything from both tables
-- Use when: auditing mismatches in both directions
```

---

### Merging All 3 Tables (Real Project Example)

```sql
-- Full Clean Sales Data: Sales + Products + Customers
SELECT
    s.SaleID,
    s.SaleDate,
    s.Quantity,
    s.SalesAmount,
    s.Unit_Cost,

    -- Calculated columns
    s.SalesAmount * s.Quantity                        AS Revenue,
    s.Unit_Cost   * s.Quantity                        AS Cost,
    (s.SalesAmount - s.Unit_Cost) * s.Quantity        AS Profit,

    -- From Products
    p.ProductName,
    p.Category,
    p.Brand,
    p.Color,

    -- From Customers
    c.CustomerName,
    c.Region,
    c.Gender,
    c.IncomeLevel

FROM Sales s
LEFT JOIN Products  p ON s.ProductID  = p.ProductID
LEFT JOIN Customers c ON s.CustomerID = c.CustomerID;
```

---

### Aliases (s, p, c)
Aliases are shorthand names for tables — they make queries readable:
```sql
FROM Sales s          -- "s" is now shorthand for Sales
LEFT JOIN Products p  -- "p" is shorthand for Products
-- Then use: s.SaleID, p.Brand, c.IncomeLevel
-- Instead of: Sales.SaleID, Products.Brand, Customers.IncomeLevel
```

---

## 5. SQL — Core Commands

### DDL — Define Structure
```sql
-- Create a database
CREATE DATABASE AviationDB;
USE AviationDB;

-- Create a table
CREATE TABLE flights (
    flight_id        INT PRIMARY KEY,
    flight_no        VARCHAR(10),
    airline          VARCHAR(50),
    origin           VARCHAR(3),
    destination      VARCHAR(3),
    dep_delay_min    INT,
    cancelled        INT,
    passengers       INT,
    load_factor_pct  DECIMAL(5,1),
    revenue_ngn      DECIMAL(15,2),
    profit_ngn       DECIMAL(15,2),
    sale_date        DATE
);

-- Add a column to existing table
ALTER TABLE flights ADD fuel_cost DECIMAL(15,2);

-- Delete entire table (irreversible — use with caution)
DROP TABLE flights;

-- Delete all data but keep structure
TRUNCATE TABLE flights;
```

### Common Data Types
| Type | Use For | Example |
|------|---------|---------|
| `INT` | Whole numbers | 250, 5200 |
| `DECIMAL(15,2)` | Money / precise numbers | 3106350.00 |
| `VARCHAR(50)` | Text up to 50 chars | "Air Peace" |
| `DATE` | Calendar date | 2023-01-01 |
| `BIT` | Yes/No, 0/1 | Cancelled flag |
| `FLOAT` | Approximate decimals | 76.4 |

---

### DML — Manipulate Data
```sql
-- Insert one row
INSERT INTO flights (flight_id, flight_no, airline, dep_delay_min, cancelled)
VALUES (1, 'P4350', 'Air Peace', 49, 0);

-- Insert multiple rows
INSERT INTO flights VALUES
    (2, 'QI201', 'Ibom Air', 5, 0),
    (3, 'UN303', 'United Nigeria', 0, 0);

-- Update existing data
UPDATE Customers
SET Gender = 'Female'
WHERE Gender = 'female';

-- Update multiple columns
UPDATE Customers
SET Gender = UPPER(LEFT(Gender,1)) + LOWER(SUBSTRING(Gender,2,100)),
    Region = TRIM(Region)
WHERE Gender IN ('female', 'male');

-- Delete specific rows
DELETE FROM Sales WHERE SaleID = 99;

-- Verify before deleting — run SELECT first
SELECT * FROM Sales WHERE SaleID = 99;
```

---

### Comments in SQL
```sql
-- This is a single-line comment

/* This is a
   multi-line comment */

SELECT
    s.SaleID,           -- transaction identifier
    s.SalesAmount,      -- unit price per item
    s.Quantity,         -- number of items purchased
    s.SalesAmount * s.Quantity AS Revenue   -- total transaction value
FROM Sales s;
```
> **Rule:** Comment every query you write. Your future self and teammates will thank you.

---

## 6. SQL — Aggregation & Grouping

### Core Aggregate Functions
```sql
COUNT(*)                       -- count all rows including nulls
COUNT(CustomerID)              -- count non-null values only
COUNT(DISTINCT CustomerID)     -- count unique values only
SUM(Revenue)                   -- total
AVG(Profit)                    -- average
MAX(SalesAmount)               -- highest value
MIN(SalesAmount)               -- lowest value
STDEV(SalesAmount)             -- standard deviation
ROUND(AVG(Profit), 2)          -- round to 2 decimal places
```

---

### GROUP BY — The Core Aggregation Tool
```sql
-- Brand by Profit (Chart 1 equivalent)
SELECT
    p.Brand,
    COUNT(*)                                          AS Total_Transactions,
    SUM(s.SalesAmount * s.Quantity)                   AS Total_Revenue,
    SUM(s.Unit_Cost * s.Quantity)                     AS Total_Cost,
    SUM((s.SalesAmount - s.Unit_Cost) * s.Quantity)   AS Total_Profit
FROM Sales s
LEFT JOIN Products p ON s.ProductID = p.ProductID
GROUP BY p.Brand
ORDER BY Total_Profit DESC;


-- Monthly Customers (Chart 4 equivalent)
SELECT
    MONTH(SaleDate)                     AS MonthNum,
    DATENAME(MONTH, SaleDate)           AS Month,
    COUNT(DISTINCT CustomerID)          AS Unique_Customers,
    SUM(SalesAmount * Quantity)         AS Revenue
FROM Sales
GROUP BY MONTH(SaleDate), DATENAME(MONTH, SaleDate)
ORDER BY MonthNum;


-- Income Level by Profit (Chart 5 equivalent)
SELECT
    c.IncomeLevel,
    SUM((s.SalesAmount - s.Unit_Cost) * s.Quantity) AS Total_Profit
FROM Sales s
LEFT JOIN Customers c ON s.CustomerID = c.CustomerID
GROUP BY c.IncomeLevel
ORDER BY Total_Profit DESC;
```

---

### HAVING — Filter After Grouping
```sql
-- WHERE filters rows BEFORE grouping
-- HAVING filters groups AFTER aggregation

-- Brands with profit over ₦150,000 only
SELECT Brand, SUM(Profit) AS Total_Profit
FROM CleanSales
GROUP BY Brand
HAVING SUM(Profit) > 150000
ORDER BY Total_Profit DESC;


-- Customers who made more than 20 purchases
SELECT CustomerID, COUNT(*) AS Purchase_Count
FROM Sales
GROUP BY CustomerID
HAVING COUNT(*) > 20
ORDER BY Purchase_Count DESC;
```

> **Rule:** If your condition uses SUM, COUNT, AVG etc. → use HAVING.  
> If it filters raw values → use WHERE.

---

## 7. SQL — Filtering

### WHERE Clause Conditions
```sql
-- Exact match
WHERE Brand = 'Apple'
WHERE Cancelled = 1

-- Comparison
WHERE Profit > 100000
WHERE Profit >= 0
WHERE Dep_Delay_Min < 15

-- Range
WHERE SaleDate BETWEEN '2023-01-01' AND '2023-03-31'
WHERE SalesAmount BETWEEN 100 AND 300

-- List
WHERE Brand IN ('Apple', 'Samsung', 'Dell')
WHERE IncomeLevel NOT IN ('Low')

-- Pattern matching
WHERE CustomerName LIKE 'Fatima%'      -- starts with Fatima
WHERE CustomerName LIKE '%Bello'       -- ends with Bello
WHERE CustomerName LIKE '%John%'       -- contains John

-- NULL checks
WHERE IncomeLevel IS NULL              -- find missing values
WHERE IncomeLevel IS NOT NULL          -- find populated values

-- Multiple conditions
WHERE Brand = 'Apple' AND Color = 'White'
WHERE Brand = 'Apple' OR Brand = 'Samsung'
WHERE Brand = 'Apple' AND (Color = 'White' OR Color = 'Gray')
```

---

### ORDER BY & LIMIT
```sql
ORDER BY Profit DESC          -- highest first
ORDER BY Profit ASC           -- lowest first
ORDER BY Brand ASC, Profit DESC   -- sort by Brand A-Z, then Profit high-low

-- Top 5 most profitable brands
SELECT Brand, SUM(Profit) AS Total_Profit
FROM CleanSales
GROUP BY Brand
ORDER BY Total_Profit DESC
LIMIT 5;                      -- MySQL / PostgreSQL

-- SQL Server equivalent:
SELECT TOP 5 Brand, SUM(Profit) AS Total_Profit
FROM CleanSales
GROUP BY Brand
ORDER BY Total_Profit DESC;
```

---

## 8. SQL — Advanced Queries

### CASE WHEN — IF/ELSE inside SQL
```sql
-- Classify customers by profit value
SELECT
    CustomerID,
    Profit,
    CASE
        WHEN Profit > 200 THEN 'High Value'
        WHEN Profit > 100 THEN 'Medium Value'
        ELSE 'Low Value'
    END AS Customer_Segment
FROM CleanSales;


-- Flag delayed vs on-time flights
SELECT
    FlightNo,
    Dep_Delay_Min,
    CASE
        WHEN Cancelled = 1           THEN 'Cancelled'
        WHEN Dep_Delay_Min <= 0      THEN 'On Time'
        WHEN Dep_Delay_Min <= 14     THEN 'Minor Delay'
        WHEN Dep_Delay_Min <= 60     THEN 'Moderate Delay'
        ELSE                              'Major Delay'
    END AS Delay_Category
FROM Flights;
```

---

### Subquery — A Query Inside a Query
```sql
-- Find all sales above average profit
SELECT SaleID, Brand, Profit
FROM CleanSales
WHERE Profit > (SELECT AVG(Profit) FROM CleanSales);


-- Find customers who bought more than the average customer
SELECT CustomerID, COUNT(*) AS Purchases
FROM Sales
GROUP BY CustomerID
HAVING COUNT(*) > (SELECT AVG(purchase_count)
                   FROM (SELECT CustomerID, COUNT(*) AS purchase_count
                         FROM Sales GROUP BY CustomerID) sub);
```

---

### CTE — Common Table Expression (Cleaner Subquery)
```sql
-- Same as subquery but easier to read and reuse
WITH BrandSummary AS (
    SELECT
        Brand,
        SUM(Profit) AS Total_Profit,
        SUM(Revenue) AS Total_Revenue
    FROM CleanSales
    GROUP BY Brand
)
SELECT *
FROM BrandSummary
WHERE Total_Profit > 150000
ORDER BY Total_Profit DESC;


-- Multiple CTEs chained
WITH
Revenue_CTE AS (
    SELECT Brand, SUM(Revenue) AS Total_Revenue FROM CleanSales GROUP BY Brand
),
Cost_CTE AS (
    SELECT Brand, SUM(Cost) AS Total_Cost FROM CleanSales GROUP BY Brand
)
SELECT r.Brand, r.Total_Revenue, c.Total_Cost,
       r.Total_Revenue - c.Total_Cost AS Profit
FROM Revenue_CTE r
JOIN Cost_CTE c ON r.Brand = c.Brand;
```

---

### Window Functions — Calculations Across Rows
```sql
-- Running total of revenue by date
SELECT
    SaleDate,
    Revenue,
    SUM(Revenue) OVER (ORDER BY SaleDate) AS Running_Total
FROM CleanSales;


-- Rank brands by profit
SELECT
    Brand,
    Total_Profit,
    RANK() OVER (ORDER BY Total_Profit DESC) AS Profit_Rank
FROM (SELECT Brand, SUM(Profit) AS Total_Profit FROM CleanSales GROUP BY Brand) t;


-- Row number per customer (first purchase, second purchase, etc.)
SELECT
    CustomerID,
    SaleDate,
    ROW_NUMBER() OVER (PARTITION BY CustomerID ORDER BY SaleDate) AS Purchase_Number
FROM Sales;
```

---

## 9. Power BI — DAX Formulas

### Basic Measures
```dax
Total Revenue   = SUM(Sales[Revenue])
Total Cost      = SUM(Sales[Cost])
Total Profit    = SUM(Sales[Profit])
Total Quantity  = SUM(Sales[Quantity])
Total Customers = DISTINCTCOUNT(Sales[CustomerID])
Total Orders    = COUNTROWS(Sales)
```

---

### Calculated Measures
```dax
Profit Margin % =
DIVIDE(SUM(Sales[Profit]), SUM(Sales[Revenue]), 0)
-- DIVIDE handles division by zero automatically (returns 0)

Average Order Value =
DIVIDE(SUM(Sales[Revenue]), COUNTROWS(Sales), 0)

Revenue per Customer =
DIVIDE(SUM(Sales[Revenue]), DISTINCTCOUNT(Sales[CustomerID]), 0)
```

---

### CALCULATE — The Most Important DAX Function
```dax
-- Filter to specific brand
Apple Revenue =
CALCULATE(
    SUM(Sales[Revenue]),
    Products[Brand] = "Apple"
)

-- Filter to multiple conditions
Apple White Revenue =
CALCULATE(
    SUM(Sales[Revenue]),
    Products[Brand] = "Apple",
    Products[Color] = "White"
)

-- Remove all filters (grand total regardless of slicer)
Total Revenue All =
CALCULATE(SUM(Sales[Revenue]), ALL(Sales))

-- % of total (respects other filters but ignores Brand filter)
Brand Revenue % =
DIVIDE(
    SUM(Sales[Revenue]),
    CALCULATE(SUM(Sales[Revenue]), ALL(Products[Brand]))
)
```

---

### Time Intelligence
```dax
-- Previous year comparison (requires proper date table)
Revenue Last Year =
CALCULATE(SUM(Sales[Revenue]), SAMEPERIODLASTYEAR('Date'[Date]))

YoY Growth % =
DIVIDE(
    SUM(Sales[Revenue]) - [Revenue Last Year],
    [Revenue Last Year],
    0
)

-- Year to Date
Revenue YTD =
TOTALYTD(SUM(Sales[Revenue]), 'Date'[Date])

-- Month to Date
Revenue MTD =
TOTALMTD(SUM(Sales[Revenue]), 'Date'[Date])
```

---

### Calculated Columns (Added to Table)
```dax
-- Add to Sales table
Revenue    = Sales[SalesAmount] * Sales[Quantity]
Cost       = Sales[Unit_Cost] * Sales[Quantity]
Profit     = Sales[Revenue] - Sales[Cost]
Month Name = FORMAT(Sales[SaleDate], "MMM")
Year       = YEAR(Sales[SaleDate])
MonthNum   = MONTH(Sales[SaleDate])

-- Classify profit
Customer Segment =
IF(Sales[Profit] > 200, "High Value",
    IF(Sales[Profit] > 100, "Medium Value", "Low Value"))
```

---

### FILTER Function
```dax
-- Revenue for High income customers only
High Income Revenue =
CALCULATE(
    SUM(Sales[Revenue]),
    FILTER(Customers, Customers[IncomeLevel] = "High")
)

-- Transactions above average profit
Above Average Sales =
CALCULATE(
    COUNTROWS(Sales),
    FILTER(Sales, Sales[Profit] > AVERAGE(Sales[Profit]))
)
```

---

## 10. Excel vs SQL vs DAX — Side by Side

| Task | Excel | SQL | DAX (Power BI) |
|------|-------|-----|----------------|
| Sum a column | `=SUM(D:D)` | `SUM(Revenue)` | `SUM(Sales[Revenue])` |
| Count rows | `=COUNTA(A:A)` | `COUNT(*)` | `COUNTROWS(Sales)` |
| Count unique | `=SUMPRODUCT(1/COUNTIF(A2:A100,A2:A100))` | `COUNT(DISTINCT ID)` | `DISTINCTCOUNT(Sales[ID])` |
| Filter & sum | `=SUMIF(E:E,"Apple",H:H)` | `SUM(...) WHERE Brand='Apple'` | `CALCULATE(SUM(...), filter)` |
| Multiple filter sum | `=SUMIFS(H:H,E:E,"Apple",F:F,"White")` | `SUM WHERE Brand='Apple' AND Color='White'` | `CALCULATE(SUM(...), filter1, filter2)` |
| If condition | `=IF(A2>0,"Yes","No")` | `CASE WHEN A>0 THEN 'Yes' ELSE 'No'` | `IF(condition, "Yes", "No")` |
| Average | `=AVERAGE(D:D)` | `AVG(column)` | `AVERAGE(Sales[column])` |
| Lookup value | `=VLOOKUP(B2,Table,4,FALSE)` | `LEFT JOIN ... ON ID = ID` | Relationship in data model |
| Group & sum | Pivot Table | `GROUP BY` + `SUM` | Visual with field drag |
| Remove spaces | `=TRIM(A2)` | `TRIM(column)` | `TRIM(column)` |
| Fix casing | `=PROPER(A2)` | `UPPER(LEFT(col,1))+LOWER(...)` | `UPPER(column)` |
| Profit margin | `=(Rev-Cost)/Rev` | `(SUM(Rev)-SUM(Cost))/SUM(Rev)` | `DIVIDE([Profit],[Revenue])` |
| Running total | `=SUM($D$2:D2)` | `SUM() OVER (ORDER BY date)` | `TOTALYTD(...)` |
| Rank | `=RANK(D2,D:D,0)` | `RANK() OVER (ORDER BY col DESC)` | `RANKX(ALL(table), measure)` |
| Replace text | `=SUBSTITUTE(A2,"/","-")` | `REPLACE(col, '/', '-')` | `SUBSTITUTE(col, "/", "-")` |

---

## 11. KPI Formulas

### Standard Business KPIs

```
Total Revenue      = SUM(SalesAmount × Quantity)
Total Cost         = SUM(UnitCost × Quantity)
Gross Profit       = Total Revenue − Total Cost
Profit Margin %    = (Gross Profit ÷ Total Revenue) × 100
Total Customers    = COUNT DISTINCT (CustomerID)
Average Order Value= Total Revenue ÷ Total Orders
Revenue per Customer = Total Revenue ÷ Unique Customers
```

### Aviation-Specific KPIs

```
On-Time Performance (OTP) % =
    (Flights with Dep_Delay ≤ 14 min) ÷ Total Flights × 100

Load Factor % =
    Passengers ÷ Capacity × 100

Cancellation Rate % =
    Cancelled Flights ÷ Total Flights × 100

Revenue per Available Seat Km (RASK) =
    Total Revenue ÷ (Capacity × Distance_km)

Cost per Available Seat Km (CASK) =
    Total Cost ÷ (Capacity × Distance_km)

Break-Even Load Factor =
    CASK ÷ Average Fare per Seat

Passenger-Minutes of Delay =
    SUM(Passengers × Dep_Delay_Min)   ← better than counting delay incidents
```

### Year-on-Year Growth
```
YoY Growth % = (This Period − Last Period) ÷ Last Period × 100

Example:
Jan 2024 Revenue: ₦1,200,000
Jan 2023 Revenue: ₦1,000,000
YoY Growth = (1,200,000 − 1,000,000) ÷ 1,000,000 × 100 = 20%
```

---

## 12. Portfolio Projects

### Project 1 — Excel Sales Analytics (TS Academy)
**File:** `excel-sales-analysis-ts-academy`  
**Dataset:** 5,200 sales transactions across 250 customers and 250 products  
**Tools:** Excel — VLOOKUP, SUMIFS, Pivot Tables, Charts, Dashboard  
**Key findings:**
- Apple most profitable brand (₦196,890 profit)
- White products highest revenue (₦1,218,350)
- Medium income segment most profitable (₦346,920)
- February worst performing month by revenue

---

### Project 2 — Aviation Operations Dashboard (Excel)
**File:** `nigerian-aviation-analytics`  
**Dataset:** 6,453 simulated domestic flight records — Jan–Jun 2025  
**Tools:** Excel — Data modelling, aggregation, 6-chart dashboard, KPI cards  
**Airlines:** Air Peace, Ibom Air, United Nigeria, Green Africa, Overland  
**Key findings:**
- Harmattan season (January) highest delay rates
- LOS–KAN highest revenue route but most delay-prone
- Technical delays longest average duration per incident
- Load factors averaged 76% across network

---

### Project 3 — Power BI Dashboard *(Coming Soon)*
**Tools:** Power BI — DAX measures, slicers, drill-through, interactive dashboard

### Project 4 — SQL Airline Queries *(Coming Soon)*
**Tools:** SQL Server — JOINs, CTEs, Window Functions, flight operations queries

---

## 🔗 Connect

- **GitHub:** [github.com/Luking007](https://github.com/Luking007)
- **LinkedIn:** [linkedin.com/in/oyewo-lukman](https://linkedin.com/in/oyewo-lukman)

---

*Last updated: April 2026 | TS Academy Data Analytics Programme*
