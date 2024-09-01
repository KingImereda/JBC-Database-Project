# Business Case: JBC Bank Credit Card Database.

## Problem Statement:

JBC Bank, a financial institution, was facing inefficiencies in managing their credit card transactions using a traditional Excel-based system. The growing volume of data and the limitations of Excel were hindering their ability to effectively analyze and leverage credit card information for business insights and decision-making.

## Proposed Solution:
To address these challenges, a SQL-based database solution was proposed. This database would provide a centralized, structured repository for credit card data, enabling efficient storage, retrieval, and analysis of information.

## Project Scope:
- Data Migration: Transfer the existing credit card data from the Excel file into a SSMS  database.
  
- Data Validation/Quality Check- Ensure data quality by cleaning and validating the migrated data to remove inconsistencies and errors.
  
- Database Normalization.
-- Create respective tables- Customer, Customer_Location, Merchant, Transaction.
-- Alter/update table.
-- Insert records.
  
- Database Design(ERD): Create a well-structured database schema-Entities,Attrributes and their relationship(s),to efficiently store and manage credit card data.
  
- Database Automation.
  - Triggers.
  - Store Procedures & Query Development: Develop SQL queries to extract, analyze, and report on credit card data for various business purposes as well as store complex SQL queries in store procedures.
 
- Performance Optimization: Optimize database performance through indexing[Clustered index, Non Clustered index], partitioning, Cache, Query tuning, and other techniques to handle large volumes of data 
  efficiently.
    
- Database Security, Users Management & Priviledges: Implement robust security measures to protect sensitive credit card information from unauthorized access.
  
- Backup & Recovery.

### NB: The proposed Database will be built on (SSMS)SQL Server Management Studio.


![Database](https://github.com/user-attachments/assets/8601c67c-2f0e-43ab-804c-8953937522ad)


## DATA MIGRATION.
i. Download and install SSMS application [Download here](https:learnmicrosoft.com/sqlserver)

ii. Create Database in SSMS installed Package.
```
-- DROP DATABASE IF ALREADY EXIST
DROP DATABASE IF EXISTS Credit_Card;
-- CREATE DATABASE Credit _Card
CREATE DATABASE Credit_Card;
```
iii. Prepare the Data for Migration:
Convert the excel file to CSV-UFC for database compatibility, efficiency, data integrity, and overall ease of the migration.

iv. Migrate Data from Excel to SSMS Database.
Move the existing credit card data from the CSV file into a SSMS database by taking the following steps.

- Step 1. Import Wizard: In SSMS, right-click on the Credit_Card database and select "Tasks" -> "Import Flat File".
- Step 2. Click Next.
- Step 3. Browse "location of the file to be imported".
- Step 4. Select "New Table Name" & "Table Schema" then  Click "Next".
- Step 5. Preview Data ,then Click "Next".
- Step 6. Modify Columns by doing column matching , ensuring the right data type for each column, setting constraint (e.g.Primary key , Allowing Null Values) for each column, then click "Next".
- Step 7. Then click "Finish".

v.   To verify data has been imported correctly, run the syntax.
```
< Select * from dbo.CC_Data >
```
In all 1,048,576 records with 23 columns were imported into the new table" CC_Data"

## DATA VALIDATION, CLEANING AND QUALITY CHECK.
To ensure data integrity and Accuracy, the following data quality and validation checke were carried out.

#### Check for  Missing Values:

```
Select
Count(Case WHEN Category is Null Then 1 END) As Category,
Count(Case WHEN CC_Number is Null Then 1 END) As CC_Number,
Count(Case WHEN Amount is Null Then 1 END) As Amt,
Count(Case WHEN FirstName is Null Then 1 END) As FirstName,
Count(Case WHEN LastName is Null Then 1 END) As LastName,
Count(Case WHEN Gender is Null Then 1 END) As Gender,
Count(Case WHEN Street is Null Then 1 END ) As Street,
Count(Case WHEN City is Null Then 1 END ) As City,
Count(Case WHEN [State] is Null Then 1 END) As [State],
Count(Case WHEN Zip is Null Then 1 END) As Zip,
Count(Case WHEN Lat is Null Then 1 END) As Lat,
Count(Case WHEN Long is Null Then 1 END) As Long,
Count(Case WHEN City_Population is Null Then 1 END) As [Population],
Count(Case WHEN Job is Null Then 1 END) As Job,
Count(Case WHEN DoB is Null Then 1 END) As DoB,
Count(Case WHEN Trans_num is Null Then 1 END) As Trans_num,
Count(Case WHEN Unix_time is Null Then 1 END) As Unix_time,
Count(Case WHEN Merch_lat is Null Then 1 END) As Merch_Lat,
Count(Case WHEN Merch_long is Null Then 1 END) As Merch_long,
Count(Case WHEN Fraud is Null Then 1 END) As No_of_Fraud,
Count(Case WHEN Merch_Zipcode is Null Then 1 END) As Merch_Zipcode
From dbo.CC_Data

```
#### Result : In all, there are 158,978 missing values. [Merch_zipcode] column account for (158,523) , [Lat] column account for (434) and [Merch_Lat] column acount for(1) missing value.

#### Upon further investigation, this result was checked by comparing it with independent  results the excel source file.

The findings was communicated to the Data Owners and management and the following resolves were taken.

- Real-time Validation : Due to the sensitivity of Credit Card Fraud "Not Null" constraint should be apply to all columns in the Credit Card table.
  
- That Merchant Latitude and Longitude columns should be use to triangulate location of the Merchants with missing value [Merch_zipcode] column .
  
- Geographic Proximity: Since we have the Merchant_Latitude and Merchant_Longitude, we should use a geocoding API to impute the missing merchant_Zipcode based on the closest known location.

### Check for Data Consistency in Amount Column.

```
SELECT *
FROM dbo.CC_Data
WHERE TRY_CAST(Amount AS DECIMAL) IS NULL;

```
#### Result: The syntax return Zero row, which implies that all values in the amount column can be converted to decimal places or is in decimal places.

#### Check for  Outliers in the Amount Column.
```
Select Avg (Amount) As Avg_Amount, Max(Amount) As Max_Amount, Min(Amount) As Min_Amount, STDEV(Amount) As STDEV_Amount
From dbo.CC_Data

```
#### Result : The syntax return the following result which shows the existence of Outliers that need to be further investigated.
```
Avg_Amount: 70.28
Max_Amount: 28,948.90
Min_Amount: 1
STDEV: 159.55
```
#### Check For Duplicate values:

```
SELECT
  COUNT(*) - COUNT(DISTINCT [Unnamed_0]) AS Duplicate_ID,
  COUNT(*) - COUNT(DISTINCT [CC_Number]) AS Duplicate_CC_Number,
  COUNT(*) - COUNT(DISTINCT [Merchant]) AS Duplicate_Merchant,
  COUNT(*) - COUNT(DISTINCT [Category]) AS Duplicate_Category,
  COUNT(*) - COUNT(DISTINCT [Amount]) AS Duplicate_Amount,
  COUNT(*) - COUNT(DISTINCT [FirstName]) AS Duplicate_FirstName,
  COUNT(*) - COUNT(DISTINCT [LastName]) AS Duplicate_LastName,
  COUNT(*) - COUNT(DISTINCT [Gender]) AS Duplicate_Gender,
  COUNT(*) - COUNT(DISTINCT [Street]) AS Duplicate_Street,
  COUNT(*) - COUNT(DISTINCT [City]) AS Duplicate_City,
  COUNT(*) - COUNT(DISTINCT [State]) AS Duplicate_State,
  COUNT(*) - COUNT(DISTINCT [Zip]) AS Duplicate_Zip,
  COUNT(*) - COUNT(DISTINCT [Lat]) AS Duplicate_Lat,
  COUNT(*) - COUNT(DISTINCT [Long]) AS Duplicate_Long,
  COUNT(*) - COUNT(DISTINCT [City_Population]) AS Duplicate_Population,
  COUNT(*) - COUNT(DISTINCT [Job]) AS Duplicate_Job,
  COUNT(*) - COUNT(DISTINCT [DoB]) AS Duplicate_DoB,
  COUNT(*) - COUNT(DISTINCT [Trans_num]) AS Duplicate_Trans_num,
  COUNT(*) - COUNT(DISTINCT [Unix_time]) AS Duplicate_Unix_time,
  COUNT(*) - COUNT(DISTINCT [Merch_lat]) AS Duplicate_Merch_lat,
  COUNT(*) - COUNT(DISTINCT [Merch_long]) AS Duplicate_Merch_long,
  COUNT(*) - COUNT(DISTINCT [Fraud]) AS Duplicate_Fraud,
  COUNT(*) - COUNT(DISTINCT [Merch_zipcode]) AS Duplicate_Merch_zipcode
FROM dbo.CC_Data;

```
#### Results: 
```
Columns	              Duplicate Count
Duplicate_ID            0
Duplicate_CC_Number	0
Duplicate_Merchant	1047882
Duplicate_Category	1048561
Duplicate_Amount	999973
Duplicate_FirstName	1048227
Duplicate_LastName	1048096
Duplicate_Gender	1048573
Duplicate_Street	1047610
Duplicate_City	        1047696
Duplicate_State	        1048524
Duplicate_Zip	        1047623
Duplicate_Lat	        1047626
Duplicate_Long	        1047624
Duplicate_Population	1047710
Duplicate_Job	        1048082
Duplicate_DoB	        1047625
Duplicate_Trans_num	0
Duplicate_Unix_time	17925
Duplicate_Merch_lat	112983
Duplicate_Merch_long	99247
Duplicate_Fraud 	1048573
Duplicate_Merch_zipcode	1020339
```
From the result, it can be deduced that only Unnamed_0 (CustomerID), CC_Number (Credit Card Number) and Trans-num (Transaction Number) columns have zero duplicate count, which will be used as Primary Key Columns.

### Data Cleaning:
Back-Up CC_Data Table.

#### I backed-up CC_Data table in Database Credit_Card before commencing data cleaning process.

```
Select * From dbo.CC_Data
Select * INTO CC_Data_backup
From dbo.CC_Data
```
#### - Rename Column [Unnamed_0] To  Column [CustomerID] 

```
EXEC sp_rename 'CC_Data.Unnamed_0', 'CustomerID', 'COLUMN';
```
#### Rename Column [Trans_date_trans_time] To Column [Trans_Date_Time]
```
EXEC sp_rename 'CC_Data.Trans_date_trans_time', 'Trans_Date_Time', 'COLUMN';

```
#### Cleaning The Merchant Column to remove unwanted character " Fraud_":

```
UPDATE CC_Data
SET Merchant = SUBSTRING(Merchant, CHARINDEX('_', Merchant) + 1, LEN(Merchant));

```

#### Alter the table to ADD MerchantID Column

```
ALTER TABLE CC_Data
ADD MerchantID INT IDENTITY(1,1) PRIMARY KEY;

```

#### Update the Amount Column to 2 decimal places

```
  UPDATE CC_Data
SET Amount = ROUND(Amount, 2);

```
#### Update the Latitude,Longitude, Merchant_Latitude and Merch_Longitude columns into 4 decimal places.

```
UPDATE CC_Data
SET Lat = ROUND(Lat, 4)

UPDATE CC_Data
SET Long = ROUND(Long, 4)

UPDATE CC_Data
SET Merch_lat = ROUND(Merch_lat, 4)

UPDATE CC_Data
SET Merch_long = ROUND(Merch_long, 4)

```
 
## DATABASE NORMALIZATION.
After successfully migrating the data from a source file into a Database Table-CC_Data, and conducting Data Validation, Cleaning and Quality Check. I noticed that the  data is Denormalize(Serve multiple Purpose  i.e. It contains information about the Customer, Customer_Location, Merchant & Transaction, Which is an anomaly). 

A database table should only serve a single purpose , hence the Denormalize table is Normalize by breaking it into Customer, Merchant, Customer-Location, Transaction Table. This brings me to the next step-Data Normalization.

Data Normalizatiom is the process of breaking down a large table into smaller, related tables to reduce duplication and ensure that data is stored in a consistent and efficient manner. Normalization optimizes the database design by creating a single purpose for each table . This helps prevent Insert, Update and Delete errors or inconsistencies in the database development.

#### Back-UP initial table before commencing the Normalization Process.

```
Select * From dbo.CC_Data

Select * INTO CC_Data2_backup
From dbo.CC_Data

```
SQL Script use to Normalize the table is below:

```
Drop Table IF EXISTS CUSTOMER
CREATE TABLE CUSTOMER (
    CC_Number BIGINT PRIMARY KEY, 
    Gender VARCHAR(10),
    FirstName VARCHAR(max),
    LastName VARCHAR(max),
    Job VARCHAR(max),
    DoB DATE
   
);

INSERT INTO CUSTOMER
SELECT DISTINCT CC_Number, Trans_Date_Time, Gender, FirstName, LastName, Job, DoB
FROM CC_Data;
```
```
Drop Table IF EXISTS CUSTOMER_LOCATION
CREATE TABLE CUSTOMER_LOCATION (
    CustomerID BIGINT PRIMARY KEY, 
    Street Varchar (225),
	City Nvarchar (255),
	Zip Int,
    Lat float,
    Long float,
   
);

INSERT INTO CUSTOMER_LOCATION
SELECT DISTINCT CustomerID, Street, City, Zip, Lat, Long
FROM CC_Data;
```
```
Drop Table IF EXISTS MERCHANT
CREATE TABLE MERCHANT (
    MerchantID BIGINT PRIMARY KEY, 
    Merchant Varchar (225),
	Category Varchar (255),
    Merch_Lat float,
    Merch_Long float,
    Merch_Zipcode Int,
);

INSERT INTO MERCHANT
SELECT DISTINCT MerchantID, Merchant, Category, Merch_lat, Merch_long, Merch_zipcode
FROM CC_Data;
```

```
Drop Table IF EXISTS TRANSACTIONS
CREATE TABLE TRANSACTIONS (
   Trans_num Nvarchar (255),
   Trans_Date_Time datetime null,
   Amount fLOAT,
   City_Population Int,
   Fraud Int,
   CC_Number Bigint Foreign Key REFERENCES CUSTOMER,
   MerchantID Bigint Foreign Key REFERENCES MERCHANT,
   CustomerID Bigint Foreign Key REFERENCES CUSTOMER_LOCATION,
);

INSERT INTO TRANSACTIONS
SELECT DISTINCT Trans_num, Trans_Date_Time, Amount, City_Population, Fraud, CC_Number, MerchantID, CustomerID
FROM CC_Data;

```
The initial table Has been Normalize into the following Entities Structures with there respective columns
```
CUSTOMER            CUSTOMER_LOCATION       MERCHANT              TRANSACTIONS
CC_Number              CustomerID              MerchantID            Trans_num
DoB                    Street                  Merchant              Amount
Gender                 City                    Category              City_Population
FirstName              Zip                     Merch_lat             Fraud
LastName               Lat                     Merch_long            CC_Number
Job                    Long                    Merch_Zipcode         MerchantID
DoB                                                                  CustomerID
								     Trans_Date_Time

```
## ENTITY RELATION DIAFRAM (ERD).
The initial table has been divide into CUSTOMER, CUSTOMER_LOCATION, MERCHANT, TRANSACTIONS Table. Then, determine the relationships among these tables through an Entity Relationship Diagram (ERD)

Using the Database Diagram features in SSMS, I was able to automatically create an Entity Relation Diagram below a #One-To- Many relationship between the Fact(Transaction Table) and the Dimensions Tables.



![Screenshot 2024-08-29 222740](https://github.com/user-attachments/assets/c4dc433d-dce3-40fe-8d68-abe284473f01)


## DATABASE AUTOMATION.

By automating database tasks, we can significantly streamline our operations, minimize errors, and optimize our database performance. This will lead to improved efficiency, accuracy, and overall management of our database environment. Specific areas of focus include automating routine tasks like backups, performance tuning, and data migration, Updates.

### TRIGGERS:
#### A Trigger that declines online(net) transaction amount above 20000 from card user.
```
CREATE TRIGGER trg_decline_high_amount_transactions
ON Transactions
INSTEAD OF INSERT
AS
BEGIN
    DECLARE @merchant_category VARCHAR(MAX);
    DECLARE @amount DECIMAL(18, 2);

    -- Retrieve the merchant's category and the transaction amount
    SELECT @merchant_category = M.category, @amount = I.Amount
    FROM Merchant M
    JOIN INSERTED I ON M.MerchantID = I.MerchantID;

    -- Check if the category is 'misc_net' or 'shopping_net' and amount exceeds 20000
    IF (@merchant_category IN ('misc_net', 'shopping_net') AND @amount > 20000)
    BEGIN
        RAISERROR('Transaction declined: Amount exceeds 20000 for category %s', 16, 1, @merchant_category);
        ROLLBACK TRANSACTION;
    END;
    ELSE
    BEGIN
        -- Insert the transaction if it does not meet the decline criteria
        INSERT INTO Transactions (MerchantID, Amount, Trans_Date_Time)
        SELECT MerchantID, Amount, Trans_Date_Time
        FROM INSERTED;
    END;
END;

```
####  A trigger that  prevent a credit card from being used more than 10 times per day:

```
CREATE TRIGGER trg_enforce_daily_usage_limit
ON Transactions
AFTER INSERT
AS
BEGIN
    DECLARE @daily_transaction_count INT;
    DECLARE @cc_number VARCHAR(20); -- Adjust the data type as per your table definition

    -- Calculate the number of transactions for the current day
    SELECT @cc_number = i.CC_Number
    FROM INSERTED i;

    SELECT @daily_transaction_count = COUNT(*)
    FROM Transactions
    WHERE CC_Number = @cc_number
      AND CAST(Trans_Date_Time AS DATE) = CAST(GETDATE() AS DATE);

    -- Check if the daily limit has been reached
    IF @daily_transaction_count >= 10
    BEGIN
        RAISERROR('Daily transaction limit of 10 exceeded for CC_Number: %s', 16, 1, @cc_number);
        ROLLBACK TRANSACTION;
    END
    ELSE
    BEGIN
        -- Insert the new transaction if the limit has not been reached
        INSERT INTO Transactions
        SELECT * FROM INSERTED;
    END
END;

```

### CREATE STORE PROCEDURES:

#### Identify high-value customers based on transaction frequency.

```
CREATE PROCEDURE sp_get_high_value_customers
AS
BEGIN
    SELECT C.CC_Number, C.FirstName, C.LastName, COUNT(T.Trans_num) AS transaction_count
    FROM Customer c
    JOIN Transactions t ON C.CC_Number = T.CC_Number
    GROUP BY C.CC_Number, C.FirstName, C.LastName
    HAVING COUNT(T.Trans_num) > 0
    ORDER BY transaction_count DESC;
END;
```

#### Identify high-value customers based on transaction Amount.
```
CREATE PROCEDURE sp_get_high_value_customers_by_amount
AS
BEGIN
    SELECT C.CC_Number, C.FirstName, C.LastName, SUM(T.Amount) AS transaction_count
    FROM Customer c
    JOIN Transactions t ON C.CC_Number = T.CC_Number
    GROUP BY C.CC_Number, C.FirstName, C.LastName
    HAVING SUM(T.Amount) > 20000
    ORDER BY transaction_count DESC;
END;
```

#### Group customers by location
```
CREATE PROCEDURE sp_get_customer_location_groups
AS
BEGIN
    SELECT cl.Street, cl.City, cl.Zip, COUNT(cl.CustomerID) AS customer_count
    FROM Customer_Location cl
    GROUP BY cl.Street, cl.City, cl.Zip
    ORDER BY customer_count DESC;
END;
```

#### Detect unusual transaction patterns or anomalies by location.
```
CREATE PROCEDURE sp_detect_unusual_transactions
AS
BEGIN
    SELECT T.CustomerID, T.Amount, T.Trans_Date_Time, cl.city
    FROM Transactions T
    JOIN Customer_Location cl ON T.CustomerID = cl.CustomerID
    WHERE (
        T.Amount > 50000 OR  -- High amount threshold
        T.Amount < 30 OR      -- Low amount threshold
        (T.Trans_Date_Time BETWEEN '2019-01-01 00:00:00.000' AND '2020-03-10 16:08:00.000')  -- Specific date range
        AND cl.city IN ('Edinburgh')  -- Specific city
    )
    ORDER BY T.Amount DESC;
END;
```

#### Identify potential fraudulent transactions based on location, amount, or time.

```
CREATE PROCEDURE sp_detect_potential_fraud
AS
BEGIN
    SELECT T.Trans_num, T.CustomerID, T.MerchantID, T.Amount, T.Trans_Date_Time, cl.city
    FROM Transactions T
    JOIN Customer_Location cl ON T.CustomerID = cl.CustomerID
    WHERE (
        -- High-risk locations
        cl.city NOT IN ('Grenada', 'Coyle') OR

        -- Unusual transaction amounts
        T.Amount > 200 OR
        T.Amount < 50000 OR

        -- Suspicious time patterns
        (T.Trans_Date_Time BETWEEN '2019-02-28 23:05:00.000' AND '2019-12-30 20:23:00.000' AND cl.city NOT IN ('Edinburg')) OR

        -- Multiple transactions in a short time
        (SELECT COUNT(*) FROM Transactions AS T2
         WHERE T2.CustomerID = T.CustomerID AND T2.Trans_Date_Time = T.Trans_Date_Time) > 5
    )
    ORDER BY T.Amount DESC;
END;

```
#### Analyze Merchant performance based on sales volume
```
CREATE PROCEDURE sp_get_merchant_performance
AS
BEGIN
    SELECT M.MerchantID, M.Merchant, SUM(T.amount) AS total_sales
    FROM Merchant M
    JOIN Transactions T ON M.MerchantID = T.MerchantID
    GROUP BY M.MerchantID, M.Merchant
    ORDER BY total_sales DESC;
END;

```
#### Identify top-performing and underperforming merchants.

```
CREATE PROCEDURE sp_get_merchant_performance_with_thresholds
(
    @high_threshold DECIMAL,
    @low_threshold DECIMAL
)
AS
BEGIN
    SELECT M.MerchantID, M.Merchant, SUM(T.amount) AS Total_sales
    FROM Merchant M
    JOIN Transactions T ON M.MerchantID = T.MerchantID
    GROUP BY M.MerchantID, M.Merchant
    HAVING SUM(T.Amount) > @high_threshold OR SUM(T.Amount) < @low_threshold
    ORDER BY SUM(T.Amount) DESC;
END;
```

## QUERY OPTIMIZATION
To enhance database query efficiency, indexes were implemented on specific columns. These indexes serve as navigational tools, accelerating the retrieval of data by optimizing search processes. The following SQL scripts were executed to create these indexes


```
--Customer_Index on Column CC_Number, FirstName, LastName
CREATE INDEX CUSTOMER_INDEX
ON CUSTOMER (CC_Number, FirstName, LastName);

```

```
--Merchant_Index on Column MerchantID,Merchant
CREATE INDEX MERCHANT_INDEX
ON MERCHANT (MerchantID, Merchant);

```

```
--Transaction_Index on Column Trans_num
CREATE INDEX TRANSACTIONS_INDEX
ON TRANSACTIONS (Trans_num);
```

#### Note:
The decision to index specific columns should be informed by a comprehensive analysis of your database's structure and usage patterns. Columns frequently involved in JOIN and WHERE clauses are prime candidates for indexing. However, remember that indexing choices are dynamic and may need to be adjusted as your database's usage evolves.


## DATABASE SECURITY, USER MANAGEMENT & PRIVILEDGES.
User management is a critical component of our database security infrastructure. By carefully managing user access, we can significantly reduce the risk of data breaches, unauthorized data manipulation, and operational disruptions. Effective user management ensures that only authorized individuals can interact with our database, safeguarding our sensitive information and maintaining the integrity of our systems.

The following scripts were use to create User Access and Priviledges.
#### Access Logins
create logins for each user who needs access to the SQL Server instance.
```
-- Create a login for a Database Administrator
CREATE LOGIN DBA_Login WITH PASSWORD = 'StrongPassword123';
```
```
-- Create a login for a Database Designer
CREATE LOGIN Designer_Login WITH PASSWORD = 'StrongPassword123';
```
```
-- Create a login for an End User
CREATE LOGIN EndUser_Login WITH PASSWORD = 'StrongPassword123';
```
```
-- Create a login for a Software Engineer
CREATE LOGIN Engineer_Login WITH PASSWORD = 'StrongPassword123';
```
```
-- Create a login for a Manager
CREATE LOGIN Manager_Login WITH PASSWORD = 'StrongPassword123';
```
```
-- Create a login for a Security Officer
CREATE LOGIN SecurityOfficer_Login WITH PASSWORD = 'StrongPassword123';
```
Create Database Users
#### Next, create database users for each login in the specific database.

```
--USE YourDatabase(Credit_Card);

-- Create a user for the Database Administrator
CREATE USER DBA_User FOR LOGIN DBA_Login;

-- Create a user for the Database Designer
CREATE USER Designer_User FOR LOGIN Designer_Login;

-- Create a user for the End User
CREATE USER EndUser_User FOR LOGIN EndUser_Login;

-- Create a user for the Software Engineer
CREATE USER Engineer_User FOR LOGIN Engineer_Login;

-- Create a user for the Manager
CREATE USER Manager_User FOR LOGIN Manager_Login;

-- Create a user for the Security Officer
CREATE USER SecurityOfficer_User FOR LOGIN SecurityOfficer_Login;
```

#### Assigninging Roles For Each User  Responsibilities.

```
-- Assign roles and permissions for the Database Administrator
ALTER ROLE db_owner ADD MEMBER DBA_User;

-- Assign roles and permissions for the Database Designer
ALTER ROLE db_datareader ADD MEMBER Designer_User;
ALTER ROLE db_datawriter ADD MEMBER Designer_User;

-- Assign roles and permissions for the End User
ALTER ROLE db_datareader ADD MEMBER EndUser_User;

-- Assign roles and permissions for the Software Engineer
ALTER ROLE db_datareader ADD MEMBER Engineer_User;
ALTER ROLE db_datawriter ADD MEMBER Engineer_User;

-- Assign roles and permissions for the Manager
ALTER ROLE db_datareader ADD MEMBER Manager_User;

-- Assign roles and permissions for the Security Officer
ALTER ROLE db_securityadmin ADD MEMBER SecurityOfficer_User;

```

## DATABASE BACKUP AND RECOVERY PLAN.

To ensure the uninterrupted and reliable operation of our banking systems, we've developed a comprehensive backup and recovery plan. This plan utilizes cutting-edge technology like Continuous Data Protection to

safeguard our data in real-time. Additionally, we've implemented offsite replication and a multi-tiered backup strategy in the Cloud e.g Azure SQL Database or Azure Cosmo DB to protect against potential

disasters. By investing in these robust measures, we're significantly reducing the risk of downtime and data loss, ensuring the continuity of our critical banking services and protecting our customers' 

financial information.








