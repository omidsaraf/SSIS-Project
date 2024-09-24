## Create a SQL Server Agent job for performance monitoring and resource utilization monitoring:

1. **Configure Database Mail**:
   - Open SQL Server Management Studio (SSMS).
   - Expand the server node, right-click on `Database Mail`, and select `Configure Database Mail`.
   - Follow the wizard to set up a new Database Mail profile and account.

2. **Create an Operator**:
   - In SSMS, expand the `SQL Server Agent` node.
   - Right-click on `Operators` and select `New Operator`.
   - Enter the operator's name and email address. This operator will receive the email notifications.

3. **Create a Performance Monitoring Job**:
   - Right-click on `Jobs` under the `SQL Server Agent` node and select `New Job`.
   - In the `New Job` window, provide a name for the job (e.g., `Performance Monitoring Job`).

4. **Add Job Steps**:
   - Go to the `Steps` page and click `New`.
   - Create steps for performance monitoring and resource utilization monitoring.

### Example SQL Scripts and PowerShell Commands

#### Step 1: Performance Monitoring

```sql
-- Step 1: Monitor ETL Job Performance
USE msdb;
GO
SELECT job_id, name, enabled, description, 
       date_created, date_modified, 
       last_run_date, last_run_time, 
       last_run_duration, current_execution_status
FROM sysjobs
WHERE name LIKE '%ETL%';
```

#### Step 2: Resource Utilization Monitoring

```powershell
# Step 2: Monitor CPU, Memory, and Disk Usage
$cpu = Get-Counter '\Processor(_Total)\% Processor Time'
$memory = Get-Counter '\Memory\Available MBytes'
$disk = Get-Counter '\LogicalDisk(_Total)\% Free Space'

# Log the results
$log = "CPU: $($cpu.CounterSamples[0].CookedValue)%, Memory: $($memory.CounterSamples[0].CookedValue)MB, Disk: $($disk.CounterSamples[0].CookedValue)% Free"
Add-Content -Path "C:\PerformanceLogs\PerformanceLog.txt" -Value $log

# Send an email notification if thresholds are exceeded
if ($cpu.CounterSamples[0].CookedValue -gt 80 -or $memory.CounterSamples[0].CookedValue -lt 1024 -or $disk.CounterSamples[0].CookedValue -lt 20) {
    Send-MailMessage -To 'admin@example.com' -From 'sqlserver@example.com' -Subject 'Performance Alert' -Body $log -SmtpServer 'smtp.example.com'
}
```

5. **Configure the Job Steps**:
   - For each step, set the `Type` to `Transact-SQL script (T-SQL)` for SQL scripts and `Operating system (CmdExec)` for PowerShell commands.
   - In the `Command` box, enter the corresponding script or command for each monitoring task.

6. **Set Job Step Actions**:
   - For each step, go to the `Advanced` tab.
   - Set `On success action` to `Go to the next step`.
   - Set `On failure action` to `Quit the job reporting failure`.

7. **Schedule the Job**:
   - Go to the `Schedules` page and click `New`.
   - Set up the schedule to run the job at regular intervals (e.g., every 15 minutes).

8. **Enable Notifications**:
   - Go to the `Notifications` page.
   - Check the `E-mail` option and select the operator you created earlier.
   - Set the condition to `When the job fails`.

