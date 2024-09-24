-- Step 1: Review Audit Logs
EXEC msdb.dbo.sp_send_dbmail
    @profile_name = 'YourMailProfile',
    @recipients = 'admin@example.com',
    @subject = 'Security Audit Log Review',
    @body = 'Review the attached audit logs for any suspicious activities.',
    @query = 'SELECT * FROM sys.fn_get_audit_file(''C:\AuditLogs\*.sqlaudit'', DEFAULT, DEFAULT)';

-- Step 2: Access Control Check
EXEC msdb.dbo.sp_send_dbmail
    @profile_name = 'YourMailProfile',
    @recipients = 'admin@example.com',
    @subject = 'Access Control Check',
    @body = 'Verify that only authorized users have access to sensitive data.',
    @query = 'SELECT * FROM sys.database_principals WHERE type IN (''S'', ''U'', ''G'')';

-- Step 3: Encryption Verification
EXEC msdb.dbo.sp_send_dbmail
    @profile_name = 'YourMailProfile',
    @recipients = 'admin@example.com',
    @subject = 'Encryption Verification',
    @body = 'Ensure that encryption is properly configured for sensitive data.',
    @query = 'SELECT * FROM sys.dm_database_encryption_keys';
