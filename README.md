# HR-Analytics-Dashboard-Employee-Attrition-Performance-Insights

Project Overview
This project analyzes employee attrition and performance data from IBM's HR dataset using SQL Server and Power BI. The goal is to uncover key insights about why employees leave, which departments are most affected, and how salary and performance relate to attrition.
---
Dataset
Source: IBM HR Analytics Employee Attrition Dataset — Kaggle
Records: 1,470 employees
Features: 35 columns including age, department, job role, salary, attrition, performance rating, and more
---
Tools Used
Tool	Purpose
SQL Server (LocalDB)	Database creation and data modeling
SSMS	Writing and executing SQL queries
Power BI Desktop	Data visualization and dashboard
---
Data Architecture — Star Schema
The raw data was transformed into a star schema with the following tables:
```
RAW_HR (source)
    ├── DIM_Department   → Department name
    ├── DIM_Job          → Job role and level
    ├── DIM_Tenure       → Tenure band (0-1 Year, 1-3 Years, etc.)
    └── FACT_Employee    → Core employee metrics
```
---
SQL Highlights
Created a normalized star schema from flat CSV data
Used `BULK INSERT` to load raw data into SQL Server
Used `DISTINCT` and `CASE WHEN` to populate dimension tables
Used `JOIN` to link fact and dimension tables
---
DAX Measures
```dax
Total Employees = COUNTROWS(FACT_Employee)

Total Attrition = CALCULATE(COUNTROWS(FACT_Employee), FACT_Employee[Attrition] = "Yes")

Attrition Rate = DIVIDE([Total Attrition], [Total Employees], 0)

Avg Monthly Income = AVERAGE(FACT_Employee[MonthlyIncome])

Avg Job Satisfaction = AVERAGE(FACT_Employee[JobSatisfaction])
```
---
Dashboard Pages
Page 1 — Overview
KPI cards: Total Employees, Total Attrition, Attrition Rate, Avg Monthly Income
Attrition Rate by Department (bar chart)
Attrition Distribution Yes/No (donut chart)
Employee Count by Age (line chart)
Filters: Gender, Department, OverTime
Page 2 — Department & Role Analysis
Avg Monthly Income by Job Role
Attrition Rate by Job Role
Total Attrition by Job Role
Avg Monthly Income by Performance Rating
Filters: Department, Job Level
Page 3 — Employee Profile
Attrition Count by Age
Attrition by Gender (donut chart)
Attrition Count by Tenure Band
Total Attrition by Work Life Balance
Total Attrition by Marital Status
Filters: Marital Status, Tenure Band, Gender
---
Key Insights
Sales department has the highest attrition rate at 20.6%
Research & Development has the lowest attrition rate at 13.8%
Sales Representatives have the highest attrition rate among all job roles
Employees with lower work-life balance scores show significantly higher attrition
Single employees are more likely to leave compared to married or divorced employees
Managers earn the highest average monthly income, while Sales Representatives earn the lowest
---
Repository Structure
```
HR-Analytics-SQL-PowerBI/
│
├── README.md
├── sql/
│   ├── 01_create_database.sql
│   ├── 02_create_tables.sql
│   ├── 03_load_data.sql
│   ├── 04_populate_dimensions.sql
│   ├── 05_populate_fact.sql
│   └── 06_analysis_queries.sql
└── dashboard/
    └── HR_Analytics.pbix
```
---
Author
Buse Yılmaz 
Data Analyst  
