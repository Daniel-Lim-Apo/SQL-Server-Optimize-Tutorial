# SQL-Server-Optimize-Tutorial

Tutorial about how to optimze and tunning SQL Server Databases

# How to Know the Size of One Record in One Table in SQL Server

In SQL Server, the size of a single record (or row) in a table depends on various factors such as the data types of the columns, any overhead added by the row itself, and other potential storage considerations (such as variable-length columns).

To know the size of one record in a SQL Server table, you can either calculate it manually based on the column data types or use system views and DMVs (dynamic management views) to get average record size data. Using these techniques, you can get a clear picture of the space consumption per record.

## Methods to Determine the Size of One Record in a Table:

### 1. Using `sys.dm_db_index_physical_stats` DMV

You can retrieve information about the size of pages in a table using the `sys.dm_db_index_physical_stats` dynamic management view (DMV). This gives you an approximate idea of the average record size.

```sql
SELECT
    avg_record_size_in_bytes
FROM
    sys.dm_db_index_physical_stats(DB_ID(N'DatabaseName'), OBJECT_ID(N'Schema.TableName'), NULL, NULL, 'DETAILED');
```

- Replace `DatabaseName` with the name of your database.
- Replace `Schema.TableName` with the schema and table name.

This query returns the average size of a record in bytes for that table.

### 2. Using `sp_spaceused` Stored Procedure

You can use the `sp_spaceused` stored procedure to determine the total size of the table and the number of rows. With this, you can calculate the average row size by dividing the total size by the number of rows.

```sql
EXEC sp_spaceused N'Schema.TableName';
```

This will return various pieces of information such as the total number of rows and the total data size. To calculate the average row size:

```sql
-- Example query to calculate average row size:
EXEC sp_spaceused N'Schema.TableName';

-- Then, use the following formula:
-- Average Row Size (in KB) = DataSize (in KB) / Number of Rows
-- Average Row Size (in Bytes) = (DataSize (in KB) / Number of Rows) * 1024
```

### 3. Using Column Data Types for Manual Calculation

You can manually calculate the size of a single record by examining the data types of each column and summing the sizes for a single row.

For example, if you have a table like this:

```sql
CREATE TABLE SampleTable (
    ID INT,
    Name NVARCHAR(50),
    Age INT
);
```

You can calculate the size of each data type:

- `INT` takes 4 bytes.
- `NVARCHAR(50)` can take up to 100 bytes (50 characters, 2 bytes each for Unicode).

Assuming a row contains data for each field, a single row size will be:

```
Size of row = 4 bytes (ID) + (Actual size of Name value) + 4 bytes (Age)
```

For variable-length columns like `NVARCHAR`, the size will depend on the actual length of the data stored in that column. So, in this case, the row size may vary based on the length of the `Name` field.

### 4. Query to Check Row Size (Using DMVs)

To get the size of all rows in a table along with the number of rows, you can use this query:

```sql
SELECT
    t.NAME AS TableName,
    p.rows AS NumberOfRows,
    SUM(a.total_pages) * 8 AS TotalSpaceKB,
    SUM(a.used_pages) * 8 AS UsedSpaceKB,
    (SUM(a.total_pages) - SUM(a.used_pages)) * 8 AS UnusedSpaceKB,
    (SUM(a.used_pages) * 8) / p.rows AS AverageRowSizeBytes
FROM
    sys.tables t
INNER JOIN
    sys.indexes i ON t.OBJECT_ID = i.object_id
INNER JOIN
    sys.partitions p ON i.object_id = p.OBJECT_ID AND i.index_id = p.index_id
INNER JOIN
    sys.allocation_units a ON p.partition_id = a.container_id
WHERE
    t.NAME = 'YourTableName'
GROUP BY
    t.NAME, p.Rows;
```

### Notes:

- `TotalSpaceKB`: Total space allocated to the table (in KB).
- `UsedSpaceKB`: Actual space used by the data.
- `AverageRowSizeBytes`: Calculated size of an average row in bytes.

This method gives you a reasonable approximation of the size of an individual record based on the current data in the table.
