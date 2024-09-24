``` sql
--SP for Key Look up ProductKey:

USE Northwind_DW
GO
CREATE OR ALTER PROCEDURE usp_LookupProductKeyInDimProducts @ProductID NCHAR(5), @OrderDate DATETIME, @ProductKey INT OUTPUT
AS
 BEGIN
    SET NOCOUNT ON
	DECLARE @MaxDate DATETIME = '9999-12-30 00:00:00.000'
	IF @OrderDate < (SELECT MIN(StartDate) FROM DimProducts WHERE ProductID = @ProductID)
            SELECT @ProductKey = MIN(ProductKey) FROM DimProducts WHERE ProductID = @ProductID
	ELSE
	    SELECT @ProductKey = ProductKey
	    FROM DimProducts
	  WHERE ProductID = @ProductID AND @OrderDate BETWEEN StartDate AND ISNULL(EndDate , @MaxDate)
END

---SP for Key Look up CustomerKey:

USE Northwind_DW
GO
CREATE OR ALTER PROCEDURE usp_LookupCustomerKeyInDimCustomers @CustomerID NCHAR(5), @OrderDate DATETIME, @CustomerKey INT OUTPUT
AS
 BEGIN
    SET NOCOUNT ON
      DECLARE @MaxDate DATETIME = '9999-12-30 00:00:00.000'
       IF @OrderDate < (SELECT MIN(StartDate) FROM DimCustomers WHERE CustomerID = @CustomerID)
	SELECT @CustomerKey = MIN(CustomerKey) FROM DimCustomers WHERE CustomerID = @CustomerID
	ELSE
	  SELECT @CustomerKey = CustomerKey
	  FROM DimCustomers
        WHERE CustomerID = @CustomerID AND @OrderDate BETWEEN StartDate AND ISNULL(EndDate , @MaxDat
END

