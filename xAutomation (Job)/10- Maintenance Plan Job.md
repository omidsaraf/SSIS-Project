Create a single SQL Server Agent job that includes all the maintenance tasks: Index Maintenance, Statistics Update, Database Integrity Checks, and Cleanup Jobs.

### Step-by-Step Guide to Create a Comprehensive Maintenance Plan Job

1. **Configure Database Mail**:
   - Open SQL Server Management Studio (SSMS).
   - Expand the server node, right-click on `Database Mail`, and select `Configure Database Mail`.
   - Follow the wizard to set up a new Database Mail profile and account.

2. **Create an Operator**:
   - In SSMS, expand the `SQL Server Agent` node.
   - Right-click on `Operators` and select `New Operator`.
   - Enter the operator's name and email address. This operator will receive the email notifications.

3. **Create a Maintenance Plan Job**:
   - Right-click on `Jobs` under the `SQL Server Agent` node and select `New Job`.
   - In the `New Job` window, provide a name for the job (e.g., `Comprehensive Maintenance Plan Job`).

4. **Add Job Steps**:
   - Go to the `Steps` page and click `New`.
   - Create steps for each maintenance task.

### Example SQL Scripts for Each Maintenance Task

#### Step 1: Index Maintenance

```sql
-- Rebuild or Reorganize Indexes
USE [YourDatabaseName];
GO
DECLARE @TableName NVARCHAR(255);
DECLARE @SQL NVARCHAR(MAX);

DECLARE TableCursor CURSOR FOR
SELECT QUOTENAME(SCHEMA_NAME(t.schema_id)) + '.' + QUOTENAME(t.name)
FROM sys.tables AS t
WHERE t.is_ms_shipped = 0;

OPEN TableCursor;
FETCH NEXT FROM TableCursor INTO @TableName;

WHILE @@FETCH_STATUS = 0
BEGIN
    SET @SQL = 'ALTER INDEX ALL ON ' + @TableName + ' REBUILD';
    EXEC sp_executesql @SQL;
    FETCH NEXT FROM TableCursor INTO @TableName;
END;

CLOSE TableCursor;
DEALLOCATE TableCursor;
```

#### Step 2: Statistics Update

```sql
-- Update Statistics
USE [YourDatabaseName];
GO
EXEC sp_updatestats;
```

#### Step 3: Database Integrity Checks

```sql
-- Database Integrity Checks
USE [YourDatabaseName];
GO
DBCC CHECKDB;
```

#### Step 4: Cleanup Jobs

```sql
-- Cleanup Old Data, Logs, and Temporary Files
USE [YourDatabaseName];
GO
-- Example: Delete old records from a log table
DELETE FROM YourLogTable WHERE LogDate < DATEADD(MONTH, -3, GETDATE());

-- Example: Clean up old backup files
EXEC xp_delete_file 0, N'C:\SQLBackups\', N'bak', DATEADD(DAY, -30, GETDATE());
```

5. **Configure the Job Steps**:
   - For each step, set the `Type` to `Transact-SQL script (T-SQL)`.
   - In the `Command` box, enter the corresponding SQL script for each maintenance task.

6. **Set Job Step Actions**:
   - For each step, go to the `Advanced` tab.
   - Set `On success action` to `Go to the next step`.
   - Set `On failure action` to `Quit the job reporting failure`.

7. **Schedule the Job**:
   - Go to the `Schedules` page and click `New`.
   - Set up the schedule to run the job at regular intervals (e.g., weekly).

8. **Enable Notifications**:
   - Go to the `Notifications` page.
   - Check the `E-mail` option and select the operator you created earlier.
   - Set the condition to `When the job fails`.

### Example Job Configuration

Here’s an example of how the job steps might look in SQL Server Management Studio:

```sql
-- Step 1: Index Maintenance
USE [YourDatabaseName];
GO
DECLARE @TableName NVARCHAR(255);
DECLARE @SQL NVARCHAR(MAX);

DECLARE TableCursor CURSOR FOR
SELECT QUOTENAME(SCHEMA_NAME(t.schema_id)) + '.' + QUOTENAME(t.name)
FROM sys.tables AS t
WHERE t.is_ms_shipped = 0;

OPEN TableCursor;
FETCH NEXT FROM TableCursor INTO @TableName;

WHILE @@FETCH_STATUS = 0
BEGIN
    SET @SQL = 'ALTER INDEX ALL ON ' + @TableName + ' REBUILD';
    EXEC sp_executesql @SQL;
    FETCH NEXT FROM TableCursor INTO @TableName;
END;

CLOSE TableCursor;
DEALLOCATE TableCursor;

-- Step 2: Statistics Update
USE [YourDatabaseName];
GO
EXEC sp_updatestats;

-- Step 3: Database Integrity Checks
USE [YourDatabaseName];
GO
DBCC CHECKDB;

-- Step 4: Cleanup Jobs
USE [YourDatabaseName];
GO
DELETE FROM YourLogTable WHERE LogDate < DATEADD(MONTH, -3, GETDATE());
EXEC xp_delete_file 0, N'C:\SQLBackups\', N'bak', DATEADD(DAY, -30, GETDATE());
```
