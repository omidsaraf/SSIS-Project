# ETL Project Using Northwind Dataset

In this ETL project, Northwind Dataset has been used as Datasource and followig steps involved in this process:
1. **Creating the Data Warehouse in SSMS**
2. **Creating Dimension and Fact tables (structure) using T-SQL in SSMS**
3. **Designing the Dimension loading process in Visual Studio:**
   - Designing the Control Flow, including Truncate Task and Data Flow Task.
   - Designing the Data Flow Task according to Attribute Types, including the data source (OLTP), SCD, and destination table.
   - Key Lookup using Stored Procedures for all Dimension tables.
4. **Designing the Fact loading process in Visual Studio:**
   - Enabling CDC for the relevant tables in OLTP.
   - Initial load by designing the Control Flow, including two CDC Tasks and one Data Flow Task (using key lookup and Stored Procedures).
   - Incremental load by designing the Control Flow, including two CDC Tasks and one Data Flow Task (recording changes), Truncate, and then starting the load with a Join in the data source, using the Key Lookup component for changeable columns, using Stored Procedures for Historical Fields, and finally Inferred Handling.
5. **Deployment (SSIS Catalog)**
   - During the installation of SQL Server and SSMS, in the Shared Features section, you need to check the SSIS Scale Out option to enable it.
   - SSIS Scale Out allows us to scale out a large SSIS Package and run it in parallel across multiple servers.
   - Create SSIS DB
   - Import Packages From Visual Studio into SSMS
   - Test Project Deployment
5. **Job (Automation)**
   - Enable SQL Server Agent Job.
   - Create Jobs to execute the SSIS packages.
   - Three Packages has been created incluing inital load for Dimention, initial load for Fact, Incremental load for fact
   - Schedule the Job according to your desired frequency and activate it.
## ETL Process
### Weekly and Initial load:
1.  **Full Load for Dimension Tables**:
   - The job1 triggers the SSIS package designed for the initial load of Dimension tables.
   - **Control Flow**: Includes tasks to truncate existing Dimension tables and load new data from the OLTP source.
   - **Data Flow Task**: Extracts data from the OLTP source, applies necessary transformations, and loads it into the Dimension tables in the Data Warehouse.

2.  **Full Load for Fact Table**:
   - The job2 triggers the SSIS package designed for the initial load of Fact tables.
   - **Control Flow**: Includes tasks to enable CDC, perform initial data extraction, and load data into Fact tables.
   - **Data Flow Task**: Extracts data from the OLTP source, performs key lookups, applies transformations, and loads data into the Fact tables.
     
### Daily (Incremental load):     
 **Incremental Load for Fact Table**:
   - The job triggers the SSIS package designed for the incremental load of Fact tables.
   - **Control Flow**: Includes tasks to capture changes using CDC, apply transformations, and load only the changed data into Fact tables.
   - **Data Flow Task**: Extracts changed data from the OLTP source, performs key lookups, applies transformations, and loads data into the Fact tables.
CDC Method:
1. **Start from Last LSN**:
   - Use CDC Control in Control Flow to read the last LSN (Log Sequence Number) and load the range from the last LSN to the current LSN into the CDC State table.
2. **Identify Changes**:
   - In Data Flow Task 1, activate CDC service, read the CDC table in the ETL Setting database, and store changes in the Change Table using `Union All`.
3. **Clear Changes from Fact**:
   - In Control Flow, remove changes from the Fact table. In Data Flow Task 2, perform an initial load of changed records from OLTP into the Data Warehouse using a join, with a condition `Where ID IN (SELECT ID FROM V_Changes)`.
4. **Mark LSN for Next Incremental Load**:
   - Store the transferred range in the CDC table in the ETL Setting database.
5. **Update Dimension Tables**:
   - Update Dimension tables in the Data Warehouse first, followed by Fact tables. Perform lookups in Dimension tables to get surrogate keys (SK) and use them as foreign keys in Fact tables

**Note**:
if Organisation 24/7 have Oprational not stop system that contiously generate Transactional Data then:
- Use Snapshot only for initial load to ensure static data capture at that moment.


### Outcome
The ETL process ensures that the Data Warehouse is consistently updated with the latest data from the OLTP source. Dimension tables are loaded first to ensure that Fact tables can reference the correct surrogate keys. The incremental load process minimizes downtime and ensures that only the latest changes are applied, maintaining the integrity and performance of the Data Warehouse. This setup supports efficient data analysis and reporting, providing accurate and up-to-date insights for business decision-making.



Here's the updated documentation with your detailed CDC method included:

---

# ETL Project Using Northwind Dataset

## Requirements

- **SQL Server Management Studio (SSMS)** with SSIS installation
- **SQL Server Engine** for OLTP, Staging, and Data Warehouse
- **Visual Studio** for creating ETL packages
- **SQL Server Agent** for job scheduling

## Project Overview

This ETL project uses the Northwind Dataset as the data source. The following steps are involved in this process:

### 1. Creating the Data Warehouse in SSMS
- Set up the Data Warehouse environment in SSMS.

### 2. Creating Dimension and Fact Tables
- Use T-SQL in SSMS to create the structure of Dimension and Fact tables.

### 3. Designing the Dimension Loading Process in Visual Studio
- **Control Flow**: Includes Truncate Task and Data Flow Task.
- **Data Flow Task**: Configured according to Attribute Types, including the data source (OLTP), Slowly Changing Dimension (SCD), and destination table.
- **Key Lookup**: Use Stored Procedures for all Dimension tables.

### 4. Designing the Fact Loading Process in Visual Studio
- **Enable CDC**: Enable Change Data Capture (CDC) for relevant tables in OLTP.
- **Initial Load**: Design Control Flow with two CDC Tasks and one Data Flow Task (using key lookup and Stored Procedures).
- **Incremental Load**: Design Control Flow with two CDC Tasks and one Data Flow Task (recording changes), Truncate, and start the load with a Join in the data source, using the Key Lookup component for changeable columns, Stored Procedures for Historical Fields, and Inferred Handling.

### 5. Deployment (SSIS Catalog)
- **SSIS Scale Out**: During SQL Server and SSMS installation, check the SSIS Scale Out option in the Shared Features section.
- **Create SSIS DB**: Set up the SSIS database.
- **Import Packages**: Import packages from Visual Studio into SSMS.
- **Test Deployment**: Ensure the project is correctly deployed.

### 6. Job Automation
- **Enable SQL Server Agent Job**: Activate SQL Server Agent.
- **Create Jobs**: Set up jobs to execute the SSIS packages.
  - **Initial Load for Dimension**: Job for initial load of Dimension tables.
  - **Initial Load for Fact**: Job for initial load of Fact tables.
  - **Incremental Load for Fact**: Job for incremental load of Fact tables.
- **Schedule Jobs**: Schedule the jobs according to the desired frequency.

## ETL Process

### Weekly and Initial Load

1. **Full Load for Dimension Tables**
   - **Job 1**: Triggers the SSIS package for the initial load of Dimension tables.
   - **Control Flow**: Tasks to truncate existing Dimension tables and load new data from the OLTP source.
   - **Data Flow Task**: Extracts data from the OLTP source, applies necessary transformations, and loads it into the Dimension tables in the Data Warehouse.

2. **Full Load for Fact Table**
   - **Job 2**: Triggers the SSIS package for the initial load of Fact tables.
   - **Control Flow**: Tasks to enable CDC, perform initial data extraction, and load data into Fact tables.
   - **Data Flow Task**: Extracts data from the OLTP source, performs key lookups, applies transformations, and loads data into the Fact tables.

### Daily (Incremental Load)

**Incremental Load for Fact Table**
   - **Job**: Triggers the SSIS package for the incremental load of Fact tables.
   - **Control Flow**: Tasks to capture changes using CDC, apply transformations, and load only the changed data into Fact tables.
   - **Data Flow Task**: Extracts changed data from the OLTP source, performs key lookups, applies transformations, and loads data into the Fact tables.

### CDC Method

1. **Start from Last LSN**
   - Use CDC Control in Control Flow to read the last LSN (Log Sequence Number) and load the range from the last LSN to the current LSN into the CDC State table.

2. **Identify Changes**
   - In Data Flow Task 1, activate CDC service, read the CDC table in the ETL Setting database, and store changes in the Change Table using `Union All`.

3. **Clear Changes from Fact**
   - In Control Flow, remove changes from the Fact table. In Data Flow Task 2, perform an initial load of changed records from OLTP into the Data Warehouse using a join, with a condition `Where ID IN (SELECT ID FROM V_Changes)`.

4. **Mark LSN for Next Incremental Load**
   - Store the transferred range in the CDC table in the ETL Setting database.

5. **Update Dimension Tables**
   - Update Dimension tables in the Data Warehouse first, followed by Fact tables. Perform lookups in Dimension tables to get surrogate keys (SK) and use them as foreign keys in Fact tables.

**Note**: If the organization operates 24/7 and continuously generates transactional data, use a snapshot only for the initial load to ensure static data capture at that moment.

### Outcome

The ETL process ensures that the Data Warehouse is consistently updated with the latest data from the OLTP source. Dimension tables are loaded first to ensure that Fact tables can reference the correct surrogate keys. The incremental load process minimizes downtime and ensures that only the latest changes are applied, maintaining the integrity and performance of the Data Warehouse. This setup supports efficient data analysis and reporting, providing accurate and up-to-date insights for business decision-making.

## Monitoring and Error Handling

- **Logging**: Implement logging to capture detailed information about the ETL process.
- **Error Handling**: Use event handlers in SSIS to manage errors and send notifications.
- **Performance Monitoring**: Regularly monitor the performance of ETL jobs and optimize as needed.
- **Alerts**: Set up alerts for job failures or performance issues.

## Additional Sections

- **Documentation**: Maintain comprehensive documentation for all ETL processes and components.
- **Version Control**: Use GitHub for version control of all scripts and packages.
- **Security**: Ensure data security and compliance with relevant regulations.

---

Feel free to adjust any sections as needed! If you have any specific requirements or additional sections you'd like to include, let me know.





