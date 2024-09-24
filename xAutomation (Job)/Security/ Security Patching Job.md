# Security Patching Job

This script applies the latest security patches to SQL Server.

```powershell
# Step 1: Download the latest updates
$updates = Get-WindowsUpdate -MicrosoftUpdate -AcceptAll -Install

# Step 2: Apply the updates
Install-WindowsUpdate -MicrosoftUpdate -AcceptAll -Install

# Step 3: Restart the SQL Server service if necessary
Restart-Service -Name 'MSSQLSERVER' -Force

# Step 4: Send an email notification
Send-MailMessage -To 'admin@example.com' -From 'sqlserver@example.com' -Subject 'SQL Server Security Patching Completed' -Body 'The latest security patches and updates have been successfully applied to the SQL Server.' -SmtpServer 'smtp.example.com'
```
