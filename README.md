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

![image](https://github.com/Ibrahim-netizen/Automation-of-Daily-Customers-data-into-Snowflake-Database/assets/76513466/207669a8-ed68-4c82-b076-1879f45e8de4)

2) Creating the Virtual Warehouse named CustomerWH and table, Customers to store the loaded data into.

![image](https://github.com/Ibrahim-netizen/Automation-of-Daily-Customers-data-into-Snowflake-Database/assets/76513466/14940958-b6ff-42cb-bca0-e01434a7bdd9)

3) The next step is to create a file format to identify CSV files only and staging area named Customer_stage to temporarily store the CSV files: 

![image](https://github.com/Ibrahim-netizen/Automation-of-Daily-Customers-data-into-Snowflake-Database/assets/76513466/0d403a85-dcd3-4b69-8328-2fc0ddbfbd82)

4) Data Loading: Loading the CSV file into the staging area with the code below in a local interpreter connected to snowflake (MS Visual Studio)
![image](https://github.com/Ibrahim-netizen/Automation-of-Daily-Customers-data-into-Snowflake-Database/assets/76513466/e90d1025-59c5-48a7-9bde-d5e2a6f7917b)

Put command is used to copy data from an external stage such as S3 bucket, Blob storage etc. into a snowflake stage.

5) Creating a Temporary table named Temp_Customers_data and remove duplicates with a Row_Number() windows function:

![image](https://github.com/Ibrahim-netizen/Automation-of-Daily-Customers-data-into-Snowflake-Database/assets/76513466/506224ed-7294-40a6-9c3b-d8986043f0e5)

6) Data Deduplication:

- Insert the records with row number as 1 such that they do not appear more than once:

![image](https://github.com/Ibrahim-netizen/Automation-of-Daily-Customers-data-into-Snowflake-Database/assets/76513466/27c52954-4412-478d-8f35-36dfbe5783f1)

- The below are the duplicate records found in the Temp_Customers_data:

![image](https://github.com/Ibrahim-netizen/Automation-of-Daily-Customers-data-into-Snowflake-Database/assets/76513466/7732b0e1-854e-4199-9ef6-37cb6e4a2763)


#### Automate daily incremental loading of data

Here, a task named Customers_load_task is created to automate the daily import of data where the Id does not match existing records in the Customers table, scheduled to run daily at 7am from Monday to Friday using the CRON expression.

![image](https://github.com/Ibrahim-netizen/Automation-of-Daily-Customers-data-into-Snowflake-Database/assets/76513466/4d86deda-9c75-44b4-bfda-13d4993a3588)

##### Managing Created Task

Customers_load_task created to run daily can be managed with the below commands: 

![image](https://github.com/Ibrahim-netizen/Automation-of-Daily-Customers-data-into-Snowflake-Database/assets/76513466/593f2def-a5c7-441c-a2c4-5dc3e8063975)





















