# SSIS Project
End-to-End ETL Process

## Development:
In this ETL project, the Northwind Dataset is used as the data source. The following steps are involved in the process:

#### 1. Creating the Data Warehouse in SSMS
- Set up the Data Warehouse structure in SQL Server Management Studio (SSMS).

#### 2. Creating Dimension and Fact Tables
- Use T-SQL in SSMS to create the structure of Dimension and Fact tables without data.
- Create and load Fact tables as a joined table in the Data Warehouse, suitable for SSAS Multi-Dimensional.

#### 3. Deployment (SSIS Catalog)
- **SSIS Scale Out**: During SQL Server and SSMS installation, enable the SSIS Scale Out option in the Shared Features section to run large SSIS Packages in parallel across multiple servers.
- **Create SSIS DB**: Set up the SSIS database.
- **Import Packages**: Import packages from Visual Studio into SSMS.
- **Test Deployment**: Test the project deployment.

#### 4. Job (Automation)
- **Enable SQL Server Agent Job**: Activate the SQL Server Agent Job.
- **Create Jobs**: Create jobs to execute the SSIS packages. Three packages are created: initial load for Dimension, initial load for Fact, and incremental load for Fact.
- **Schedule Jobs**: Schedule the jobs according to your desired frequency and activate them.

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

6. **Incremental Load Using CDC**:
   - **Fact Table Load**: Use two SSIS Packages for initial and incremental loads. Initial load is done once, while incremental loads are ongoing.
   - **Change Detection**: Use CDC service or Compare Hash Value for detecting changes in OLTP tables.
   - **Initial Load for 24/7 Business**: Use Snapshot for initial load if the business operates 24/7. Otherwise, use CDC Control 1 to mark the start and CDC Control 2 to mark the end of the initial load.
   - **Snapshot**: Used only for the initial load to ensure static data capture at that moment.



### Detailed ETL Process for Dimension and Fact Tables



### Outcome
The ETL process ensures that the Data Warehouse is consistently updated with the latest data from the OLTP source. Dimension tables are loaded first to ensure that Fact tables can reference the correct surrogate keys. The incremental load process minimizes downtime and ensures that only the latest changes are applied, maintaining the integrity and performance of the Data Warehouse. This setup supports efficient data analysis and reporting, providing accurate and up-to-date insights for business decision-making.
