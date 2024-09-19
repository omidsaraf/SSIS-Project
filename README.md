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