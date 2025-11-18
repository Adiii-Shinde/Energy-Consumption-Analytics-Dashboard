
# Energy Consumption Analytics Dashboard

A modern end-to-end analytics project for global renewable energy consumption, leveraging **AWS S3**, **Snowflake**, and **Tableau**. This pipeline demonstrates cloud-based data storage, scalable data transformation, and interactive dashboarding for deep insights on energy usage, cost savings, and demographic trends.

***

## 📊 Project Overview

This project analyzes renewable energy usage at the household level, spanning multiple regions, countries, and energy sources (Hydro, Solar, Wind, Biomass, Geothermal). The workflow connects raw CSV data from AWS S3 to Snowflake for SQL-based transformation, then surfaces powerful insights with Tableau visualizations.

***

## 🚀 Architecture \& Tools

| Stage | Tool/Service |
| :-- | :-- |
| Data Storage | AWS S3 |
| Data Warehouse | Snowflake |
| Data Processing | Snowflake SQL |
| Visualization | Tableau |
| Integration | S3-Snowflake |
| Dataset Format | CSV |

**Architecture flow:**
CSV (AWS S3) → Snowflake Integration \& Stage → Data Cleaning/Transformation (SQL) → Analytical Tables → Tableau (via Snowflake connector) → Dashboard

***

## 🗂️ Dataset Description

**File:** `Renewable_Energy_Usage_Sampled.csv`

Key columns:

- Household information: `Household_ID`, `Region`, `Country`, `Household_Size`, `Income_Level`, `Urban_Rural`
- Usage metrics: `Energy_Source`, `Monthly_Usage_kWh`, `Year`, `Adoption_Year`, `Subsidy_Received`, `Cost_Savings_USD`
- Granularity: Household-month, rich metadata on demographics and energy access[^1]

***

## 🔗 Cloud Integration: Snowflake ↔️ AWS S3

### Step 1: AWS S3 Setup

- Upload `Renewable_Energy_Usage_Sampled.csv` to a dedicated S3 bucket.
- Create an **IAM role** for Snowflake with access to the S3 bucket.


### Step 2: Create Snowflake Storage Integration

```sql
CREATE OR REPLACE STORAGE INTEGRATION tableau_Integration
  TYPE = EXTERNAL_STAGE
  STORAGE_PROVIDER = 'S3'
  ENABLED = TRUE
  STORAGE_AWS_ROLE_ARN = 'arn:aws:iam::your-account-id:role/your-role-name'
  STORAGE_ALLOWED_LOCATIONS = ('s3://your-bucket-name/')
  COMMENT = 'Access for Tableau energy project';
```


### Step 3: Verify Integration

```sql
DESC INTEGRATION tableau_Integration;
```


### Step 4: Create Snowflake Stage

```sql
CREATE STAGE tableau.tableau_Data.tableau_stage
  URL = 's3://your-bucket-name/'
  STORAGE_INTEGRATION = tableau_Integration;
```


***

## 🛠️ Data Preparation \& Transformation (Snowflake SQL)

- Create utility schema/tables:

```sql
CREATE DATABASE tableau;
CREATE SCHEMA tableau_Data;

CREATE TABLE tableau_dataset (
    Household_ID STRING,
    Region STRING,
    Country STRING,
    Energy_Source STRING,
    Monthly_Usage_kWh FLOAT,
    Year INT,
    Household_Size INT,
    Income_Level STRING,
    Urban_Rural STRING,
    Adoption_Year INT,
    Subsidy_Received STRING,
    Cost_Savings_USD FLOAT
);
```

- Copy data from S3:

```sql
COPY INTO tableau_dataset
FROM @tableau_stage
FILE_FORMAT = (TYPE=csv FIELD_DELIMITER=',' SKIP_HEADER=1)
ON_ERROR = 'CONTINUE';
```

- Example transformation: Income-based augmentation

```sql
-- Increase Monthly Usage based on Income
UPDATE energy_consumption SET monthly_usage_kwh = monthly_usage_kwh * 1.1 WHERE income_level = 'Low';
UPDATE energy_consumption SET monthly_usage_kwh = monthly_usage_kwh * 1.2 WHERE income_level = 'Middle';
UPDATE energy_consumption SET monthly_usage_kwh = monthly_usage_kwh * 1.5 WHERE income_level = 'High';

-- Reduce Cost Savings by tier
UPDATE energy_consumption SET cost_savings_usd = cost_savings_usd * 0.9 WHERE income_level = 'Low';
UPDATE energy_consumption SET cost_savings_usd = cost_savings_usd * 0.8 WHERE income_level = 'Middle';
UPDATE energy_consumption SET cost_savings_usd = cost_savings_usd * 0.7 WHERE income_level = 'High';
```


***

## 📈 Tableau Dashboarding

1. Open Tableau
2. Connect to Snowflake, authenticate, and select the transformed/analytic tables
3. Build the dashboard with visualizations like:
    - **KWH by Country/Region**
    - **KWH/CSU by Energy Source**
    - **Usage \& Savings breakdowns by demographics, region, energy source, year**

**Sample dashboard preview:**
<img width="1512" height="871" alt="Energy Consumption Dasgboard" src="https://github.com/user-attachments/assets/2200fbd2-7bcb-4571-9f9c-23e3d08b7f9c" />


***

## 📑 Project Structure

```
energy-consumption-dashboard/
├── README.md
├── data/
│   └── Renewable_Energy_Usage_Sampled.csv
├── sql/
│   └── transformation.sql
├── images/
│   └── Energy-Consumption-Dashboard.jpg
└── Sql.txt
```


***

## 📝 Demo Queries (Snowflake)

```sql
-- Top countries by KWH
SELECT Country, SUM(Monthly_Usage_kWh) AS Total_KWH
FROM energy_consumption
GROUP BY Country
ORDER BY Total_KWH DESC;

-- CSU by Region
SELECT Region, SUM(Cost_Savings_USD) AS Total_CSU
FROM energy_consumption
GROUP BY Region
ORDER BY Total_CSU DESC;
```


***

## 🔐 Best Practices

- Restrict IAM permissions to minimum required for security.
- Set up Snowflake roles for table and stage access.
- Version SQL/ETL scripts for reproducibility.
- Document integration and transformation decisions.

***

## 🔗 References \& Resources

- [Snowflake S3 External Stage Docs](https://docs.snowflake.com/en/user-guide/data-load-s3)
- [Tableau Snowflake Integration](https://help.tableau.com/current/pro/desktop/en-us/examplesnowflake.htm)
- [AWS S3 IAM Roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html)

***

**Project maintained by: [Aditya.S]**
*Last updated: Nov 2025*
<span style="display:none">[^2][^3]</span>

<div align="center">⁂</div>

[^1]: Renewable_Energy_Usage_Sampled.csv

[^2]: Energy-Consumption-Dasgboard.jpg

[^3]: Sql.txt

