Create a SQL Server Agent job for backing up the `Northwind_DW_Backup` database using full backups, differential backups, and transaction log backups. This setup ensures that your database is regularly backed up, minimizing the risk of data loss.

**If the organization's policy is Full backup:**
After the ETL process completes, for example at 3 AM, take a backup. Perform Full Backups from Friday to Friday, and Differential Backups from Saturday to Thursday.

**If not:**
Only take a backup of Dimensions with Historical Attributes at the end of each ETL (this occupies less space). Since SQL Server does not have a command to backup a table, create another database and define a Job for it to:
- Drop existing tables in that database.
- Insert tables into that database using `SELECT INTO`.
- Take a backup after that.


### Step-by-Step Guide to Create a Backup Job

1. **Configure Database Mail**:
   - Open SQL Server Management Studio (SSMS).
   - Expand the server node, right-click on `Database Mail`, and select `Configure Database Mail`.
   - Follow the wizard to set up a new Database Mail profile and account.

2. **Create an Operator**:
   - In SSMS, expand the `SQL Server Agent` node.
   - Right-click on `Operators` and select `New Operator`.
   - Enter the operator's name and email address. This operator will receive the email notifications.

3. **Create a Backup Job**:
   - Right-click on `Jobs` under the `SQL Server Agent` node and select `New Job`.
   - In the `New Job` window, provide a name for the job (e.g., `Northwind_DW_Backup Job`).

4. **Add Job Steps**:
   - Go to the `Steps` page and click `New`.
   - Create steps for full backups, differential backups, and transaction log backups.

### SQL Scripts for Each Backup Type

#### Step 1: Full Backup (Weekly)

```sql
-- Full Backup
BACKUP DATABASE [Northwind_DW_Backup]
TO DISK = N'C:\SQLBackups\Northwind_DW_Backup_Full.bak'
WITH FORMAT,
     MEDIANAME = 'SQLServerBackups',
     NAME = 'Full Backup of Northwind_DW_Backup';
```

#### Step 2: Differential Backup (Daily)

```sql
-- Differential Backup
BACKUP DATABASE [Northwind_DW_Backup]
TO DISK = N'C:\SQLBackups\Northwind_DW_Backup_Diff.bak'
WITH DIFFERENTIAL,
     MEDIANAME = 'SQLServerBackups',
     NAME = 'Differential Backup of Northwind_DW_Backup';
```

#### Step 3: Transaction Log Backup (Every 15 Minutes)

```sql
-- Transaction Log Backup
BACKUP LOG [Northwind_DW_Backup]
TO DISK = N'C:\SQLBackups\Northwind_DW_Backup_Log.trn'
WITH NOFORMAT,
     NOINIT,
     NAME = 'Transaction Log Backup of Northwind_DW_Backup';
```

5. **Configure the Job Steps**:
   - For each step, set the `Type` to `Transact-SQL script (T-SQL)`.
   - In the `Command` box, enter the corresponding SQL script for each backup type.

6. **Set Job Step Actions**:
   - For each step, go to the `Advanced` tab.
   - Set `On success action` to `Go to the next step` (if applicable).
   - Set `On failure action` to `Quit the job reporting failure`.

7. **Schedule the Job**:
   - Go to the `Schedules` page and click `New`.
   - Create three schedules:
     - **Full Backup**: Schedule to run weekly (e.g., every Sunday at 2:00 AM).
     - **Differential Backup**: Schedule to run daily (e.g., every day at 2:00 AM, except Sunday).
     - **Transaction Log Backup**: Schedule to run every 15 minutes.

8. **Enable Notifications**:
   - Go to the `Notifications` page.
   - Check the `E-mail` option and select the operator you created earlier.
   - Set the condition to `When the job fails`.
