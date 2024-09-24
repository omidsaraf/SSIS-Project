enabling Change Data Capture (CDC) on the `Orders` and `Order Details` tables in the `Northwind` database.

```sql
-- Activate CDC for Orders and Order Details

USE Northwind;
GO

-- Enable CDC at the database level (must be activated by DBA)
EXEC sys.sp_cdc_enable_db;
GO

-- Enable CDC for the Orders table
EXEC sys.sp_cdc_enable_table 
    @source_schema = N'dbo',
    @source_name = N'Orders',
    @role_name = NULL,
    @supports_net_changes = 1;
GO

-- Enable CDC for the Order Details table
EXEC sys.sp_cdc_enable_table 
    @source_schema = N'dbo',
    @source_name = N'Order Details',
    @role_name = NULL,
    @supports_net_changes = 1;
GO
```
