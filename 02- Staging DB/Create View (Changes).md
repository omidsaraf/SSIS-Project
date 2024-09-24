
```sql
USE [ETL Settings]
GO
CREATE OR ALTER VIEW V_ChangedOrders
AS
  SELECT OrderID
  FROM OrdersChanges
  UNION ALL
  SELECT OrderID
  FROM OrderDetailsChanges

GO
