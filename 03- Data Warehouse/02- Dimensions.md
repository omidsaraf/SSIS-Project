# Create Tables (Dimension) only Metadata
```sql
USE Northwind_DW
GO
--DimCustomers:
CREATE TABLE DimCustomers (CustomerKey INT NOT NULL IDENTITY PRIMARY KEY,
                             CustomerID NCHAR(5) NOT NULL,
			     CompanyName NVARCHAR(40) NOT NULL,
			     Country NVARCHAR(15) NOT NULL,
		             Current_Country NVARCHAR(15) NOT NULL,
			     StartDate DATETIME NOT NULL,
			     EndDate DATETIME NULL,
                         ISInferred Bit Not Null Default(0)
			     )
GO
--DimEmployees:
CREATE TABLE DimEmployees (EmployeeKey INT NOT NULL IDENTITY PRIMARY KEY,
                           EmployeeID INT NOT NULL,
                           FirstName NVARCHAR(10) NOT NULL,
		           LastName NVARCHAR(20) NOT NULL
			  )
GO
--DimSuppliers:

CREATE TABLE DimSuppliers (SupplierKey INT NOT NULL IDENTITY PRIMARY KEY,
                           SupplierID INT NOT NULL,
			  CompanyName NVARCHAR(40) NOT NULL,
			  Country NVARCHAR(15) NOT NULL
			     )
GO
--DimProducts:
CREATE TABLE [dbo].[DimProducts](
       [ProductKey] [INT] NOT NULL IDENTITY PRIMARY KEY,
	[ProductID] [int] NOT NULL,
	[ProductName] [nvarchar](40) NOT NULL,
	[CategoryID] [int] NOT NULL,
	[CategoryName] [nvarchar](15) NOT NULL,
	[Status] [nvarchar](8) NOT NULL,
	[SupplierKey] [int] NOT NULL,
	StartDate DATETIME NOT NULL,
	EndDate DATETIME NULL,
)
GO
--DimShippers:
CREATE TABLE DimShippers (ShipperKey INT NOT NULL IDENTITY PRIMARY KEY,
                          ShipperID INT NOT NULL,
		          CompanyName NVARCHAR(40) NOT NULL
			  )
GO