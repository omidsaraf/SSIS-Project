Create a SQL Server Agent job to automate the generation and emailing of the job performance report.

#### 1. **Create the JobLogs Table**
```sql

CREATE Database Logs
Go
Use Database Logs
Go
CREATE TABLE JobLogs (
    JobID INT PRIMARY KEY,
    JobName NVARCHAR(100),
    JobType NVARCHAR(50),
    Status NVARCHAR(20),
    StartTime DATETIME,
    EndTime DATETIME,
    DurationMinutes INT,
    ResourceUsage NVARCHAR(100),
    ErrorDetails NVARCHAR(MAX)
);
```

#### 2. **Create new Job**

#### 3. **Configure the Job Properties**
1. **General Tab**:
   - **Name**: "Job Performance Report".
   - **Description**: "Generates and emails the weekly job performance report".

2. **Steps Tab**:
   - Click **New** to create a new job step.
   - **Step Name**: "Generate and Email Report".
   - **Type**: Select **Transact-SQL script (T-SQL)**.
   - **Database**: Select the database where the `JobLogs` table is located.
   - **Command**: Enter the following T-SQL script to generate the report and send it via email:
   ```sql
   -- Insert job performance data into JobLogs table
   INSERT INTO JobLogs (JobID, JobName, JobType, Status, StartTime, EndTime, DurationMinutes, ResourceUsage, ErrorDetails)
   SELECT 
       j.job_id AS JobID,
       j.name AS JobName,
       'ETL' AS JobType,
       CASE 
           WHEN h.run_status = 1 THEN 'Success'
           ELSE 'Failure'
       END AS Status,
       DATEADD(SECOND, h.run_time / 10000 * 3600 + (h.run_time / 100 % 100) * 60 + (h.run_time % 100), CONVERT(DATETIME, RTRIM(h.run_date))) AS StartTime,
       DATEADD(SECOND, h.run_duration / 10000 * 3600 + (h.run_duration / 100 % 100) * 60 + (h.run_duration % 100), DATEADD(SECOND, h.run_time / 10000 * 3600 + (h.run_time / 100 % 100) * 60 + (h.run_time % 100), CONVERT(DATETIME, RTRIM(h.run_date)))) AS EndTime,
       DATEDIFF(minute, DATEADD(SECOND, h.run_time / 10000 * 3600 + (h.run_time / 100 % 100) * 60 + (h.run_time % 100), CONVERT(DATETIME, RTRIM(h.run_date))), DATEADD(SECOND, h.run_duration / 10000 * 3600 + (h.run_duration / 100 % 100) * 60 + (h.run_duration % 100), DATEADD(SECOND, h.run_time / 10000 * 3600 + (h.run_time / 100 % 100) * 60 + (h.run_time % 100), CONVERT(DATETIME, RTRIM(h.run_date))))) AS DurationMinutes,
       'N/A' AS ResourceUsage,
       'N/A' AS ErrorDetails
   FROM 
       msdb.dbo.sysjobs j
       JOIN msdb.dbo.sysjobhistory h ON j.job_id = h.job_id
   WHERE 
       h.run_date >= CONVERT(VARCHAR(8), DATEADD(day, -7, GETDATE()), 112);

   -- Send the report via email
   EXEC msdb.dbo.sp_send_dbmail
       @profile_name = 'YourMailProfile',
       @recipients = 'admin@example.com',
       @subject = 'Weekly Job Performance Report',
       @body = 'Please find the attached job performance report for the last week.',
       @query = 'SELECT JobName, JobType, Status, StartTime, EndTime, DATEDIFF(minute, StartTime, EndTime) AS DurationMinutes, ResourceUsage, ErrorDetails FROM JobLogs WHERE StartTime >= DATEADD(day, -7, GETDATE()) ORDER BY StartTime DESC;',
       @attach_query_result_as_file = 1,
       @query_attachment_filename = 'JobPerformanceReport.csv',
       @query_result_separator = ',',
       @query_result_no_padding = 1,
       @query_result_header = 1;
   ```

3. **Schedules Tab**:
   - Click **New** to create a new schedule.
   - **Name**: Enter a name for the schedule, e.g., "Weekly Schedule".
   - **Schedule Type**: Select **Recurring**.
   - **Frequency**: Set the frequency to **Weekly** and choose the day and time you want the report to be generated and sent.

4. **Notifications Tab**:
   - Configure notifications to alert you if the job fails.

5. **Save the Job**:
   - Click **OK** to save the job.
