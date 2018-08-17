---
layout: post
category: sql
title: Handy Sql snips
tagline: by Henry
tags:
  - sql
  - snips
published: true
---

# Handy Sql statements

## Sql statements
```SQL
spdbhelp databasename
use databasename
spspace
```

## Get tables in databases
``` SQL
SELECT * FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_TYPE='BASE TABLE'
```

## Get count of duplicate
``` SQL
;WITH Vals AS (
        SELECT [Value],
                ROW_NUMBER() OVER(PARTITION BY [Value] ORDER BY [Value]) RowID
        FROM    MyTable
    )
select * --DELETE
FROM    Vals
WHERE   RowID > 1
```

``` Sql
USER_LOOKUPS,          
USER_UPDATES
FROM     SYS.DM_DB_INDEX_USAGE_STATS AS S          
INNER JOIN SYS.INDEXES AS I ON I.[OBJECT_ID] = S.[OBJECT_ID]               
AND I.INDEX_ID = S.INDEX_ID
Join sys.Databases d on s.database_id = d.database_id
WHERE    OBJECTPROPERTY(S.[OBJECT_ID],'IsUserTable') = 1
Order by USER_SEEKS + USER_SCANS + USER_LOOKUPS + USER_UPDATES desc
```

``` Sql
SELECT d.name, t.name, OBJECT_NAME(A.[OBJECT_ID]) AS [OBJECT NAME],       
 I.[NAME] AS [INDEX NAME],       
  A.LEAF_INSERT_COUNT,        
  A.LEAF_UPDATE_COUNT,        
  A.LEAF_DELETE_COUNT

FROM   SYS.DM_DB_INDEX_OPERATIONAL_STATS (NULL,NULL,NULL,NULL ) A        
INNER JOIN SYS.INDEXES AS I          ON I.[OBJECT_ID] = A.[OBJECT_ID]   
join sys.tables t on i.object_id = t.object_id      
join sys.databases d on a.database_id = d.database_id
AND I.INDEX_ID = A.INDEX_ID WHERE  OBJECTPROPERTY(A.[OBJECT_ID],'IsUserTable') = 1

order by A.LEAF_INSERT_COUNT + A.LEAF_UPDATE_COUNT + A.LEAF_DELETE_COUNT desc
```

``` Sql
DECLARE @dbid int
SELECT @dbid = db_id('databasename')

SELECT TableName = object_name(s.object_id),
       Reads = SUM(user_seeks + user_scans + user_lookups), Writes =  SUM(user_updates)
FROM sys.dm_db_index_usage_stats AS s
INNER JOIN sys.indexes AS i
ON s.object_id = i.object_id
AND i.index_id = s.index_id
WHERE objectproperty(s.object_id,'IsUserTable') = 1
AND s.database_id = @dbid
GROUP BY object_name(s.object_id)
ORDER BY writes DESC
```

``` SQL
SELECT
    t.NAME AS TableName,
    s.Name AS SchemaName,
    p.rows AS RowCounts,
    SUM(a.total_pages) * 8 AS TotalSpaceKB,
    CAST(ROUND(((SUM(a.total_pages) * 8) / 1024.00), 2) AS NUMERIC(36, 2)) AS TotalSpaceMB,
    SUM(a.used_pages) * 8 AS UsedSpaceKB,
    CAST(ROUND(((SUM(a.used_pages) * 8) / 1024.00), 2) AS NUMERIC(36, 2)) AS UsedSpaceMB,
    (SUM(a.total_pages) - SUM(a.used_pages)) * 8 AS UnusedSpaceKB,
    CAST(ROUND(((SUM(a.total_pages) - SUM(a.used_pages)) * 8) / 1024.00, 2) AS NUMERIC(36, 2)) AS UnusedSpaceMB
FROM
    sys.tables t
INNER JOIN      
    sys.indexes i ON t.OBJECT_ID = i.object_id
INNER JOIN
    sys.partitions p ON i.object_id = p.OBJECT_ID AND i.index_id = p.index_id
INNER JOIN
    sys.allocation_units a ON p.partition_id = a.container_id
LEFT OUTER JOIN
    sys.schemas s ON t.schema_id = s.schema_id
WHERE
    t.NAME NOT LIKE 'dt%'
    AND t.is_ms_shipped = 0
    AND i.OBJECT_ID > 255
GROUP BY
    t.Name, s.Name, p.Rows
ORDER BY
    t.Name
```

## Find sql locks
``` SQL
SELECT  L.request_session_id AS SPID,
        DB_NAME(L.resource_database_id) AS DatabaseName,
        O.Name AS LockedObjectName,
        P.object_id AS LockedObjectId,
        L.resource_type AS LockedResource,
        L.request_mode AS LockType,
        ST.text AS SqlStatementText,        
        ES.login_name AS LoginName,
        ES.host_name AS HostName,
        TST.is_user_transaction as IsUserTransaction,
        AT.name as TransactionName,
        CN.auth_scheme as AuthenticationMethod
FROM    sys.dm_tran_locks L
        JOIN sys.partitions P ON P.hobt_id = L.resource_associated_entity_id
        JOIN sys.objects O ON O.object_id = P.object_id
        JOIN sys.dm_exec_sessions ES ON ES.session_id = L.request_session_id
        JOIN sys.dm_tran_session_transactions TST ON ES.session_id = TST.session_id
        JOIN sys.dm_tran_active_transactions AT ON TST.transaction_id = AT.transaction_id
        JOIN sys.dm_exec_connections CN ON CN.session_id = ES.session_id
        CROSS APPLY sys.dm_exec_sql_text(CN.most_recent_sql_handle) AS ST
WHERE   resource_database_id = db_id()
ORDER BY L.request_session_id
```


## Sql query everything
``` Sql
-- SQL Script to capture the database table structure
select
[table_name] as  [Table Name],
[column_name] as [Column Name],
case [data_type]
  when 'varchar' then [data_type] + '(' + cast([character_maximum_length]  as varchar) + ')'
  when 'nvarchar' then [data_type] + '(' + cast([character_maximum_length]  as nvarchar) + ')'
  else [data_type]
end as [Data Type],
case [is_nullable]
  when 'No' then 'No'
  else 'Yes'
end as [Nullable],
isnull([column_default], '') as [Default Value],
isnull(cast([numeric_precision] as nvarchar),'') as [Precision],
isnull(cast([numeric_precision_radix] as nvarchar),'') as [Precision Radix],
isnull([collation_name],'') as [Collation Name]
from information_schema.[columns]
where [table_catalog]   like '%'
    and [table_schema]  like 'dbo'
    and [table_name]     like '%'
    and [column_name]  like '%Lastlogon%'
    and [data_type]        like '%'
order by [table_name], [ordinal_position]
-- SQL Script to capture the names of the database objects
select distinct
case [xtype]
 when 'C'   then 'CHECK constraint'
 when 'D'   then 'DEFAULT constraint'
 when 'F'    then 'FOREIGN KEY constraint'
 when 'L'    then 'Log'
 when 'FN' then 'Scalar function'
 when 'IF'   then 'Inlined table-function'
 when 'P'    then 'Stored procedure'
 when 'PK' then 'PRIMARY KEY constraint'
 when 'RF'  then 'Replication filter stored procedure'
 when 'S'    then 'SYSTEM table'
 when 'TF'  then 'Table function'
 when 'TR'  then 'Trigger'
 when 'U'    then 'User table'
 when 'UQ' then 'UNIQUE constraint'
 when 'V'    then 'View'
 when 'X'    then 'Extended stored procedure'
end as [xtype],
[name] as [Name]
from [dbo].[sysobjects]
where [xtype] in ('C', 'D', 'FN','P', 'PK', 'TR', 'U', 'V')
  and [name] like '%'
order by [xtype], [name]
-- SQL Script to capture the Names and Definitions of Views
select
   [table_name] as [View Name],
   [view_definition] as [View Definition]
from information_schema.[views]
where [table_name] like '%'
order by [table_name]

-- SQL Script to capture the Names and Definitions of functions and procedures
select
   [routine_type] as [Routine Type],
   [routine_name] as [Routine Name],
   [routine_definition] as [Routine Definition]
from information_schema.[routines]
where [routine_type] in ('FUNCTION','PROCEDURE')
    and [routine_name]  like '%'
order by [routine_name]
-- SQL Script to capture the Names and Parameters of functions and procedures
select
   [specific_name] as [Routine Name],
   [parameter_name] as [Parameter Name],
case [data_type]
   when 'varchar' then [data_type] + '(' + cast([character_maximum_length]  as varchar) + ')'
   when 'nvarchar' then [data_type] + '(' + cast([character_maximum_length]  as nvarchar) + ')'
   else [data_type]
end as [Data Type]
from information_schema.[parameters]
where [specific_name] like '%'
order by [specific_name], [ordinal_position]

-- SQL Script to capture the GUID and Table names
-- This could help your SQL joins on what table have what guids.
select
[column_name] as [Guid Name],
[table_name] as  [Table Name],
case [is_nullable]
when 'YES' then 'Yes'
  else 'No'
end as [Nullable],
isnull([column_default], '') as [Default Value]
from information_schema.[columns]
where [table_schema]  = 'dbo'
  and [data_type] like 'uniqueidentifier'
order by [column_name], [table_name]


-- To look at the table or view definition

sp_help TableName
sp_help ViewName

sp_columns TableName

-- To look at what indexes are in a table

sp_helpindex TableName

-- To look at the definition of a stored procedure or a view

sp_helptext StoredProcedureName


sp_helptext ViewName

-- To look at the database object (tables, views, stored procedures) dependencies

sp_depends DatabaseObject"

```


## Sql the State everything
``` sql
Declare @StartDate datetime =  getdate()

Declare @TrendCurrentMonthStart  datetime = DATEADD(month, DATEDIFF(month,1 , @StartDate), 0)
Declare @TrendCurrentMonthEnd datetime = EOMONTH(@StartDate)
Declare @TrendTMinus1Start datetime = DATEADD(month, DATEDIFF(month,40 , @StartDate), 0)
Declare @TrendTMinus1End datetime = EOMONTH(dateadd(month,datediff(month,40,@StartDate),0))
Declare @TrendTMinus2Start datetime = DATEADD(month, DATEDIFF(month,80 , @StartDate), 0)
Declare @TrendTMinus2End datetime = EOMONTH(dateadd(month,datediff(month,80,@StartDate),0))
Declare @TrendTMinus3Start datetime = DATEADD(month, DATEDIFF(month,100 , @StartDate), 0)
Declare @TrendTMinus3End datetime =  EOMONTH(dateadd(month,datediff(month,100,@StartDate),0))


select  @TrendCurrentMonthStart
select  @TrendCurrentMonthEnd  
select  @TrendTMinus1Start
select  @TrendTMinus1End
select  @TrendTMinus2Start
select  @TrendTMinus2End
select  @TrendTMinus3Start
select  @TrendTMinus3End

Declare @StartDate datetime =  getdate()

Declare @TrendCurrentMonthStart  datetime = DATEADD(month, DATEDIFF(month,1 , @StartDate), 0)
Declare @TrendCurrentMonthEnd datetime = EOMONTH(@StartDate)
Declare @TrendTMinus1Start datetime = DATEADD(month, DATEDIFF(month,40 , @StartDate), 0)
Declare @TrendTMinus1End datetime = EOMONTH(dateadd(month,datediff(month,40,@StartDate),0))
Declare @TrendTMinus2Start datetime = DATEADD(month, DATEDIFF(month,80 , @StartDate), 0)
Declare @TrendTMinus2End datetime = EOMONTH(dateadd(month,datediff(month,80,@StartDate),0))
Declare @TrendTMinus3Start datetime = DATEADD(month, DATEDIFF(month,100 , @StartDate), 0)
Declare @TrendTMinus3End datetime =  EOMONTH(dateadd(month,datediff(month,100,@StartDate),0))


select  @TrendCurrentMonthStart
select  @TrendCurrentMonthEnd  
select  @TrendTMinus1Start
select  @TrendTMinus1End
select  @TrendTMinus2Start
select  @TrendTMinus2End
select  @TrendTMinus3Start
select  @TrendTMinus3End



--SQL Round to nearest Minute
select dateadd(mi, datediff(mi, 0, @dt), 0)

--Sql Round to Nearest Hour
select dateadd(hour, datediff(hour, 0, @dt), 0)

--Sql Last day of week
first day - last day

--Sql First Day of Week
select dateadd(week, datediff(week, 0, getdate()), 0);

--Sql Get DAte Name
SELECT DATENAME(dw,GETDATE()) -- Friday

--Sql Get Date Number
SELECT DATEPART(dw,GETDATE()) -- 6

--Sql Last day of month
DECLARE @date DATETIME = GETDATE();  
SELECT EOMONTH ( @date ) AS 'This Month';  
SELECT EOMONTH ( @date, 1 ) AS 'Next Month';  
SELECT EOMONTH ( @date, -1 ) AS 'Last Month';  

--Sql first day of the month
select dateadd( s, -1, dateadd( mm, datediff( m, 0, getdate() ) + 1, 0 ) );
/*
Explantion:
			To understand how it works we have to look at the dateadd() and datediff() functions.
			DATEADD(datepart, number, date)  
			DATEDIFF(datepart, startdate, enddate)
			If you run just the most inner call to datediff(), you get the current month number since timestamp 0.
			select datediff(m, 0, getdate() );  
			1327
			The next part adds that number of months plus 1 to the 0 timestamp, giving you the starting point of the next calendar month.
			select dateadd( s, -1, dateadd( mm, datediff( m, 0, getdate() ) + 1, 0 ) );
			2010-08-31 23:59:59.000
			select dateadd( mm, datediff( m, 0, getdate() ) + 1, 0 );
			2010-09-01 00:00:00.000
/*


select datepart(day,getdate())
select datepart(dayofyear,getdate())
select datepart(WEEKDAY,getdate())
select datepart(DAYOFYEAR,getdate())
select datepart(YEAR,getdate())
select datepart(MONTH,getdate())
select datepart(WEEK,getdate())
select datepart(HOUR,getdate())
select datepart(MINUTE,getdate())
select datepart(SECOND,getdate())
select datepart(MILLISECOND,getdate())
select datepart(MICROSECOND,getdate())
select datepart(NANOSECOND,getdate())
select datepart(YEAR,getdate())
select datepart(QUARTER,getdate())



/*
datepart	Abbreviation
year		yy, yyyy
quarter		qq, q
month		mm, m
dayofyear	dy, y
day			dd, d
week		wk, ww
weekday		dw, w
hour		hh
minute		mi, n
second		ss, s
millisecond	ms
microsecond	mcs
nanosecond	ns
*/



--Other Ways
----Last Day of Previous Month
SELECT DATEADD(s,-1,DATEADD(mm, DATEDIFF(m,0,GETDATE()),0))
LastDay_PreviousMonth
----Last Day of Current Month
SELECT DATEADD(s,-1,DATEADD(mm, DATEDIFF(m,0,GETDATE())+1,0))
LastDay_CurrentMonth
----Last Day of Next Month
SELECT DATEADD(s,-1,DATEADD(mm, DATEDIFF(m,0,GETDATE())+2,0))
LastDay_NextMonth

--First Day of Last Year
SELECT DATEADD(YEAR, DATEDIFF(YEAR, '19000101', GETDATE()) - 1 , '19000101')
&nbsp;AS [FIRST DAY OF LAST YEAR];
GO

--First Day of This Year
SELECT DATEADD(YEAR, DATEDIFF(YEAR, '19000101', GETDATE()), '19000101')
&nbsp;AS [FIRST DAY OF This YEAR];
GO

--First Day of Next Year
SELECT DATEADD(YEAR, DATEDIFF(YEAR, '19000101', GETDATE()) + 1 , '19000101')
&nbsp;AS [FIRST DAY OF NEXT YEAR];
GO

--Last Day of Last Year
SELECT DATEADD(d, -1, DATEADD(YEAR, DATEDIFF(YEAR, '19000101', GETDATE()), '19000101'))
 AS [LAST DAY OF This YEAR];
GO

--Last Day of This Year
SELECT DATEADD(d, -1, DATEADD(YEAR, DATEDIFF(YEAR, '19000101', GETDATE()) + 1 , '19000101'))
 AS [LAST DAY OF This YEAR];
GO

--Last Day of Next Year
SELECT DATEADD(d, -1, DATEADD(YEAR, DATEDIFF(YEAR, '19000101', GETDATE()) + 2 , '19000101'))
 AS [LAST DAY OF NEXT YEAR];
GO

-- To Get First Day of Previous Month
SELECT DATEADD(MONTH, DATEDIFF(MONTH, '19000101', GETDATE()) - 1, '19000101')
 AS [FIRST DAY Previous MONTH];
GO

-- To Get First Day of Current Month
SELECT DATEADD(MONTH, DATEDIFF(MONTH, '19000101', GETDATE()), '19000101')
 AS [FIRST DAY CURRENT MONTH];
GO

-- To Get First Day of Next Month
SELECT DATEADD(MONTH, DATEDIFF(MONTH, '19000101', GETDATE()) + 1, '19000101')
 AS [FIRST DAY NEXT MONTH];
GO

-- To Get Last Day of Previous Month
SELECT DATEADD(D, -1, DATEADD(MONTH, DATEDIFF(MONTH, '19000101', GETDATE()), '19000101'))
AS [LAST DAY Previous MONTH];
GO

-- To Get Last Day of This Month
SELECT DATEADD(D, -1, DATEADD(MONTH, DATEDIFF(MONTH, '19000101', GETDATE()) + 1, '19000101'))
AS [LAST DAY This MONTH];
GO

-- To Get Last Day of Next Month
SELECT DATEADD(D, -1, DATEADD(MONTH, DATEDIFF(MONTH, '19000101', GETDATE()) + 2, '19000101'))
AS [LAST DAY NEXT MONTH];
GO

-- To Get Midnight Yesterday
SELECT DATEADD(d, -1, DATEDIFF(d, 0, GETDATE()))
 AS [Midnight Yesterday];

-- To Get Midnight Today
SELECT DATEADD(d, -0, DATEDIFF(d, 0, GETDATE()))
 AS [Midnight Today];

-- To Get Midnight Tomorrow
SELECT DATEADD(d, 1, DATEDIFF(d, 0, GETDATE()))
 AS [Midnight Tomorrow];


 --To Get 11:59:59 Yesterday
SELECT DATEADD(ss, (60*60*24)-1, DATEADD(d, -1, DATEDIFF(d, 0, GETDATE())))
 AS [11:59:59 Yesterday];


--To Get Noon Yesterday
SELECT DATEADD(hh, 12, DATEADD(d, -1, DATEDIFF(d, 0, GETDATE())))
 AS [Noon Yesterday];


--To Get 11:59:59:997 Yesterday
SELECT DATEADD(ms, (1000*60*60*24)-2, DATEADD(d, -1, DATEDIFF(d, 0, GETDATE())))
 AS [11:59:59.997 Yesterday];

```
