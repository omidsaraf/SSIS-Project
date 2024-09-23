### End-to-End ETL Project Using Northwind Dataset

In this ETL project, the Northwind Dataset is used as the data source. The following steps are involved in the process:

#### 1. Creating the Data Warehouse in SSMS
- Set up the Data Warehouse structure in SQL Server Management Studio (SSMS).

#### 2. Creating Dimension and Fact Tables
- Use T-SQL in SSMS to create the structure of Dimension and Fact tables without data.

#### 3. Designing the Dimension Loading Process in Visual Studio
- **Control Flow**: Design includes Truncate Task and Data Flow Task.
- **Data Flow Task**: Design based on attribute types, including source data (OLTP), Slowly Changing Dimensions (SCD), and destination table.
- **Key Lookup**: Use stored procedures (SP) for key lookups in all Dimension tables.

#### 4. Designing the Fact Loading Process in Visual Studio
- **Enable CDC**: Activate Change Data Capture (CDC) for relevant tables in OLTP.
- **Initial Load**: Design Control Flow with two CDC Tasks and one Data Flow Task (using key lookup and SP).
- **Incremental Load**: Design Control Flow with two CDC Tasks and one Data Flow Task to capture changes, truncate, and then load changes using a join in the source data. Use Key Lookup component for changing columns, SP for historical fields, and handle inferred members.

#### 5. Deployment (SSIS Catalog)
- **SSIS Scale Out**: During SQL Server and SSMS installation, enable the SSIS Scale Out option in the Shared Features section to run large SSIS Packages in parallel across multiple servers.
- **Create SSIS DB**: Set up the SSIS database.
- **Import Packages**: Import packages from Visual Studio into SSMS.
- **Test Deployment**: Test the project deployment.

#### 6. Job (Automation)
- **Enable SQL Server Agent Job**: Activate the SQL Server Agent Job.
- **Create Jobs**: Create jobs to execute the SSIS packages. Three packages are created: initial load for Dimension, initial load for Fact, and incremental load for Fact.
- **Schedule Jobs**: Schedule the jobs according to your desired frequency and activate them.

### Detailed ETL Process for Dimension and Fact Tables

1. **Create Data Warehouse in SSMS**:
   - Set up the Data Warehouse structure in SQL Server Management Studio (SSMS).

2. **Create Dimension and Fact Tables**:
   - Use T-SQL in SSMS to create the structure of Dimension and Fact tables without data.

3. **Design Dimension Load Process in Visual Studio**:
   - **Control Flow**: Design includes Truncate Task and Data Flow Task.
   - **Data Flow Task**: Design based on attribute types, including source data (OLTP), Slowly Changing Dimensions (SCD), and destination table.
   - **Key Lookup**: Use stored procedures (SP) for key lookups in all Dimension tables.

4. **Design Fact Load Process in Visual Studio**:
   - **Enable CDC**: Activate Change Data Capture (CDC) for relevant tables in OLTP.
   - **Initial Load**: Design Control Flow with two CDC Tasks and one Data Flow Task (using key lookup and SP).
   - **Incremental Load**: Design Control Flow with two CDC Tasks and one Data Flow Task to capture changes, truncate, and then load changes using a join in the source data. Use Key Lookup component for changing columns, SP for historical fields, and handle inferred members.

5. **Build and Load Fact Tables**:
   - Create and load Fact tables as a joined table in the Data Warehouse, suitable for SSAS Multi-Dimensional.

6. **Start from Last LSN**:
   - Use CDC Control in Control Flow to read the last LSN (Log Sequence Number) and load the range from the last LSN to the current LSN into the CDC State table.

7. **Identify Changes**:
   - In Data Flow Task 1, activate CDC service, read the CDC table in the ETL Setting database, and store changes in the Change Table using `Union All`.

8. **Clear Changes from Fact**:
   - In Control Flow, remove changes from the Fact table. In Data Flow Task 2, perform an initial load of changed records from OLTP into the Data Warehouse using a join, with a condition `Where ID IN (SELECT ID FROM V_Changes)`.

9. **Mark LSN for Next Incremental Load**:
   - Store the transferred range in the CDC table in the ETL Setting database.

10. **Update Dimension Tables**:
    - Update Dimension tables in the Data Warehouse first, followed by Fact tables. Perform lookups in Dimension tables to get surrogate keys (SK) and use them as foreign keys in Fact tables.

11. **Incremental Load Using CDC**:
    - **Fact Table Load**: Use two SSIS Packages for initial and incremental loads. Initial load is done once, while incremental loads are ongoing.
    - **Change Detection**: Use CDC service or Compare Hash Value for detecting changes in OLTP tables.
    - **Initial Load for 24/7 Business**: Use Snapshot for initial load if the business operates 24/7. Otherwise, use CDC Control 1 to mark the start and CDC Control 2 to mark the end of the initial load.
    - **Snapshot**: Used only for the initial load to ensure static data capture at that moment.

This comprehensive approach ensures efficient and accurate ETL operations for loading Dimension and Fact tables in a Data Warehouse environment. If you need further details or have any questions, feel free to ask!

Source: Conversation with Copilot, 23/09/2024
(1) 4 HOURS ]] SSIS Complete Tutorial - { End to End } Full Course - SQL .... https://www.youtube.com/watch?v=G_wG-bzTCZY.
(2) end to end msbi project in real time||part1|ssis project#ssrs#ssis#ssas. https://www.youtube.com/watch?v=sxClwmWigc4.
(3) [[ 3 HOURS ]] Data Warehouse Complete Tutorial - SQL + SSIS + SSAS + Power BI - { End to End }. https://www.youtube.com/watch?v=eNxbMwUGl1g.
(4) Integration Services (SSIS) Projects and Solutions. https://learn.microsoft.com/en-us/sql/integration-services/integration-services-ssis-projects-and-solutions?view=sql-server-ver16.
(5) GitHub - AshrafMohsen/End-to-EndDWSolution: The project involves .... https://github.com/AshrafMohsen/End-to-EndDWSolution.
(6) GitHub - omidsaraf/SSIS-Project: End to end project. https://github.com/omidsaraf/SSIS-Project/.
(7) integration-services-ssis-projects-and-solutions.md - GitHub. https://github.com/MicrosoftDocs/sql-docs/blob/live/docs/integration-services/integration-services-ssis-projects-and-solutions.md.
