# Automation-of-Daily-Customers-data-into-Snowflake-Database
The aim of this project is to create a database that stores company’s customer details with an incremental data loading in Snowflake. 

#### Aim and Objectives

The aim of this project is to create a database that stores company's customer details who had a sale or not with an incremental data loading into Snowflake. In order to complete this, the following objectives will be achieved:
- Create a CSV file with columns: : id, customer_name, occupation, salary, sales flag (sales (1), no sales (0)).
- Create Snowfake objects: customer table, Virtual warehouse, Schema and Database.
- Data Quality Assessment.
- Data Deduplication.
- Create task automation to regularly update database with new records.

#### Introduction 

The project is done to automate the daily import of new customer data who made sales or not to ensure the database is regularly updated.  

![image](https://github.com/Ibrahim-netizen/Automation-of-Daily-Customers-data-into-Snowflake-Database/assets/76513466/f82ee264-76e2-46ef-82cd-2034f400d979)

- The table shows attributes  of the dataset which includes ID, Name, Occupation, Salary and Sales_Flag.
- A record is considered duplicate in the table if the record appears more than once.

#### Data Quality Assesssment

Data Quality Framework: Ensuring the data passes the data quality framework to meet the business requirement in terms of data conciseness, data normalization, data is free of errors, data is consistent, unique, complete, accurate, valid.
Data dependency, Schema and normalization.
Data Lifecycle: From staging area to temporary table to target table. Filtering data from staged file before Setting constraints to table schema and file.

<img width="719" alt="image" src="https://github.com/Ibrahim-netizen/Automation-of-Daily-Customers-data-into-Snowflake-Database/assets/76513466/c3de4e25-374e-4479-8905-35df865e0022">

#### Data Architecture 

The below demonstrates the structure of the tools and data lifecycle involved in the design of the initial upload of customers data into a database table named Customers as well as the task created to automated the incremental upload of subsequent data from new CSV files on a daily basis.

![image](https://github.com/Ibrahim-netizen/Automation-of-Daily-Customers-data-into-Snowflake-Database/assets/76513466/e608ff52-8885-41f0-8aa9-44c5ce8dcde4)

#### ETL Process

1) Creation of Snowflake Objects: 
After connecting to snowflake, Snowflake objects such as the database to store table and data, virtual warehouse for compute resource, schema to define the database design and its relationship with other tables and data models, and tables to set constraints which are CustomerDB,  CustomerWH, public, and Customers respectively created as shown below:

-- Create Snowflake objects --

-- Delete database CustomerDB if it exists 
drop database if exists CustomerDB;

-- Create a database named CustomerDB 
create database CustomerDB;

/* confirm current the database and schema 
default schema is used for this database */

select current_database(), current_schema(); 

2) Creating the Virtual Warehouse named CustomerWH and table, Customers to store the loaded data into.

-- Create a warehouse to provide compute resources to run queries

create or replace warehouse CustomerWH with 
warehouse_size = 'x-small'
Auto_suspend = 180
Auto_resume = True
Initially_suspended = True;

-- Return the current database we are working on 
select current_warehouse(); 

-- Delete table CustomerDB if it exists already in the worksheet 
drop table if exists Customers;

-- Create Customers table to import data into 

create table Customers(
    Id int primary key,
    Name varchar(100),
    Occupation varchar(50),
    Salary int,
    sales_flag INTEGER
);


3) The next step is to create a file format to identify CSV files only and staging area named Customer_stage to temporarily store the CSV files: 

-- Create file format 

create or replace file format mycsvformat
Type = 'CSV'
Field_Delimiter = ','
Skip_header = 1;

-- Create a staging area to temporarily store CSV file

create or replace stage Customer_stage
file_format = mycsvformat;


4) Data Loading: Loading the CSV file into the staging area with the code below in a local interpreter connected to snowflake (MS Visual Studio)
   
PUT file://C:\Users\giwai\Downloads\Customers.csv @"CUSTOMERDB"."PUBLIC".Customer_stage;

Put command is used to copy data from an external stage such as S3 bucket, Blob storage etc. into a snowflake stage.

5) Creating a Temporary table named Temp_Customers_data and remove duplicates with a Row_Number() windows function:

-- Create a Temporary Temp_Customers_data table to temporarily store data and remove duplicates
CREATE OR REPLACE TEMPORARY TABLE Temp_Customers_data AS
SELECT *, ROW_NUMBER() OVER (PARTITION BY Name ORDER BY ID) AS rn
FROM @customer_stage/Customers.csv.gz;

6) Data Deduplication:

- Insert the records with row number = 1 such that they do not appear more than once:

-- Copy back the non-duplicate records to the original table
INSERT INTO Customers
SELECT ID, NAME, OCCUPATION, SALARY, SALES_FLAG 
FROM Temp_Customers_data
WHERE Row_Num = 1;

- The image below displays the duplicate records found in the Temp_Customers_data:

![image](https://github.com/Ibrahim-netizen/Automation-of-Daily-Customers-data-into-Snowflake-Database/assets/76513466/7732b0e1-854e-4199-9ef6-37cb6e4a2763)


#### Automate daily incremental loading of data

Here, a task named Customers_load_task is created to automate the daily import of data where the Id does not match existing records in the Customers table, scheduled to run daily at 7am from Monday to Friday using the CRON expression.

-- Creating a Task to automate the daily incremental loading of data into 
CREATE TASK IF NOT EXISTS Customers_load_task
WAREHOUSE = CustomerWH
SCHEDULE = 'USING CRON 0 7 * * 1-5 GMT' 
AS
-- Step 2: Storing Data in a Temporary Table and Removing Duplicates
CREATE OR REPLACE TEMPORARY TABLE Temp_Customers_data AS
SELECT ID, Name, Occupation, Salary, Sales_Flag,
       ROW_NUMBER() OVER (PARTITION BY Name ORDER BY ID) AS rn
FROM temp_table;

-- Step 3: Inserting Deduplicated Data into Central Customers Table
MERGE INTO Customers AS target
USING (SELECT ID, Name, Occupation, Salary, Sales_Flag
       FROM Temp_Customers_data
       WHERE rn = 1) AS source
ON target.ID = source.ID
WHEN MATCHED THEN
  UPDATE SET
    Name = source.Name,
    Occupation = source.Occupation,
    Salary = source.Salary,
    Sales_Flag = source.Sales_Flag
WHEN NOT MATCHED THEN
  INSERT (ID, Name, Occupation, Salary, Sales_Flag)
  VALUES (source.ID, source.Name, source.Occupation, source.Salary, source.Sales_Flag)
  -- Drop the Temporary Table
DROP TABLE IF EXISTS Temp_Customers_data;

##### Managing Created Task

Customers_load_task created to run daily can be managed with the below commands: 

--View created task
Show Tasks;

-- To kickoff task

ALTER TASK Customers_load_task RESUME;

-- To end the task
ALTER TASK Customers_load_task SUSPEND;






















