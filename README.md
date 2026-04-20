📊 Data Analytics Reference Guide
By Oyewo Lukman Segun | TS Academy 2026 | Aviation & Operations Analytics
Personal reference toolkit covering Excel, SQL, and Power BI DAX.
Built alongside the TS Academy Data Analytics Programme (Feb – May 2026).

📌 Table of Contents
The Analytical Framework
Data Cleaning Checklist
Excel Formulas
SQL — Joins & Merging Tables
SQL — Core Commands
SQL — Aggregation & Grouping
SQL — Filtering
SQL — Advanced Queries
Power BI — DAX Formulas
Excel vs SQL vs DAX — Side by Side
KPI Formulas
Portfolio Projects

1. The Analytical Framework
5 Questions to Ask Before Touching Any Data
#
Question
How to Check
1
What is the grain? What does one row represent?
Read column headers, check for duplicate IDs
2
What period does it cover?
SELECT MIN(date), MAX(date) FROM table
3
Are there nulls and do they make sense?
SELECT COUNT(*) WHERE column IS NULL
4
What is the baseline? Compare against last period / target / industry
Find benchmarks before reporting numbers
5
What decision will this answer?
Write your recommendation BEFORE building charts
Finding → Recommendation Framework
Code
The Full Data Journey
Code

2. Data Cleaning Checklist
Run through every column of every table before any analysis.
Check 1 — Missing / NULL Values
Sql
Excel
Action: Decide — fill with default, flag as "Unknown", or exclude from analysis?
Check 2 — Duplicate IDs
Sql
Excel
Action: Keep first occurrence, remove duplicates, or investigate source system?
Check 3 — Inconsistent Casing
Sql
Excel
Common culprits: Gender, Region, City, Category columns
Check 4 — Extra Spaces
Sql
Excel
Check 5 — Mixed Date Formats
Sql
Excel
Rule: Always standardise to YYYY-MM-DD — it sorts correctly and is universally recognised.
Check 6 — Wrong Data Types
Sql
Excel
Check 7 — Outliers
Sql
Excel
Check 8 — Referential Integrity (Orphan Records)
Sql
Excel

3. Excel Formulas
Lookup Formulas
Excel
Which to use?
| Formula | Use When |
|---------|----------|
| VLOOKUP | Simple, quick, widely compatible |
| INDEX MATCH | ID not in first column, or looking left |
| XLOOKUP | Excel 2019+ available, cleanest option |
Aggregation Formulas
Excel
Text Cleaning Formulas
Excel
Date Formulas
Excel
Conditional Logic Formulas
Excel
Financial / KPI Formulas
Excel

4. SQL — Joins & Merging Tables
The 4 Join Types
Code
INNER JOIN — Only matching rows from BOTH tables
Sql
LEFT JOIN — All rows from LEFT table + matches from RIGHT
Sql
RIGHT JOIN — All rows from RIGHT table + matches from LEFT
Sql
FULL OUTER JOIN — All rows from BOTH tables
Sql
Merging All 3 Tables (Real Project Example)
Sql
Aliases (s, p, c)
Aliases are shorthand names for tables — they make queries readable:

5. SQL — Core Commands
DDL — Define Structure
Sql
Common Data Types
Type
Use For
Example
INT
Whole numbers
250, 5200
DECIMAL(15,2)
Money / precise numbers
3106350.00
VARCHAR(50)
Text up to 50 chars
"Air Peace"
DATE
Calendar date
2023-01-01
BIT
Yes/No, 0/1
Cancelled flag
FLOAT
Approximate decimals
76.4
DML — Manipulate Data
Sql
Comments in SQL
Sql
Rule: Comment every query you write. Your future self and teammates will thank you.

6. SQL — Aggregation & Grouping
Core Aggregate Functions
Sql
GROUP BY — The Core Aggregation Tool
Sql
HAVING — Filter After Grouping
Sql
Rule: If your condition uses SUM, COUNT, AVG etc. → use HAVING.
If it filters raw values → use WHERE.

7. SQL — Filtering
WHERE Clause Conditions
Sql
ORDER BY & LIMIT

Sql
8. SQL — Advanced Queries

CASE WHEN — IF/ELSE inside SQL
Sql
Subquery — A Query Inside a Query
Sql
CTE — Common Table Expression (Cleaner Subquery)
Sql
Window Functions — Calculations Across Rows
Sql

9. Power BI — DAX Formulas
Basic Measures
Dax
Calculated Measures
Dax
CALCULATE — The Most Important DAX Function
Dax
Time Intelligence
Dax
Calculated Columns (Added to Table)
Dax
FILTER Function

10. Excel vs SQL vs DAX — Side by Side
Task
Excel
SQL
DAX (Power BI)
Sum a column
=SUM(D:D)
SUM(Revenue)
SUM(Sales[Revenue])
Count rows
=COUNTA(A:A)
COUNT(*)
COUNTROWS(Sales)
Count unique
=SUMPRODUCT(1/COUNTIF(A2:A100,A2:A100))
COUNT(DISTINCT ID)
DISTINCTCOUNT(Sales[ID])
Filter & sum
=SUMIF(E:E,"Apple",H:H)
SUM(...) WHERE Brand='Apple'
CALCULATE(SUM(...), filter)
Multiple filter sum
=SUMIFS(H:H,E:E,"Apple",F:F,"White")
SUM WHERE Brand='Apple' AND Color='White'
CALCULATE(SUM(...), filter1, filter2)
If condition
=IF(A2>0,"Yes","No")
CASE WHEN A>0 THEN 'Yes' ELSE 'No'
IF(condition, "Yes", "No")
Average
=AVERAGE(D:D)
AVG(column)
AVERAGE(Sales[column])
Lookup value
=VLOOKUP(B2,Table,4,FALSE)
LEFT JOIN ... ON ID = ID
Relationship in data model
Group & sum
Pivot Table
GROUP BY + SUM
Visual with field drag
Remove spaces
=TRIM(A2)
TRIM(column)
TRIM(column)
Fix casing
=PROPER(A2)
UPPER(LEFT(col,1))+LOWER(...)
UPPER(column)
Profit margin
=(Rev-Cost)/Rev
(SUM(Rev)-SUM(Cost))/SUM(Rev)
DIVIDE([Profit],[Revenue])
Running total
=SUM($D$2:D2)
SUM() OVER (ORDER BY date)
TOTALYTD(...)
Rank
=RANK(D2,D:D,0)
RANK() OVER (ORDER BY col DESC)
RANKX(ALL(table), measure)
Replace text
=SUBSTITUTE(A2,"/","-")
REPLACE(col, '/', '-')
SUBSTITUTE(col, "/", "-")

11. KPI Formulas
Standard Business KPIs
Code
Aviation-Specific KPIs
Code
Year-on-Year Growth

Code
12. Portfolio Projects
Project 1 — Excel Sales Analytics (TS Academy)
File: excel-sales-analysis-ts-academy
Dataset: 5,200 sales transactions across 250 customers and 250 products
Tools: Excel — VLOOKUP, SUMIFS, Pivot Tables, Charts, Dashboard
Key findings:
Apple most profitable brand (₦196,890 profit)
White products highest revenue (₦1,218,350)
Medium income segment most profitable (₦346,920)
February worst performing month by revenue
Project 2 — Aviation Operations Dashboard (Excel)
File: nigerian-aviation-analytics
Dataset: 6,453 simulated domestic flight records — Jan–Jun 2025
Tools: Excel — Data modelling, aggregation, 6-chart dashboard, KPI cards
Airlines: Air Peace, Ibom Air, United Nigeria, Green Africa, Overland
Key findings:
Harmattan season (January) highest delay rates
LOS–KAN highest revenue route but most delay-prone
Technical delays longest average duration per incident
Load factors averaged 76% across network
Project 3 — Power BI Dashboard (Coming Soon)
Tools: Power BI — DAX measures, slicers, drill-through, interactive dashboard
Project 4 — SQL Airline Queries (Coming Soon)
Tools: SQL Server — JOINs, CTEs, Window Functions, flight operations queries

🔗 Connect
GitHub: github.com/Luking007
LinkedIn: linkedin.com/in/oyewo-lukman
Last updated: April 2026 | TS Academy Data Analytics Programme
