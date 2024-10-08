# ETL Project Using Northwind Dataset
![Project Architecture](https://github.com/user-attachments/assets/2d410670-d77a-4ef6-8164-e553a047b3f8)

## Requirements

- **SQL Server Management Studio (SSMS)**: Ensure SSIS is installed.
- **SQL Server Engine**: Set up for OLTP, Staging, and Data Warehouse environments.
- **Visual Studio**: Used for creating and managing ETL packages.
- **SQL Server Agent**: For scheduling and automating ETL jobs.

## Project Overview

This ETL project utilizes the Northwind Dataset as the data source. The project involves several key steps to ensure data is efficiently extracted, transformed, and loaded into a Data Warehouse.

### 1. Creating the Data Warehouse in SSMS
- **Setup**: Establish the Data Warehouse environment in SSMS, ensuring it is optimized for performance and scalability.

### 2. Creating Dimension and Fact Tables
- **T-SQL Scripts**: Use T-SQL scripts in SSMS to define the structure of Dimension and Fact tables, ensuring they are designed to support efficient querying and reporting.

### 3. Creating the Staging Database in SSMS
- **Staging Tables**: The ETL Setting database includes staging tables to facilitate the loading and transformation of data.
- **Detect Changes**: If CDC (Change Data Capture) is not available in OLTP, create tables to detect changes.
- **Views with Union All**: Create views using `Union All` to record changes.
- **Incremental Load Connections**: Establish connections for incremental loading to avoid truncating and reloading tables constantly. Only apply recent changes to Fact tables.
- **Stored Procedures**: Include the following Stored Procedures (SPs):
  1. **Hash Value SP**: Used if CDC service is not possible to detect changes.
  2. **Key Lookup SCD2 SP**: For handling Slowly Changing Dimensions Type 2.
  3. **Inferred Record SP**: For handling inferred records during the ETL process.
- **Data Validation**: Implement data validation checks in the staging area to ensure data quality before loading into the Data Warehouse.
- **Temporary Storage**: Use staging tables for temporary storage of data during the ETL process to manage data transformations and cleansing.

### 4. Designing the Dimension Loading Process in Visual Studio
- **Control Flow**: Design the Control Flow to include tasks such as Truncate Task and Data Flow Task.
- **Data Flow Task**: Configure the Data Flow Task to handle different Attribute Types, including data extraction from OLTP, transformation, and loading into the destination table.
- **Key Lookup**: Implement Stored Procedures for key lookups in all Dimension tables to ensure data integrity and consistency.

### 5. Designing the Fact Loading Process in Visual Studio
- **Enable CDC**: Enable Change Data Capture (CDC) for relevant OLTP tables to track changes.
- **Initial Load**: Design the Control Flow to include CDC Tasks and Data Flow Task for the initial load, using key lookups and Stored Procedures.
- **Incremental Load**: Design the Control Flow for incremental loads, including CDC Tasks, Data Flow Task, and handling changes with key lookups and Stored Procedures.

### 6. Deployment (SSIS Catalog)
- **SSIS Scale Out**: During SQL Server and SSMS installation, enable the SSIS Scale Out option to allow parallel execution of large SSIS packages.
- **Create SSIS DB**: Set up the SSIS database to manage and store SSIS packages.
- **Import Packages**: Import ETL packages from Visual Studio into SSMS.
- **Test Deployment**: Conduct thorough testing to ensure the project is correctly deployed and functioning as expected.

### 7. Job Automation

#### Enable SQL Server Agent Job
- **Activate SQL Server Agent**: Ensure SQL Server Agent is enabled to manage job scheduling and execution.
#### Create Jobs
1. **Initial Load Jobs**:
   - **Initial Load for Dimension**: Create a job to perform the initial load of Dimension tables. This job should run once to populate the Dimension tables with historical data.
   - **Initial Load for Fact**: Create a job to perform the initial load of Fact tables. This job should run once to load historical data into the Fact tables.
2. **Incremental Load Jobs**:
   - **Incremental Load for Fact**: Create a job to perform the incremental load of Fact tables. This job should run at regular intervals (e.g., daily, hourly) to load new or updated data into the Fact tables.
3. **Performance Monitoring Job**:
   - **Performance Monitoring**: Create a job to regularly monitor the performance of ETL jobs using tools like SQL Server Profiler, Performance Monitor, and Azure Monitor. This job should track performance metrics and identify any bottlenecks or inefficiencies.
   - **Resource Utilization Monitoring**: Create a job to monitor CPU, memory, and disk usage. This job should ensure that ETL processes are not overloading the system and adjust resource allocation as needed.
5. **Job Execution Monitoring**:
   - **Job Execution Tracking**: Create a job to track the execution time of ETL jobs. This job should use SQL Server Agent job history and SSIS logging to review job performance and identify any delays or bottlenecks.
6. **Automatic Email Notifications for Job Failures**:
   - **Email Notifications**: Create a job to automatically send email notifications in case of job failures. Configure Database Mail in SQL Server to send alerts and customize email templates to include relevant job details and error messages.
7. **Maintenance Plan Job**:
   - **Index Maintenance**: Create a job to rebuild or reorganize indexes regularly to maintain query performance.
   - **Statistics Update**: Create a job to update statistics regularly to ensure query optimization.
   - **Database Integrity Checks**: Create a job to perform regular integrity checks on databases to detect and repair corruption.
   - **Cleanup Jobs**: Create a job to clean up old data, logs, and temporary files to free up space and improve performance.
8. **Backup and Recovery Jobs**:
   - **Regular Backups**: Create a job to back up the Data Warehouse and ETL configurations regularly.
   - **Backup Verification**: Create a job to verify the integrity of backups to ensure they can be restored successfully.
   - **Disaster Recovery Drills**: Create a job to perform regular disaster recovery drills to test the effectiveness of recovery procedures.
9. **Security Jobs**:
   - **Security Audit Job**: Create a job to perform regular security audits. This job should check for compliance with security policies, review audit logs, verify access controls, and ensure encryption is properly configured.
   - **Security Alerts**: Create a job to send email notifications for specific security events, such as unauthorized access attempts or changes to security settings.
   - **Patch Management**: Create a job to apply security patches and updates to SQL Server and related components regularly.
#### Schedule Jobs
- **Frequency Planning**: Determine the optimal frequency for each job based on data update requirements and system performance considerations.
- **Job Scheduling**: Use SQL Server Agent to schedule jobs, ensuring they run at the specified times without manual intervention.

By implementing these job automation tasks, you can ensure that your ETL processes and data warehouse operations are efficient, reliable, and secure. This comprehensive approach helps maintain optimal performance, minimize resource contention, and ensure timely data updates while safeguarding sensitive data.

## ETL Process

### Weekly and Initial Load

1. **Full Load for Dimension Tables**
   - **Job 1**: Triggers the SSIS package for the initial load of Dimension tables.
   - **Control Flow**: Includes tasks to truncate existing Dimension tables and load new data from the OLTP source.
   - **Data Flow Task**: Extracts data from the OLTP source, applies necessary transformations, and loads it into the Dimension tables in the Data Warehouse.
![01- Full Load Dimension](https://github.com/user-attachments/assets/10310c3c-9fb5-4bd6-ace3-717cc0013128)


2. **Full Load for Fact Table**
   - **Job 2**: Triggers the SSIS package for the initial load of Fact tables.
   - **Control Flow**: Includes tasks to enable CDC, perform initial data extraction, and load data into Fact tables.
   - **Data Flow Task**: Extracts data from the OLTP source, performs key lookups, applies transformations, and loads data into the Fact tables.
  
  ![02- Initial Load Fact](https://github.com/user-attachments/assets/30be4575-ae8c-4af2-9dc2-bf7ad5a87976)


### Daily (Incremental Load)

**Incremental Load for Fact Table**
   - **Job**: Triggers the SSIS package for the incremental load of Fact tables.
   - **Control Flow**: Includes tasks to capture changes using CDC, apply transformations, and load only the changed data into Fact tables.
   - **Data Flow Task**: Extracts changed data from the OLTP source, performs key lookups, applies transformations, and loads data into the Fact tables.
![03- Incremental Load Fact](https://github.com/user-attachments/assets/a93e8546-70e3-46a6-a8eb-87a113503d8b)


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

### Error Handling

- **Event Handlers**: Implement event handlers in SSIS to manage errors. Configure OnError, OnWarning, and OnTaskFailed event handlers to capture and respond to issues.
- **Logging**: Set up comprehensive logging to capture detailed information about the ETL process. Use SSIS log providers to log events to SQL Server, text files, or Windows Event Log.
- **Notifications**: Configure email notifications to alert administrators of job failures or critical errors. Use Database Mail in SQL Server to send alerts.
- **Retry Logic**: Implement retry logic for transient errors. Configure SSIS packages to retry tasks a specified number of times before failing.

### Outcome
#### Sales Analysis Dashboard - Power BI
![Sales Analysis Dashboard](https://github.com/user-attachments/assets/f0755064-fa39-45a7-85ae-63807ea8713e)

The ETL process ensures that the Data Warehouse is consistently updated with the latest data from the OLTP source. Dimension tables are loaded first to ensure that Fact tables can reference the correct surrogate keys. The incremental load process minimizes downtime and ensures that only the latest changes are applied, maintaining the integrity and performance of the Data Warehouse. This setup supports efficient data analysis and reporting, providing accurate and up-to-date insights for business decision-making.

Data Model
![PowerBI_Data Model](https://github.com/user-attachments/assets/1bb9c052-7610-4c02-8b75-4fe408b7922b)


### Additional Considerations:
1. **Load Balancing**: Implement load balancing techniques to distribute ETL workloads evenly across available resources. This helps in maximizing resource utilization and improving overall performance.
2. **Capacity Planning**: Regularly review and plan for future capacity needs based on data growth trends. This ensures that the ETL infrastructure can handle increasing data volumes without performance degradation.
3. **Backup and Recovery**: 
   - **Regular Backups**: Schedule regular backups of the Data Warehouse and ETL configurations to prevent data loss. Ensure that backups are stored securely and can be restored quickly in case of failure.
   - **Disaster Recovery Plan**: Develop and test a disaster recovery plan to ensure business continuity in case of major system failures. This should include procedures for data restoration, system recovery, and communication with stakeholders.
   - **Redundancy**: Implement redundancy measures to minimize the impact of hardware failures. This can include using RAID configurations, failover clusters, and geographically distributed data centers.
   - **Testing**: Regularly test backup and recovery procedures to ensure they work as expected. This helps in identifying and addressing any issues before they impact production systems.
4. **Documentation**: Maintain comprehensive documentation for all ETL processes and components. Include details on data sources, transformation logic, data flow, and job schedules.
5. **Version Control**: Use GitHub or another version control system to manage all scripts and packages. Ensure that changes are tracked and can be rolled back if necessary.
6. **Security**: Ensure data security and compliance with relevant regulations. Implement encryption for sensitive data, use role-based access control, and regularly audit access logs.
7. **Testing**: Conduct thorough testing of ETL processes, including unit tests, integration tests, and performance tests. Use test data to validate the accuracy and efficiency of ETL jobs.
8. **Optimization**: Regularly review and optimize ETL processes to improve performance. Identify and eliminate bottlenecks, optimize SQL queries, and ensure efficient use of resources.
9. **Scalability**: Design ETL processes to be scalable. Use SSIS Scale Out to distribute workloads across multiple servers and ensure that the system can handle increasing data volumes.



