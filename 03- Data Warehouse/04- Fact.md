# Create Fact only Metadata
```sql
USE Northwind_DW
GO
--FactOrders:

CREATE TABLE [FactOrders] (
    [OrderID] INT NOT NULL,
    [CustomerID] NCHAR(5) NOT NULL,
    [EmployeeID] INT NOT NULL,
    [OrderDate] DATETIME NOT NULL,
    [ShipVia] INT NOT NULL,
    [Freight] MONEY NOT NULL,
    [ProductID] INT NOT NULL,
    [Quantity] SMALLINT NOT NULL,
    [UnitPrice] MONEY NOT NULL,
    [SalesAmount] MONEY NOT NULL,
    [DiscountAmount] Money NOT NULL,
    [NetAmount] Money NOT NULL,
    [EmployeeKey] INT NOT NULL,
    [ShipperKey] INT NOT NULL,
    [ProductKey] INT NOT NULL,
    [CustomerKey] INT NOT NULL,
    [MiladiDateKey] INT NOT NULL
)
GO
