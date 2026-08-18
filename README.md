# Customer Churn & Retention Analysis Dashboard

## About This Project
An end-to-end churn analysis project for a telecom company. I built a full 
ETL pipeline in SQL Server to clean and prepare customer data, then designed 
an interactive Power BI dashboard to identify why customers leave and where 
the business should focus retention efforts.

## Business Question
- How many customers are churning, and what's the overall churn rate?
- Which customer segments (contract type, tenure, payment method) churn 
  the most?
- What are the top reasons customers leave?
- How much revenue is being lost to churn?

## Tech Stack
SQL Server, Power BI (Power Query, DAX), SSMS

## Data
Telecom customer dataset with 6,418+ records, including demographic info, 
account/contract details, services subscribed, billing information, and 
churn status/reason.

## Process

### 1. ETL Process in SQL Server
- Set up a SQL Server database and imported raw customer data into a 
  staging table
- Fixed BIT column data types to VARCHAR to resolve import errors
- Ran data exploration queries to check value distributions (Gender, 
  Contract, State, Customer Status) and identify null counts across all 
  columns
- Cleaned the data using COALESCE to replace nulls with sensible defaults 
  (e.g., missing service fields default to "No", missing churn category/
  reason default to "Others")
- Loaded the cleaned dataset into a production table (`prod_Churn`) used 
  for all downstream Power BI analysis

See [`sql/churn_etl_queries.sql`](sql/churn_etl_queries.sql) for the full 
set of queries.

### 2. Power BI Data Transformation (Power Query)
- Added calculated columns: Churn Status (binary flag from Customer 
  Status), Monthly Charge Range (bucketed into price tiers)
- Built reference/mapping tables for Age Group and Tenure Group with 
  custom sort ordering, so charts display in logical order instead of 
  alphabetically
- Established relationships between the main churn table, mapping tables, 
  and a dedicated measures table

### 3. DAX Measures
```dax
Total Customers = COUNT(prod_Churn[Customer_ID])

New Joiners = CALCULATE(
    COUNT(prod_Churn[Customer_ID]),
    prod_Churn[Customer_Status] = "Joined"
)

Total Churn = SUM(prod_Churn[Churn Status])

Churn Rate = DIVIDE([Total Churn], [Total Customers])

Revenue Lost to Churn = CALCULATE(
    SUM(prod_Churn[Total_Revenue]),
    prod_Churn[Customer_Status] = "Churned"
)
```

### 4. Dashboard Design
Built an interactive, single-page dashboard with:
- KPI cards: Total Customers, New Joiners, Total Churn, Churn Rate, 
  Revenue Lost to Churn
- Churn Rate by Contract Type, Payment Method, and Tenure Group
- Churn distribution by Age Group, Gender, and Churn Category
- Custom purple/violet color theme and formatted visuals for a clean, 
  professional look

## Key Findings
- Overall churn rate is **26.99%** (1,732 of 6,418 customers)
- **Month-to-Month contracts have by far the highest churn rate (46.53%)**, 
  compared to One Year (11.04%) and Two Year (2.73%) contracts — nearly a 
  17x difference between the riskiest and safest contract types
- **Competitor-related reasons** are the single largest driver of churn 
  (761 customers), followed by Attitude (301) and Dissatisfaction (300)
- Churn is costing the business an estimated **$3.41M** in lost revenue
- Churn rate stays relatively consistent across tenure groups (26-28%), 
  suggesting churn risk doesn't meaningfully decrease just because a 
  customer has stayed longer — contract type is a stronger predictor than 
  tenure alone

## Recommendation
Since month-to-month customers churn at nearly 17x the rate of two-year 
contract customers, the business should prioritize converting month-to-
month subscribers to longer-term contracts through targeted incentives or 
discounted annual pricing. Since "Competitor" is the top churn reason, 
introducing a competitive retention offer — triggered when a customer 
shows early churn risk signals — could meaningfully reduce losses. Given 
that tenure alone doesn't reduce churn risk, retention efforts should 
focus on contract type and competitive positioning rather than assuming 
loyalty naturally builds over time.

## Dashboard Preview
![Dashboard Page 1](screenshots/dashboard-page1.png)

## Files
- `Churn_analysis.pbix` — Power BI dashboard file
- `sql/churn_etl_queries.sql` — SQL Server ETL and data exploration scripts
- `screenshots/` — Dashboard preview images

## Tools Used
SQL Server, SQL Server Management Studio (SSMS), Power BI Desktop, 
Power Query, DAX
