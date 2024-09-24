## Activate CDC for Orders and Order Details


use Northwind
GO
EXEC sys.sp_cdc_enable_db  --must be activated by DBA
GO
EXEC sys.sp_cdc_enable_table  @source_schema = N'dbo',@source_name = N'Orders',@role_name = NULL , @Supports_Net_Changes = 1
EXEC sys.sp_cdc_enable_table  @source_schema = N'dbo',@source_name = N'Order Details',@role_name = NULL , @Supports_Net_Changes = 1
GO
