# Automation-of-Daily-Customers-data-into-Snowflake-Database
This project aims to create a database that stores the company’s customer details with incremental data loading in Snowflake. 

### Aim and Objectives

This project aims to create a database that stores the company's customer details who had a sale or not with incremental data loading into Snowflake. To complete this, the following objectives will be achieved:
- Create a CSV file with columns: id, customer_name, occupation, salary, sales flag (sales (1), no sales (0)).
- Create Snowflake objects: customer table, Virtual warehouse, Schema, and Database.
- Data Quality Assessment.
- Data Deduplication.
- Create task automation to regularly update the database with new records.

### Introduction 

The project is done to automate the daily import of new customer data who made sales or not to ensure the database is regularly updated.  

![image](https://github.com/Ibrahim-netizen/Automation-of-Daily-Customers-data-into-Snowflake-Database/assets/76513466/f82ee264-76e2-46ef-82cd-2034f400d979)

- The table shows attributes  of the dataset which includes ID, Name, Occupation, Salary and Sales_Flag.
- A record is considered a duplicate in the table if the record appears more than once.

### Data Quality Assessment

Data Quality Framework: Ensuring the data passes the data quality framework to meet the business requirement in terms of data conciseness, data normalization, data is free of errors, data is consistent, unique, complete, accurate, and valid.
Data dependency, Schema, and normalization.
Data Lifecycle: From staging area to temporary table to target table. Filtering data from the staged file before Setting constraints to table schema and file.

<img width="719" alt="image" src="https://github.com/Ibrahim-netizen/Automation-of-Daily-Customers-data-into-Snowflake-Database/assets/76513466/c3de4e25-374e-4479-8905-35df865e0022">


### Data Architecture 

The below demonstrates the structure of the tools and data lifecycle involved in the design of the initial upload of customer data into a database table named Customers as well as the task created to automate the incremental upload of subsequent data from new CSV files daily.

![image](https://github.com/Ibrahim-netizen/Automation-of-Daily-Customers-data-into-Snowflake-Database/assets/76513466/e608ff52-8885-41f0-8aa9-44c5ce8dcde4)


### ETL Process

#### 1) Creation of Snowflake Objects: 
After connecting to Snowflake, Snowflake objects such as the database to store tables and data, virtual warehouse for compute resources, schema to define the database design and its relationship with other tables and data models, and tables to set constraints which are CustomerDB,  CustomerWH, public, and Customers respectively created as shown below:

![image](https://github.com/Ibrahim-netizen/Automation-of-Daily-Customers-data-into-Snowflake-Database/assets/76513466/a3c9c2c4-c65e-4e72-95d6-88cfd0ea45d5)



2) Creating the Virtual Warehouse named CustomerWH and table, Customers to store the loaded data into.

![image](https://github.com/Ibrahim-netizen/Automation-of-Daily-Customers-data-into-Snowflake-Database/assets/76513466/0d2d6ad5-a6dd-46ee-9971-59e8085bdf85)


3) The next step is to create a file format to identify CSV files only and staging area named Customer_stage to temporarily store the CSV files: 

![image](https://github.com/Ibrahim-netizen/Automation-of-Daily-Customers-data-into-Snowflake-Database/assets/76513466/9e966f8c-b8d6-447f-970f-f0afe6c09be7)


4) Data Loading: Loading the CSV file into the staging area with the code below in a local interpreter connected to snowflake (MS Visual Studio)
   
![image](https://github.com/Ibrahim-netizen/Automation-of-Daily-Customers-data-into-Snowflake-Database/assets/76513466/87843b2e-c1d1-4825-903a-0168412ec6f2)


Put command is used to copy data from an external stage such as S3 bucket, Blob storage etc. into a snowflake stage.

5) Creating a Temporary table named Temp_Customers_data and remove duplicates with a Row_Number() windows function:

![image](https://github.com/Ibrahim-netizen/Automation-of-Daily-Customers-data-into-Snowflake-Database/assets/76513466/4b5fa05a-ea80-4871-8e54-315fb08fcfd2)


6) Data Deduplication:

- Insert the records with row number = 1 such that they do not appear more than once:

![image](https://github.com/Ibrahim-netizen/Automation-of-Daily-Customers-data-into-Snowflake-Database/assets/76513466/1b9883be-4001-4a3d-b56b-8d4d5e5c834d)

- The image below displays the duplicate records found in the Temp_Customers_data:

![image](https://github.com/Ibrahim-netizen/Automation-of-Daily-Customers-data-into-Snowflake-Database/assets/76513466/7732b0e1-854e-4199-9ef6-37cb6e4a2763)


#### Automate daily incremental loading of data

Here, a task named Customers_load_task is created to automate the daily import of data where the Id does not match existing records in the Customers table, scheduled to run daily at 7am from Monday to Friday using the CRON expression.

![image](https://github.com/Ibrahim-netizen/Automation-of-Daily-Customers-data-into-Snowflake-Database/assets/76513466/4032b589-870f-4619-83ac-d509e01d8302)


##### Managing Created Task

Customers_load_task created to run daily can be managed with the below commands: 

![image](https://github.com/Ibrahim-netizen/Automation-of-Daily-Customers-data-into-Snowflake-Database/assets/76513466/499550e9-b400-4e67-8859-57eb7e663789)


### Conclusion


This project demonstrates the ability of Snowflake to store incremental data loads as well as provide resources to store and query Big data. It also serve as solution for daily incremental loading task for instances where data is required to be collated into a central table daily for future use and ensure the database is kept up-to-date.

The process can be improved by improving the performance of the warehouse for when the data in the imported CSV file is large. Use copy command in task created for bulk loading of CSV files.






















