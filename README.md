Business Case: JBC Bank Credit Card Database

Problem Statement:

JBC Bank, a financial institution, was facing inefficiencies in managing their credit card transactions using a traditional Excel-based system. The growing volume of data and the limitations of Excel were hindering their ability to effectively analyze and leverage credit card information for business insights and decision-making.

Proposed Solution:
To address these challenges, a SQL-based database solution was proposed. This database would provide a centralized, structured repository for credit card data, enabling efficient storage, retrieval, and analysis of information.

Project Scope:
1. Data Migration: Transfer the existing credit card data from the Excel file into a SSMS  database.
2.Data Validation/Quality Check- Ensure data quality by cleaning and validating the migrated data to remove inconsistencies and errors.
3. Database Normalization
- Create respective tables- Customer, Customer_Location, Merchant, Transaction
- Alter/update table
- Insert records
4.Database Design(ERD): Create a well-structured database schema-Entities,Attrributes and their relationship(s),to efficiently store and manage credit card data.
5. Performance Optimization: Optimize database performance through indexing[Clustered index, Non Clustered index], partitioning, Cache, Query tuning, and other techniques to handle large volumes of data efficiently.

6.Database Automation
-Triggers
- Store Procedures & Query Development: Develop SQL queries to extract, analyze, and report on credit card data for various business purposes as well as store complex SQL queries in store procedures.
7. Data Security: Implement robust security measures to protect sensitive credit card information from unauthorized access.
8.Backup & Recovery.

NB: The proposed Database will be built on (SSMS)SQL Server Management Studio

A. DATA MIGRATION
i. Download and install SSMS application [Download here](https:learnmicrosoft.com/sqlserver

ii. Create Database in SSMS installed Package

-- DROP DATABASE IF ALREADY EXIST
DROP DATABASE IF EXISTS Credit_Card;

-- CREATE DATABASE Credit _Card
CREATE DATABASE Credit_Card;

iii. Prepare the Data for Migration:
Convert the excel file to CSV-UFC for database compatibility, efficiency, data integrity, and overall ease of the migration.

iv. Migrate Data from Excel to SSMS Database
Move the existing credit card data from the CSV file into a SSMS database by taking the following steps.

Step 1. Import Wizard: In SSMS, right-click on the Credit_Card database and select "Tasks" -> "Import Flat File".
Step 2. Click Next
Step 3. Browse "location of the file to be imported".
Step 4. Select "New Table Name" & "Table Schema" then  Click "Next"
Step 5. Preview Data ,then Click "Next"
Step 6. Modify Columns by doing column matching , ensuring the right data type for each column, setting constraint (e.g.Primary key , Allowing Null Values) for each column, then click     "Next"
Step 7. Then click "Finish".

v.  Verify Data Import : Run the syntax < Select * from dbo.CC_Data > to verify data has been imported correctly. In all 1,048,576 records with 23 columns were imported into table" CC_Data"

B. DATA VALIDATION, CLEANING AND QUALITY CHECK.
To ensure data integrity and Accuracy, the following data quality and validation are carried out

B1i. Check for  Missing Values:

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
Result : In all there are 158,978 missing values. [Merch_zipcode] column account for (158,523) , [Lat] column account for (434) and [Merch_Lat] column acount for(1).
Upon further investigation, this result was corroborated by comparing it with the excel source file.

This findings was communicated to the Data Owners and management and the following resolved were taken

- Real-time Validation : Due to the sensitivity of Credit Card Fraud "Not Null" constraint should be apply to all columns in the Credit Card table.
- That Merchant Latitude and Longitude columns should be use to triangulate location of the Merchants with missing value [Merch_zipcode] column .
- Geographic Proximity: Since we have the Merchant_Latitude and Merchant_Longitude, we should use a geocoding API to impute the missing merchant_Zipcode based on the closest known location.

B1ii. Check for Data Consistency in [Amount] Column

```
SELECT *
FROM dbo.CC_Data
WHERE TRY_CAST(Amount AS DECIMAL) IS NULL;

```
Result: The syntax return Zero row, which implies that all values in the amount column can be converted to decimal places or is in decimal places



B1iii. Check for  Outliers in the Amount Column
```
Select Avg (Amount) As Avg_Amount, Max(Amount) As Max_Amount, Min(Amount) As Min_Amount, STDEV(Amount) As STDEV_Amount
From dbo.CC_Data

```
Result : The syntax return the following result which shows the existence of Outliers that need to be further investigated.
Avg_Amount: 70.28
Max_Amount: 28,948.90
Min_Amount: 1
STDEV: 159.55

B1iv. Check For Duplicate values:

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
Results: 
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

From the result, it can be deduce that only Unnamed_0 (CustomerID), CC_Number (Credit Card Number) and Trans-num (Transaction Number) columns have zero duplicate count, which will be use as Primary Key Columns

B2.Data Cleaning:
  Back-Up CC_Data Table

B2i. I backed-up CC_Data table in Database Credit_Card before commencing data cleaning process.

```
Select * From dbo.CC_Data


Select * INTO CC_Data_backup
From dbo.CC_Data

```
B2ii.- Rename Column [Unnamed_0] To  Column [CustomerID] & Column [Trans_date_trans_time] To Column [Trans_Date_Time]

```
EXEC sp_rename 'CC_Data.Unnamed_0', 'CustomerID', 'COLUMN';

EXEC sp_rename 'CC_Data.Trans_date_trans_time', 'Trans_Date_Time', 'COLUMN';

```
B2iii. - Cleaning The Merchant Column to remove unwanted character " Fraud_":

```
UPDATE CC_Data
SET Merchant = SUBSTRING(Merchant, CHARINDEX('_', Merchant) + 1, LEN(Merchant));

```

B2iv. Alter the table to ADD MerchantID Column

```
ALTER TABLE CC_Data
ADD MerchantID INT IDENTITY(1,1) PRIMARY KEY;

```

B2v.Update the Amount Column to 2 decimal places

```
  UPDATE CC_Data
SET Amount = ROUND(Amount, 2);

```
B2v. Update the Latitude,Longitude, Merchant_Latitude and Merch_Longitude columns into 4 decimal places

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
 
C. DATABASE NORMALIZATION
After successfully migrating the data from an Excel file to a Database Table CC_Data, and conducting Data Validation, Cleaning and Quality Check I noticed that the  data is Denormalize(Serve multiple Purpose i.e It contain information about the Customer, Customer_Location, Merchant & Transaction, Which is an anomaly. A database table should only serve a single purpose , hence the Denormalize table is Normalize by breaking it into Customer, Merchant, Customer-Location, Transaction Table). This brings me to the next step Data Normalization.

Data Normalizatiom optimize the database design by creating a single purpose for each table . This helps prevent Insert, Update and Delete errors or inconsistencies in the data which are obtainable with Tables that has multiple purposes. Its main goal is to reduce redundancy and improve data integrity.

Back-UP Denormalize  table before commencing the Normalization Process

```Select * From dbo.CC_Data


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

   CUSTOMER            CUSTOMER_LOCATION       MERCHANT              TRANSACTIONS
CC_Number              CustomerID              MerchantID            Trans_num
DoB                    Street                  Merchant              Amount
Gender                 City                    Category              City_Population
FirstName              Zip                     Merch_lat             Fraud
LastName               Lat                     Merch_long            CC_Number
Job                    Long                    Merch_Zipcode         MerchantID
DoB                                                                  CustomerID
								     Trans_Date_Time

D. ENTITY RELATION DIAFRAM (ERD)
The initial table has been divide into CUSTOMER, CUSTOMER_LOCATION, MERCHANT, TRANSACTIONS Table. Then, determine the relationships among these tables through an Entity Relationship Diagram (ERD)

Using the Database Diagram features in SSMS, I was able to automatically create an Entity Relation Diagram below a #One-To- Many relationship between the Fact(Transaction Table) and the Dimensions Tables.

Entity Relation Diagram Picture



CREATING STORE PROCEDURE

CUSTOMER SEGMENTATION 

-- Identify high-value customers based on transaction frequency.

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

-- Identify high-value customers based on transaction Amount.
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

-- Group customers by location
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
```
-- Detect unusual transaction patterns or anomalies by location.

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

--Identify potential fraudulent transactions based on location, amount, or time.

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

MERCHANT QUERIES
```
-- Analyze Merchant performance based on sales volume

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
--Identify top-performing and underperforming merchants.

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

TRIGGERS








Benefits:
Improved Efficiency: Streamline credit card transaction processing and reduce manual errors.
Enhanced Data Management: Centralize credit card data, providing better organization and accessibility.
Enhanced Analytics: Enable deeper analysis of credit card data to identify trends, patterns, and opportunities.
Scalability: Accommodate future growth in credit card transactions without compromising performance.
Security: Protect sensitive credit card information with robust security measures.
By implementing a SQL-based database solution, JBC Bank can significantly improve their credit card operations, gain valuable insights, and enhance their overall customer experience.


