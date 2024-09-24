```sql
--- SP Inferred Record:

CREATE OR ALTER PROCEDURE usp_GetCustomerKey @CustomerID nchar(5), @CustomerKey int OUTPUT
AS
 BEGIN
      SET NOCOUNT O
	  SELECT @CustomerKey = CustomerKey
	     FROM Northwind_DW.dbo.DimCustomers
	     WHERE CustomerID = @CustomerID and EndDate IS NULL ----> in Incremental Load
      IF @@ROWCOUNT = 0
	    BEGIN
INSERT INTO Northwind_DW.dbo.DimCustomers (CustomerID, CompanyName, Country,    Current_Country, StartDate)
SELECT CustomerID, CompanyName, ISNULL(Country , N'-'), ISNULL(Country , N'-'), GETDATE()
	        FROM Northwind.dbo.Customers
	        WHERE CustomerID = @CustomerID
             SET @CustomerKey = SCOPE_IDENTITY()
        END
 END
GO