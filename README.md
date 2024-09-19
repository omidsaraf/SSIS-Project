# SSIS Project
End to end project

In This ETL project, the goal is to first create Dimension tables and then Fact tables. The steps involved in this process are:

1. **Creating the Data Warehouse in SSMS**
2. **Creating Dimension and Fact tables (structure only, no data) using T-SQL in SSMS**
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
5. **Job (Execute Packages)**
   - Three Packages has been created incluing inital load for Dimention, initial load for Fact, Incremental load for fact
   - Enable SQL Server Agent Job
   - Create Jobs to execute the SSIS packages.
   - Schedule the Job according to your desired frequency and activate it.
