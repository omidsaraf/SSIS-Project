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

