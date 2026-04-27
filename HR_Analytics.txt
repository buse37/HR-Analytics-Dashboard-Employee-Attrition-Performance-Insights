-- ============================================
-- HR Analytics Project
-- Tool: SQL Server Management Studio (SSMS)
-- Dataset: IBM HR Analytics - Kaggle
-- ============================================


-- ============================================
-- STEP 1: Create Database
-- ============================================

CREATE DATABASE HRAnalytics;
GO

USE HRAnalytics;
GO


-- ============================================
-- STEP 2: Create Raw Table
-- ============================================

CREATE TABLE RAW_HR (
    Age INT,
    Attrition NVARCHAR(5),
    BusinessTravel NVARCHAR(50),
    DailyRate INT,
    Department NVARCHAR(100),
    DistanceFromHome INT,
    Education INT,
    EducationField NVARCHAR(100),
    EmployeeCount INT,
    EmployeeNumber INT,
    EnvironmentSatisfaction INT,
    Gender NVARCHAR(10),
    HourlyRate INT,
    JobInvolvement INT,
    JobLevel INT,
    JobRole NVARCHAR(100),
    JobSatisfaction INT,
    MaritalStatus NVARCHAR(20),
    MonthlyIncome INT,
    MonthlyRate INT,
    NumCompaniesWorked INT,
    Over18 NVARCHAR(5),
    OverTime NVARCHAR(5),
    PercentSalaryHike INT,
    PerformanceRating INT,
    RelationshipSatisfaction INT,
    StandardHours INT,
    StockOptionLevel INT,
    TotalWorkingYears INT,
    TrainingTimesLastYear INT,
    WorkLifeBalance INT,
    YearsAtCompany INT,
    YearsInCurrentRole INT,
    YearsSinceLastPromotion INT,
    YearsWithCurrManager INT
);


-- ============================================
-- STEP 3: Load CSV Data into Raw Table
-- ============================================

BULK INSERT RAW_HR
FROM 'C:\Users\yilma\Desktop\HR-Employee.csv'
WITH (
    FIRSTROW = 2,
    FIELDTERMINATOR = ';',
    ROWTERMINATOR = '\n',
    TABLOCK
);

-- Verify
SELECT COUNT(*) AS TotalRows FROM RAW_HR;


-- ============================================
-- STEP 4: Create Dimension Tables
-- ============================================

-- Department dimension
CREATE TABLE DIM_Department (
    DepartmentID   INT PRIMARY KEY IDENTITY(1,1),
    DepartmentName NVARCHAR(100)
);

-- Job dimension
CREATE TABLE DIM_Job (
    JobID    INT PRIMARY KEY IDENTITY(1,1),
    JobRole  NVARCHAR(100),
    JobLevel INT
);

-- Tenure dimension
CREATE TABLE DIM_Tenure (
    TenureID   INT PRIMARY KEY IDENTITY(1,1),
    TenureBand NVARCHAR(50)
);


-- ============================================
-- STEP 5: Populate Dimension Tables
-- ============================================

-- Populate departments
INSERT INTO DIM_Department (DepartmentName)
SELECT DISTINCT Department FROM RAW_HR;

-- Populate job roles
INSERT INTO DIM_Job (JobRole, JobLevel)
SELECT DISTINCT JobRole, JobLevel FROM RAW_HR;

-- Populate tenure bands
INSERT INTO DIM_Tenure (TenureBand)
SELECT DISTINCT
    CASE
        WHEN YearsAtCompany = 0              THEN '0-1 Year'
        WHEN YearsAtCompany BETWEEN 1 AND 3  THEN '1-3 Years'
        WHEN YearsAtCompany BETWEEN 4 AND 7  THEN '4-7 Years'
        WHEN YearsAtCompany BETWEEN 8 AND 15 THEN '8-15 Years'
        ELSE '15+ Years'
    END
FROM RAW_HR;


-- ============================================
-- STEP 6: Create and Populate Fact Table
-- ============================================

CREATE TABLE FACT_Employee (
    EmployeeID            INT PRIMARY KEY,
    DepartmentID          INT,
    JobID                 INT,
    TenureID              INT,
    Age                   INT,
    MonthlyIncome         INT,
    YearsAtCompany        INT,
    YearsWithCurrManager  INT,
    TrainingTimesLastYear INT,
    Attrition             NVARCHAR(5),
    PerformanceRating     INT,
    JobSatisfaction       INT,
    WorkLifeBalance       INT,
    OverTime              NVARCHAR(5),
    Gender                NVARCHAR(10),
    MaritalStatus         NVARCHAR(20)
);

INSERT INTO FACT_Employee (
    EmployeeID, DepartmentID, JobID, TenureID,
    Age, MonthlyIncome, YearsAtCompany, YearsWithCurrManager,
    TrainingTimesLastYear, Attrition, PerformanceRating,
    JobSatisfaction, WorkLifeBalance, OverTime, Gender, MaritalStatus
)
SELECT
    r.EmployeeNumber,
    d.DepartmentID,
    j.JobID,
    t.TenureID,
    r.Age,
    r.MonthlyIncome,
    r.YearsAtCompany,
    r.YearsWithCurrManager,
    r.TrainingTimesLastYear,
    r.Attrition,
    r.PerformanceRating,
    r.JobSatisfaction,
    r.WorkLifeBalance,
    r.OverTime,
    r.Gender,
    r.MaritalStatus
FROM RAW_HR r
JOIN DIM_Department d ON r.Department = d.DepartmentName
JOIN DIM_Job j ON r.JobRole = j.JobRole AND r.JobLevel = j.JobLevel
JOIN DIM_Tenure t ON (
    CASE
        WHEN r.YearsAtCompany = 0              THEN '0-1 Year'
        WHEN r.YearsAtCompany BETWEEN 1 AND 3  THEN '1-3 Years'
        WHEN r.YearsAtCompany BETWEEN 4 AND 7  THEN '4-7 Years'
        WHEN r.YearsAtCompany BETWEEN 8 AND 15 THEN '8-15 Years'
        ELSE '15+ Years'
    END = t.TenureBand
);

-- Verify
SELECT COUNT(*) AS TotalEmployees FROM FACT_Employee;


-- ============================================
-- STEP 7: Analysis Queries
-- ============================================

-- Attrition rate by department
SELECT
    d.DepartmentName,
    COUNT(*) AS TotalEmployees,
    SUM(CASE WHEN f.Attrition = 'Yes' THEN 1 ELSE 0 END) AS TotalAttrition,
    ROUND(100.0 * SUM(CASE WHEN f.Attrition = 'Yes' THEN 1 ELSE 0 END) / COUNT(*), 1) AS AttritionRate
FROM FACT_Employee f
JOIN DIM_Department d ON f.DepartmentID = d.DepartmentID
GROUP BY d.DepartmentName
ORDER BY AttritionRate DESC;

-- Average monthly income by job role
SELECT
    j.JobRole,
    ROUND(AVG(f.MonthlyIncome), 0) AS AvgMonthlyIncome,
    COUNT(*) AS TotalEmployees
FROM FACT_Employee f
JOIN DIM_Job j ON f.JobID = j.JobID
GROUP BY j.JobRole
ORDER BY AvgMonthlyIncome DESC;

-- Attrition rate by tenure band
SELECT
    t.TenureBand,
    COUNT(*) AS TotalEmployees,
    SUM(CASE WHEN f.Attrition = 'Yes' THEN 1 ELSE 0 END) AS TotalAttrition,
    ROUND(100.0 * SUM(CASE WHEN f.Attrition = 'Yes' THEN 1 ELSE 0 END) / COUNT(*), 1) AS AttritionRate
FROM FACT_Employee f
JOIN DIM_Tenure t ON f.TenureID = t.TenureID
GROUP BY t.TenureBand
ORDER BY AttritionRate DESC;

-- Attrition rate by gender
SELECT
    f.Gender,
    COUNT(*) AS TotalEmployees,
    SUM(CASE WHEN f.Attrition = 'Yes' THEN 1 ELSE 0 END) AS TotalAttrition,
    ROUND(100.0 * SUM(CASE WHEN f.Attrition = 'Yes' THEN 1 ELSE 0 END) / COUNT(*), 1) AS AttritionRate
FROM FACT_Employee f
GROUP BY f.Gender
ORDER BY AttritionRate DESC;

-- Attrition rate by marital status
SELECT
    f.MaritalStatus,
    COUNT(*) AS TotalEmployees,
    SUM(CASE WHEN f.Attrition = 'Yes' THEN 1 ELSE 0 END) AS TotalAttrition,
    ROUND(100.0 * SUM(CASE WHEN f.Attrition = 'Yes' THEN 1 ELSE 0 END) / COUNT(*), 1) AS AttritionRate
FROM FACT_Employee f
GROUP BY f.MaritalStatus
ORDER BY AttritionRate DESC;

-- Average income by performance rating
SELECT
    f.PerformanceRating,
    ROUND(AVG(f.MonthlyIncome), 0) AS AvgMonthlyIncome,
    COUNT(*) AS TotalEmployees
FROM FACT_Employee f
GROUP BY f.PerformanceRating
ORDER BY f.PerformanceRating;
