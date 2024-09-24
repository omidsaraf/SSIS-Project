```sql
USE ETL_Setting
GO

-- Create the stored procedure
CREATE PROC usp_ETL_IncrementalLoad @DaysLastNight INT, @DaysTonight INT AS
BEGIN
    SET NOCOUNT ON;

    -- Drop and recreate LastNight table in staging database
    IF OBJECT_ID('ETL_Setting.dbo.LastNight', 'U') IS NOT NULL
        DROP TABLE ETL_Setting.dbo.LastNight;

    SELECT OrderID, CHECKSUM(*) AS HashValue
    INTO ETL_Setting.dbo.LastNight
    FROM Northwind.dbo.Orders
    WHERE OrderDate > (GETDATE() - @DaysLastNight);

    -- Drop and recreate Tonight table in staging database
    IF OBJECT_ID('ETL_Setting.dbo.Tonight', 'U') IS NOT NULL
        DROP TABLE ETL_Setting.dbo.Tonight;

    SELECT OrderID, CHECKSUM(*) AS HashValue
    INTO ETL_Setting.dbo.Tonight
    FROM Northwind.dbo.Orders
    WHERE OrderDate > (GETDATE() - @DaysTonight);

    -- Detect changes in staging database
    IF OBJECT_ID('ETL_Setting.dbo.OrderChanges', 'U') IS NOT NULL
        DROP TABLE ETL_Setting.dbo.OrderChanges;

    SELECT 
        L.OrderID AS LastNight_OrderID, 
        T.OrderID AS Tonight_OrderID,
        CASE
            WHEN L.OrderID IS NULL THEN 'Inserted'
            WHEN T.OrderID IS NULL THEN 'Deleted'
            ELSE 'Updated'
        END AS [Status]
    INTO ETL_Setting.dbo.OrderChanges
    FROM ETL_Setting.dbo.LastNight L
    FULL OUTER JOIN ETL_Setting.dbo.Tonight T
        ON L.OrderID = T.OrderID
    WHERE L.OrderID IS NULL OR T.OrderID IS NULL OR L.HashValue != T.HashValue;

    -- Apply changes to FactOrders table in data warehouse
    INSERT INTO NorthwindDW.dbo.FactOrders (OrderID, ProductID, OrderDate, EmployeeID, CustomerID, ShipVia, Freight, SalesAmount, DiscountAmount, NetAmount)
    SELECT T.OrderID, T.ProductID, T.OrderDate, T.EmployeeID, T.CustomerID, T.ShipVia, T.Freight, T.SalesAmount, T.DiscountAmount, T.NetAmount
    FROM Northwind.dbo.Orders T
    JOIN ETL_Setting.dbo.OrderChanges OC
        ON T.OrderID = OC.Tonight_OrderID
    WHERE OC.Status = 'Inserted';

    DELETE FROM NorthwindDW.dbo.FactOrders
    WHERE OrderID IN (
        SELECT LastNight_OrderID
        FROM ETL_Setting.dbo.OrderChanges
        WHERE Status = 'Deleted'
    );

    UPDATE F
    SET F.ProductID = O.ProductID,
        F.OrderDate = O.OrderDate,
        F.EmployeeID = O.EmployeeID,
        F.CustomerID = O.CustomerID,
        F.ShipVia = O.ShipVia,
        F.Freight = O.Freight,
        F.SalesAmount = O.SalesAmount,
        F.DiscountAmount = O.DiscountAmount,
        F.NetAmount = O.NetAmount
    FROM NorthwindDW.dbo.FactOrders F
    JOIN Northwind.dbo.Orders O
        ON F.OrderID = O.OrderID
    JOIN ETL_Setting.dbo.OrderChanges OC
        ON O.OrderID = OC.Tonight_OrderID
    WHERE OC.Status = 'Updated';

    -- Drop LastNight table in staging database
    DROP TABLE ETL_Setting.dbo.LastNight;

    -- Rename Tonight table to LastNight in staging database
    EXEC sp_rename 'ETL_Setting.dbo.Tonight', 'LastNight', 'OBJECT';
END
GO
