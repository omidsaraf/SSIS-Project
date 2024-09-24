```sql
--Make sure you set the Start and End Date below on row 58 and 59
--Create the tables
BEGIN TRY
DROP TABLE [DimDate]
END TRY
BEGIN CATCH
--DO NOTHING
END CATCH

CREATE TABLE [dbo].[DimDate](
--[DateSK] [int] IDENTITY(1,1) NOT NULL--Use this line if you just want an autoincrementing counter AND COMMENT BELOW LINE
  [DateKey] [int] NOT NULL--TO MAKE THE DateSK THE YYYYMMDD FORMAT USE THIS LINE AND COMMENT ABOVE LINE.
, [FullDate] [datetime] NOT NULL
, [Day] [tinyint] NOT NULL
, [DaySuffix] [varchar](4) NOT NULL
, [DayOfWeek] [varchar](9) NOT NULL
, [DayOfWeekNumber] [int] NOT NULL
, [DayOfWeekInMonth] [tinyint] NOT NULL
, [DayOfYearNumber] [int] NOT NULL
, [RelativeDays] int NOT NULL
, [WeekOfYearNumber] [tinyint] NOT NULL
, [WeekOfMonthNumber] [tinyint] NOT NULL
, [RelativeWeeks] int NOT NULL
, [CalendarMonthNumber] [tinyint] NOT NULL
, [CalendarMonthName] [varchar](9) NOT NULL
, [RelativeMonths] int NOT NULL
, [CalendarQuarterNumber] [tinyint] NOT NULL
, [CalendarQuarterName] [varchar](6) NOT NULL
, [RelativeQuarters] int NOT NULL
, [CalendarYearNumber] int NOT NULL
, [RelativeYears] int NOT NULL
, [StandardDate] [varchar](10) NULL
, [WeekDayFlag] bit NOT NULL
, [HolidayFlag] bit NOT NULL
, [OpenFlag] bit NOT NULL
, [FirstDayOfCalendarMonthFlag] bit NOT NULL
, [LastDayOfCalendarMonthFlag] bit NOT NULL
, [HolidayText] [varchar](50) NULL
CONSTRAINT [PK_DimDate] PRIMARY KEY CLUSTERED
(
[DateKey] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, FILLFACTOR = 90) ON [PRIMARY]
) ON [PRIMARY]

GO

--Populate Date dimension

TRUNCATE TABLE DimDate

--IF YOU ARE USING THE YYYYMMDD format for the primary key then you need to comment out this line.
--DBCC CHECKIDENT (DimDate, RESEED, 60000) --In case you need to add earlier dates later.

DECLARE @tmpDOW TABLE (DOW INT, Cntr INT)--Table for counting DOW occurance in a month
INSERT INTO @tmpDOW(DOW, Cntr) VALUES(1,0)--Used in the loop below
INSERT INTO @tmpDOW(DOW, Cntr) VALUES(2,0)
INSERT INTO @tmpDOW(DOW, Cntr) VALUES(3,0)
INSERT INTO @tmpDOW(DOW, Cntr) VALUES(4,0)
INSERT INTO @tmpDOW(DOW, Cntr) VALUES(5,0)
INSERT INTO @tmpDOW(DOW, Cntr) VALUES(6,0)
INSERT INTO @tmpDOW(DOW, Cntr) VALUES(7,0)

DECLARE @StartDate datetime
, @EndDate datetime
, @Date datetime
, @WDofMonth INT
, @CurrentMonth INT
, @CurrentDate date = getdate()

SELECT @StartDate = '1/01/1995'  -- -- Set The start and end date
, @EndDate = '1/01/2024'--Non inclusive. Stops on the day before this.
, @CurrentMonth = 1 --Counter used in loop below.

SELECT @Date = @StartDate

WHILE @Date < @EndDate
BEGIN

IF DATEPART(MONTH,@Date) <> @CurrentMonth
BEGIN
SELECT @CurrentMonth = DATEPART(MONTH,@Date)
UPDATE @tmpDOW SET Cntr = 0
END

UPDATE @tmpDOW
SET Cntr = Cntr + 1
WHERE DOW = DATEPART(DW,@DATE)

SELECT @WDofMonth = Cntr
FROM @tmpDOW
WHERE DOW = DATEPART(DW,@DATE)

INSERT INTO DimDate
(
[DateKey],       -- --TO MAKE THE DateSK THE YYYYMMDD FORMAT UNCOMMENT THIS LINE… Comment for autoincrementing.
[FullDate]
, [Day]
, [DaySuffix]
, [DayOfWeek]
, [DayOfWeekNumber]
, [DayOfWeekInMonth]
, [DayOfYearNumber]
, [RelativeDays]

, [WeekOfYearNumber]
, [WeekOfMonthNumber]
, [RelativeWeeks]

, [CalendarMonthNumber]
, [CalendarMonthName]
, [RelativeMonths]

, [CalendarQuarterNumber]
, [CalendarQuarterName]
, [RelativeQuarters]

, [CalendarYearNumber]
, [RelativeYears]

, [StandardDate]
, [WeekDayFlag]
, [HolidayFlag]
, [OpenFlag]
, [FirstDayOfCalendarMonthFlag]
, [LastDayOfCalendarMonthFlag]

)

SELECT

CONVERT(VARCHAR,@Date,112), --TO MAKE THE DateSK THE YYYYMMDD FORMAT UNCOMMENT THIS LINE COMMENT FOR AUTOINCREMENT
@Date [FullDate]
, DATEPART(DAY,@DATE) [Day]
, CASE
WHEN DATEPART(DAY,@DATE) IN (11,12,13) THEN CAST(DATEPART(DAY,@DATE) AS VARCHAR) + 'th'
WHEN RIGHT(DATEPART(DAY,@DATE),1) = 1 THEN CAST(DATEPART(DAY,@DATE) AS VARCHAR) + 'st'
WHEN RIGHT(DATEPART(DAY,@DATE),1) = 2 THEN CAST(DATEPART(DAY,@DATE) AS VARCHAR) + 'nd'
WHEN RIGHT(DATEPART(DAY,@DATE),1) = 3 THEN CAST(DATEPART(DAY,@DATE) AS VARCHAR) + 'rd'
ELSE CAST(DATEPART(DAY,@DATE) AS VARCHAR) + 'th'
END AS [DaySuffix]
, CASE DATEPART(DW, @DATE)
WHEN 1 THEN 'Sunday'
WHEN 2 THEN 'Monday'
WHEN 3 THEN 'Tuesday'
WHEN 4 THEN 'Wednesday'
WHEN 5 THEN 'Thursday'
WHEN 6 THEN 'Friday'
WHEN 7 THEN 'Saturday'
END AS [DayOfWeek]
,DATEPART(DW, @DATE) AS [DayOfWeekNumber]
, @WDofMonth [DOWInMonth]--Occurance of this day in this month. If Third Monday then 3 and DOW would be Monday.
, DATEPART(dy,@Date) [DayOfYearNumber]--Day of the year. 0 -- 365/366
, DATEDIFF(dd,@CurrentDate,@Date) as [RelativeDays]

, DATEPART(ww,@Date) [WeekOfYearNumber]--0-52/53
, DATEPART(ww,@Date) + 1 -
DATEPART(ww,CAST(DATEPART(mm,@Date) AS VARCHAR) + '/1/' + CAST(DATEPART(yy,@Date) AS VARCHAR)) [WeekOfMonthNumber]
, DATEDIFF(ww,@CurrentDate,@Date) as [RelativeWeeks]

, DATEPART(MONTH,@DATE) as [CalendarMonthNumber] --To be converted with leading zero later.
, DATENAME(MONTH,@DATE) as [CalendarMonthName]
, DATEDIFF(MONTH,@CurrentDate,@Date) as [RelativeMonths]

, DATEPART(qq,@DATE) as [CalendarQuarterNumber] --Calendar quarter
, CASE DATEPART(qq,@DATE)
WHEN 1 THEN 'First'
WHEN 2 THEN 'Second'
WHEN 3 THEN 'Third'
WHEN 4 THEN 'Fourth'
END AS [CalendarQuarterName]
, DATEDIFF(qq,@CurrentDate,@Date) as [RelativeQuarters]

, DATEPART(YEAR,@Date) as [CalendarYearNumber]
, DATEDIFF(YEAR,@CurrentDate,@Date) as [RelativeYears]

, RIGHT('0' + convert(varchar(2),MONTH(@Date)),2) + '/' + Right('0' + convert(varchar(2),DAY(@Date)),2) + '/' + convert(varchar(4),YEAR(@Date))
, CASE DATEPART(DW, @DATE)
WHEN 1 THEN 0
WHEN 2 THEN 1
WHEN 3 THEN 1
WHEN 4 THEN 1
WHEN 5 THEN 1
WHEN 6 THEN 1
WHEN 7 THEN 0
END AS [WeekDayFlag]

, 0 as HolidayFlag

, CASE DATEPART(DW, @DATE)
WHEN 1 THEN 0
WHEN 2 THEN 1
WHEN 3 THEN 1
WHEN 4 THEN 1
WHEN 5 THEN 1
WHEN 6 THEN 1
WHEN 7 THEN 1
END AS OpenFlag

, CASE DATEPART(dd,@Date)
WHEN 1
THEN 1
ELSE 0
END as [FirstDayOfCalendarMonthFlag]

, CASE
WHEN DateAdd(day, -1, DateAdd( month, DateDiff(month , 0,@Date)+1 , 0)) = @Date
THEN 1
ELSE 0
END as [LastDayOfCalendarMonthFlag]

SELECT @Date = DATEADD(dd,1,@Date)
END

--Add HOLIDAYS --------------------------------------------------------------------------

-- New Years Day --------------------------------------------------------------
UPDATE dbo.DimDate
SET HolidayText = 'New Year”s Day',
HolidayFlag = 1,
OpenFlag = 0
WHERE [CalendarMonthNumber] = 1 AND [DAY] = 1
--Set OpenFlag = 0 if New Year's Day is on weekend
UPDATE dbo.DimDate
SET OpenFlag = 0
WHERE DateKey in
(Select CASE
WHEN DayOfWeek = 'Sunday'
THEN DATEKey + 1
END
FRom DimDate
where CalendarMonthNumber = 1
and [DAY] = 1)

--Martin Luther King Day ----------------------------------------------------------
--Third Monday in January starting in 1983
UPDATE DimDate
SET HolidayText = 'Martin Luther King Jr. Day',
HolidayFlag = 1,
OpenFlag = 0
WHERE [CalendarMonthNumber] = 1--January
AND [Dayofweek] = 'Monday'
AND CalendarYearNumber >= 1983--When holiday was official
AND [DayOfWeekInMonth] = 3--Third X day of current month.
GO

--President's Day ----------------------------------------------------------
--Third Monday in February.
UPDATE DimDate
SET HolidayText = 'President”s Day',
HolidayFlag = 1,
OpenFlag = 0
WHERE [CalendarMonthNumber] = 2--February
AND [Dayofweek] = 'Monday'
AND [DayOfWeekInMonth] = 3--Third occurance of a monday in this month.
GO

--Memorial Day -----------------------------------------------------------
--Last Monday in May
UPDATE dbo.DimDate
SET HolidayText = 'Memorial Day',
HolidayFlag = 1,
OpenFlag = 0
FROM DimDate
WHERE DateKey IN
(
SELECT MAX([DateKey])
FROM dbo.DimDate
WHERE [CalendarMonthName] = 'May'
AND [DayOfWeek] = 'Monday'
GROUP BY CalendarYearNumber, [CalendarMonthNumber]
)

--4th of July --------------------------------------------------------------
UPDATE dbo.DimDate
SET HolidayText = 'Independance Day',
HolidayFlag = 1,
OpenFlag = 0
WHERE [CalendarMonthNumber] = 7 AND [DAY] = 4
--Set OpenFlag = 0 if July 4th is on weekend
UPDATE dbo.DimDate
SET OpenFlag = 0
WHERE DateKey in
(Select CASE
WHEN DayOfWeek = 'Sunday'
THEN DATEKey + 1
END
FRom DimDate
where CalendarMonthNumber = 7
and [DAY] = 4)

--Labor Day -------------------------------------------------------------
--First Monday in September
UPDATE dbo.DimDate
SET HolidayText = 'Labor Day',
HolidayFlag = 1,
OpenFlag = 0
FROM DimDate
WHERE DateKey IN
(
SELECT MIN([DateKey])
FROM dbo.DimDate
WHERE [CalendarMonthName] = 'September'
AND [DayOfWeek] = 'Monday'
GROUP BY CalendarYearNumber, [CalendarMonthNumber]
)

--Columbus Day------------------------------------------------------------
--2nd Monday in October
UPDATE dbo.DimDate
SET HolidayText = 'Columbus Day',
HolidayFlag = 1,
OpenFlag = 0
FROM DimDate
WHERE DateKey IN
(
SELECT MIN(DateKey)
FROM dbo.DimDate
WHERE [CalendarMonthName] = 'October'
AND [DayOfWeek] = 'Monday'
AND [DayOfWeekInMonth] = 2
GROUP BY CalendarYearNumber,
[CalendarMonthNumber]
)

--Veteran's Day --------------------------------------------------------------------------
UPDATE DimDate
SET HolidayText = 'Veteran”s Day',
HolidayFlag = 1,
OpenFlag = 0
WHERE DateKey in (
Select CASE
WHEN DayOfWeek = 'Saturday'
THEN DateKey -- 1
WHEN DayOfWeek = 'Sunday'
THEN DateKey + 1
ELSE DateKey
END as VeteransDateSK
FROM DimDate
WHERE [CalendarMonthNumber]  = 11
AND [DAY] = 11)
GO

--THANKSGIVING --------------------------------------------------------------------------
--Fourth THURSDAY in November.
UPDATE DimDate
SET HolidayText = 'Thanksgiving Day',
HolidayFlag = 1,
OpenFlag = 0
WHERE [CalendarMonthNumber] = 11
AND [DAYOFWEEK] = 'Thursday'
AND [DayOfWeekInMonth] = 4
GO

--CHRISTMAS -------------------------------------------------------------
UPDATE dbo.DimDate
SET HolidayText = 'Christmas Day',
HolidayFlag = 1,
OpenFlag = 0
WHERE [CalendarMonthNumber] = 12 AND [DAY] = 25
--Set OpenFlag = 0 if Christmas on weekend
UPDATE dbo.DimDate
SET OpenFlag = 0
WHERE DateKey in
(Select CASE
WHEN DayOfWeek = 'Sunday'
THEN DateKey + 1
WHEN Dayofweek = 'Saturday'
THEN DateKey -- 1
END
FRom DimDate
where CalendarMonthNumber = 12
and DAY = 25)

-- Valentine's Day --------------------------------------------------------------
UPDATE dbo.DimDate
SET HolidayText = 'Valentine”s Day'
WHERE CalendarMonthNumber = 2 AND [DAY] = 14

-- Saint Patrick's Day ------------------------------------------------------------
UPDATE dbo.DimDate
SET HolidayText = 'Saint Patrick”s Day'
WHERE [CalendarMonthNumber] = 3 AND [DAY] = 17
GO

--Mother's Day ----------------------------------------------------------
--Second Sunday of May
UPDATE DimDate
SET HolidayText = 'Mother”s Day'--select * from DimDate
WHERE [CalendarMonthNumber] = 5--May
AND [Dayofweek] = 'Sunday'
AND [DayOfWeekInMonth] = 2--Second occurance of a monday in this month.
GO
--Father's Day ----------------------------------------------------------
--Third Sunday of June
UPDATE DimDate
SET HolidayText = 'Father”s Day'--select * from DimDate
WHERE [CalendarMonthNumber] = 6--June
AND [Dayofweek] = 'Sunday'
AND [DayOfWeekInMonth] = 3--Third occurance of a monday in this month.
GO
--Halloween 10/31 -------------------------------------------------------
UPDATE dbo.DimDate
SET HolidayText = 'Halloween'
WHERE [CalendarMonthNumber] = 10 AND [DAY] = 31
--Election Day----------------------------------------------------------
--The first Tuesday after the first Monday in November.
BEGIN TRY
drop table #tmpHoliday
END TRY
BEGIN CATCH
--do nothing
END CATCH

CREATE TABLE #tmpHoliday(ID INT IDENTITY(1,1), DateID int, Week TINYINT, YEAR CHAR(4), DAY CHAR(2))

INSERT INTO #tmpHoliday(DateID, [YEAR],[DAY])
SELECT [DateKey], CalendarYearNumber, [DAY]
FROM dbo.DimDate
WHERE [CalendarMonthNumber] = 11
AND [Dayofweek] = 'Monday'
ORDER BY CalendarYearNumber, [DAY]

DECLARE @CNTR INT, @POS INT, @STARTYEAR INT, @ENDYEAR INT, @CURRENTYEAR INT, @MINDAY INT

SELECT @CURRENTYEAR = MIN([YEAR])
, @STARTYEAR = MIN([YEAR])
, @ENDYEAR = MAX([YEAR])
FROM #tmpHoliday

WHILE @CURRENTYEAR <= @ENDYEAR
BEGIN
SELECT @CNTR = COUNT([YEAR])
FROM #tmpHoliday
WHERE [YEAR] = @CURRENTYEAR

SET @POS = 1

WHILE @POS <= @CNTR
BEGIN
SELECT @MINDAY = MIN(DAY)
FROM #tmpHoliday
WHERE [YEAR] = @CURRENTYEAR
AND [WEEK] IS NULL

UPDATE #tmpHoliday
SET [WEEK] = @POS
WHERE [YEAR] = @CURRENTYEAR
AND [DAY] = @MINDAY

SELECT @POS = @POS + 1
END

SELECT @CURRENTYEAR = @CURRENTYEAR + 1
END

UPDATE DT
SET HolidayText = 'Election Day'
FROM dbo.DimDate DT
JOIN #tmpHoliday HL
ON (HL.DateID + 1) = DT.DateKey
WHERE [WEEK] = 1

DROP TABLE #tmpHoliday
GO
----------------------------------------------------------------------
PRINT CONVERT(VARCHAR,GETDATE(),113)--USED FOR CHECKING RUN TIME.

--DimDate indexes--------------------------------------------------------------
CREATE UNIQUE NONCLUSTERED INDEX [IDX_DimDate_Date] ON [dbo].[DimDate]
(
[FullDate] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, SORT_IN_TEMPDB = OFF, IGNORE_DUP_KEY = OFF, DROP_EXISTING = OFF, ONLINE = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, FILLFACTOR = 90) ON [PRIMARY]

CREATE NONCLUSTERED INDEX [IDX_DimDate_Day] ON [dbo].[DimDate]
(
[Day] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, SORT_IN_TEMPDB = OFF, IGNORE_DUP_KEY = OFF, DROP_EXISTING = OFF, ONLINE = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, FILLFACTOR = 90) ON [PRIMARY]

CREATE NONCLUSTERED INDEX [IDX_DimDate_DayOfWeek] ON [dbo].[DimDate]
(
[DayOfWeek] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, SORT_IN_TEMPDB = OFF, IGNORE_DUP_KEY = OFF, DROP_EXISTING = OFF, ONLINE = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, FILLFACTOR = 90) ON [PRIMARY]

CREATE NONCLUSTERED INDEX [IDX_DimDate_DOWInMonth] ON [dbo].[DimDate]
(
[DayOfWeekInMonth] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, SORT_IN_TEMPDB = OFF, IGNORE_DUP_KEY = OFF, DROP_EXISTING = OFF, ONLINE = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, FILLFACTOR = 90) ON [PRIMARY]

CREATE NONCLUSTERED INDEX [IDX_DimDate_DayOfYear] ON [dbo].[DimDate]
(
[DayOfYearNumber] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, SORT_IN_TEMPDB = OFF, IGNORE_DUP_KEY = OFF, DROP_EXISTING = OFF, ONLINE = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, FILLFACTOR = 90) ON [PRIMARY]

CREATE NONCLUSTERED INDEX [IDX_DimDate_WeekOfYear] ON [dbo].[DimDate]
(
[WeekOfYearNumber] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, SORT_IN_TEMPDB = OFF, IGNORE_DUP_KEY = OFF, DROP_EXISTING = OFF, ONLINE = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, FILLFACTOR = 90) ON [PRIMARY]

CREATE NONCLUSTERED INDEX [IDX_DimDate_WeekOfMonth] ON [dbo].[DimDate]
(
[WeekOfMonthNumber] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, SORT_IN_TEMPDB = OFF, IGNORE_DUP_KEY = OFF, DROP_EXISTING = OFF, ONLINE = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, FILLFACTOR = 90) ON [PRIMARY]

CREATE NONCLUSTERED INDEX [IDX_DimDate_Month] ON [dbo].[DimDate]
(
[CalendarMonthNumber] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, SORT_IN_TEMPDB = OFF, IGNORE_DUP_KEY = OFF, DROP_EXISTING = OFF, ONLINE = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, FILLFACTOR = 90) ON [PRIMARY]

CREATE NONCLUSTERED INDEX [IDX_DimDate_MonthName] ON [dbo].[DimDate]
(
[CalendarMonthName] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, SORT_IN_TEMPDB = OFF, IGNORE_DUP_KEY = OFF, DROP_EXISTING = OFF, ONLINE = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, FILLFACTOR = 90) ON [PRIMARY]

CREATE NONCLUSTERED INDEX [IDX_DimDate_Quarter] ON [dbo].[DimDate]
(
[CalendarQuarterNumber] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, SORT_IN_TEMPDB = OFF, IGNORE_DUP_KEY = OFF, DROP_EXISTING = OFF, ONLINE = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, FILLFACTOR = 90) ON [PRIMARY]

CREATE NONCLUSTERED INDEX [IDX_DimDate_QuarterName] ON [dbo].[DimDate]
(
[CalendarQuarterName] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, SORT_IN_TEMPDB = OFF, IGNORE_DUP_KEY = OFF, DROP_EXISTING = OFF, ONLINE = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, FILLFACTOR = 90) ON [PRIMARY]

CREATE NONCLUSTERED INDEX [IDX_DimDate_Year] ON [dbo].[DimDate]
(
[CalendarYearNumber] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, SORT_IN_TEMPDB = OFF, IGNORE_DUP_KEY = OFF, DROP_EXISTING = OFF, ONLINE = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, FILLFACTOR = 90) ON [PRIMARY]

CREATE NONCLUSTERED INDEX [IDX_dim_Time_HolidayText] ON [dbo].[DimDate]
(
[HolidayText] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, SORT_IN_TEMPDB = OFF, IGNORE_DUP_KEY = OFF, DROP_EXISTING = OFF, ONLINE = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, FILLFACTOR = 90) ON [PRIMARY]

PRINT convert(varchar,getdate(),113)--USED FOR CHECKING RUN TIME.

