
### Step-by-Step Guide to Create a Job for Sending Email on Job Failure

1. **Configure Database Mail**:
   - Open SQL Server Management Studio (SSMS).
   - Expand the server node, right-click on `Database Mail`, and select `Configure Database Mail`.
   - Follow the wizard to set up a new Database Mail profile and account.

2. **Create an Operator**:
   - In SSMS, expand the `SQL Server Agent` node.
   - Right-click on `Operators` and select `New Operator`.
   - Enter the operator's name and email address. This operator will receive the email notifications.

3. **Set Up the Job**:
   - Right-click on `Jobs` under the `SQL Server Agent` node and select `New Job`.
   - In the `New Job` window, provide a name for the job (e.g., `Job Failure Notification`).

4. **Add Job Steps**:
   - Go to the `Steps` page and click `New`.
   - Create a step that performs the main task of your job (e.g., running an SSIS package or a T-SQL script).
   - Add another step specifically for sending the email notification. This step will be executed only if the main task fails.

5. **Configure the Email Notification Step**:
   - In the `New Job Step` window, set the `Type` to `Operating system (CmdExec)`.
   - In the `Command` box, enter the following PowerShell script to send an email:
     ```powershell
     Powershell -command "& {Send-MailMessage -To 'admin@example.com' -From 'sqlserver@example.com' -Subject 'Job Failure: $(ESCAPE_SQUOTE(JOBNAME))' -Body 'The SQL Server Agent job $(ESCAPE_SQUOTE(JOBNAME)) has failed.' -SmtpServer 'smtp.example.com'}"
     ```
   - Replace the email addresses and SMTP server with your actual values.

6. **Set Job Step Actions**:
   - For the main task step, go to the `Advanced` tab.
   - Set `On success action` to `Go to the next step`.
   - Set `On failure action` to `Go to step: [Email Notification Step]`.
   - For the email notification step, set `On success action` to `Quit the job reporting failure` and `On failure action` to `Quit the job reporting failure`.

7. **Schedule the Job**:
   - Go to the `Schedules` page and click `New`.
   - Set up the schedule according to your requirements (e.g., daily, hourly).

8. **Enable Notifications**:
   - Go to the `Notifications` page.
   - Check the `E-mail` option and select the operator you created earlier.
   - Set the condition to `When the job fails`.

###Job Configuration

a simplified example of the PowerShell command used in the email notification step:

```powershell
Powershell -command "& {Send-MailMessage -To 'admin@example.com' -From 'sqlserver@example.com' -Subject 'Job Failure: $(ESCAPE_SQUOTE(JOBNAME))' -Body 'The SQL Server Agent job $(ESCAPE_SQUOTE(JOBNAME)) has failed.' -SmtpServer 'smtp.example.com'}"
```
