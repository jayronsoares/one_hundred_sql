## A Comprehensive Guide to SQL Query Tuning

<p>It's all about cardinality</p>

### Step 1: Identify Slow Queries

The first step in SQL query tuning is to identify slow-running queries. Utilizing database-specific tools can help you pinpoint queries with high execution times.

**PostgreSQL**:
```sql
SELECT pid, now() - pg_stat_activity.query_start AS duration, query
FROM pg_stat_activity
WHERE state = 'active' AND now() - pg_stat_activity.query_start > interval '1 minute';
```

**SQL Server**: Use Query Store or SQL Server Profiler to capture slow queries.

**MySQL**: Enable the Slow Query Log or use the Performance Schema.

### Step 2: Examine Execution Plans

<p>Understanding how a query is executed is essential. Use `EXPLAIN` or `EXPLAIN ANALYZE` to obtain the execution plan.</p>

**PostgreSQL**:
```sql
EXPLAIN ANALYZE SELECT * FROM my_table WHERE column = 'value';
```

**SQL Server**:
```sql
SET STATISTICS PROFILE ON;
SELECT * FROM my_table WHERE column = 'value';
SET STATISTICS PROFILE OFF;
```

**MySQL**:
```sql
EXPLAIN SELECT * FROM my_table WHERE column = 'value';
```

### Step 3: Check Cardinality Estimates

<p>Analyze the execution plan to compare estimated vs. actual row counts. Significant discrepancies often indicate inaccurate cardinality estimates.
Cardinality refers to the number of distinct values in a column or the number of rows produced by a query operation. Accurate cardinality estimation is crucial for the database optimizer to choose the most efficient execution plan.</p>

<p>These SQL queries for PostgreSQL provide a detailed view of the estimated vs actual rows cardinality context, including index scans, tuple reads and fetches, and table size.</p>

### Option 1: Using `pg_stats` and `pg_class`

```sql
SELECT
    relname AS table_name,
    reltuples::bigint AS actual_cardinality,
    (SELECT reltuples::bigint FROM pg_class WHERE relname = '{table_name}') AS estimated_cardinality,
    idx_scan AS index_scans,
    idx_tup_read AS index_tuple_reads,
    idx_tup_fetch AS index_tuple_fetches,
    pg_size_pretty(pg_total_relation_size(relid)) AS table_size
FROM
    pg_stat_all_tables
WHERE
    relname = '{table_name}';
```

### Option 2: Using `pg_stats` and `pg_index`

```sql
SELECT
    t.relname AS table_name,
    s.n_live_tup AS actual_cardinality,
    (SELECT s.n_live_tup FROM pg_stat_user_tables s INNER JOIN pg_class t ON s.relid = t.oid WHERE t.relname = '{table_name}') AS estimated_cardinality,
    idx_scan AS index_scans,
    idx_tup_read AS index_tuple_reads,
    idx_tup_fetch AS index_tuple_fetches,
    pg_size_pretty(pg_total_relation_size(t.oid)) AS table_size
FROM
    pg_stat_user_tables s
INNER JOIN
    pg_class t ON s.relid = t.oid
INNER JOIN
    pg_index i ON t.oid = i.indrelid
WHERE
    t.relname = '{table_name}';
```
<p>A comprehensive context for the estimated vs actual rows cardinality, to include additional information such as index statistics, fragmentation, and index usage statistics.</p>

### SQL Server:

#### Option 1: `sys.dm_db_partition_stats` and `sys.dm_db_index_usage_stats`

```sql
SELECT
    OBJECT_NAME(p.object_id) AS table_name,
    SUM(p.rows) AS actual_cardinality,
    (SELECT SUM(p.rows) FROM sys.dm_db_partition_stats p WHERE OBJECT_NAME(p.object_id) = '{table_name}') AS estimated_cardinality,
    SUM(s.user_seeks + s.user_scans) AS index_usage,
    s.avg_fragmentation_in_percent AS fragmentation_percentage
FROM 
    sys.dm_db_partition_stats p
INNER JOIN 
    sys.dm_db_index_usage_stats s ON p.object_id = s.object_id
WHERE 
    OBJECT_NAME(p.object_id) = '{table_name}'
GROUP BY 
    p.object_id, s.user_seeks, s.user_scans, s.avg_fragmentation_in_percent;
```

#### Option 2: `sys.dm_db_index_physical_stats`

```sql
SELECT 
    OBJECT_NAME(p.object_id) AS table_name,
    SUM(p.rows) AS actual_cardinality,
    (SELECT SUM(p.rows) FROM sys.dm_db_index_physical_stats(DB_ID(), OBJECT_ID('{table_name}'), NULL, NULL, 'DETAILED')) AS estimated_cardinality,
    p.avg_fragmentation_in_percent
FROM 
    sys.dm_db_index_physical_stats(DB_ID(), OBJECT_ID('{table_name}'), NULL, NULL, 'DETAILED') p
WHERE 
    OBJECT_NAME(p.object_id) = '{table_name}'
GROUP BY 
    p.object_id, p.avg_fragmentation_in_percent;
```

### MySQL:

#### Option 1: `information_schema.TABLES` and `information_schema.STATISTICS`

```sql
SELECT
    TABLE_NAME,
    TABLE_ROWS AS actual_cardinality,
    (SELECT TABLE_ROWS FROM information_schema.TABLES WHERE TABLE_SCHEMA = '{database_name}' AND TABLE_NAME = '{table_name}') AS estimated_cardinality,
    INDEX_NAME,
    NON_UNIQUE,
    SEQ_IN_INDEX,
    INDEX_TYPE,
    COMMENT
FROM 
    information_schema.TABLES t
INNER JOIN 
    information_schema.STATISTICS s ON t.TABLE_SCHEMA = s.TABLE_SCHEMA AND t.TABLE_NAME = s.TABLE_NAME
WHERE 
    t.TABLE_NAME = '{table_name}';
```

#### Option 2: `SHOW INDEX` and `SHOW TABLE STATUS`

```sql
SHOW INDEX FROM {table_name};
SHOW TABLE STATUS WHERE Name = '{table_name}';
```       

### Step 4: Update Statistics

<p>Updating statistics ensures the query optimizer has the most current data distribution information.</p>

**PostgreSQL**:
```sql
ANALYZE my_table;
```

**SQL Server**:
```sql
UPDATE STATISTICS my_table;
```

**MySQL**:
```sql
ANALYZE TABLE my_table;
```

### Step 5: Create/Update Indexes

<p>Indexes are critical for query performance. Ensure appropriate indexes are in place, utilized in WHERE and JOIN clauses, and consider composite indexes for columns frequently used together in queries.</p>

**PostgreSQL**:
```sql
CREATE INDEX idx_column ON my_table(column);
CREATE INDEX idx_composite ON my_table(column1, column2);
```

**SQL Server**:
```sql
CREATE INDEX idx_column ON my_table(column);
CREATE INDEX idx_composite ON my_table(column1, column2);
```

**MySQL**:
```sql
CREATE INDEX idx_column ON my_table(column);
CREATE INDEX idx_composite ON my_table(column1, column2);
```

### Step 6: Rewrite Queries

<p>Rewriting complex queries can enhance performance. Breaking down queries or using Common Table Expressions (CTEs) can be beneficial.</p>

**PostgreSQL**:
```sql
WITH cte AS (
  SELECT * FROM my_table WHERE column = 'value'
)
SELECT * FROM cte WHERE another_column = 'another_value';
```

**SQL Server**:
```sql
WITH cte AS (
  SELECT * FROM my_table WHERE column = 'value'
)
SELECT * FROM cte WHERE another_column = 'another_value';
```

**MySQL** (from version 8.0):
```sql
WITH cte AS (
  SELECT * FROM my_table WHERE column = 'value'
)
SELECT * FROM cte WHERE another_column = 'another_value';
```

### Step 7: Partitioning

For large tables, partitioning can significantly improve query performance by reducing the amount of data scanned.

**PostgreSQL**:
```sql
CREATE TABLE my_table (
  id SERIAL PRIMARY KEY,
  column1 INT,
  column2 TEXT
) PARTITION BY RANGE (column1);

CREATE TABLE my_table_part1 PARTITION OF my_table FOR VALUES FROM (1) TO (1000);
CREATE TABLE my_table_part2 PARTITION OF my_table FOR VALUES FROM (1000) TO (2000);
```

**SQL Server**:
```sql
CREATE PARTITION FUNCTION myRangePF1 (int)
AS RANGE LEFT FOR VALUES (1000, 2000, 3000);

CREATE PARTITION SCHEME myRangePS1
AS PARTITION myRangePF1 ALL TO ([PRIMARY]);

CREATE TABLE my_table (
  id INT,
  column1 INT,
  column2 NVARCHAR(50)
) ON myRangePS1(column1);
```

**MySQL**:
```sql
CREATE TABLE my_table (
  id INT PRIMARY KEY,
  column1 INT,
  column2 VARCHAR(255)
)
PARTITION BY RANGE (column1) (
  PARTITION p0 VALUES LESS THAN (1000),
  PARTITION p1 VALUES LESS THAN (2000),
  PARTITION p2 VALUES LESS THAN (3000)
);
```

### Step 8: Materialized Views

Materialized views store precomputed query results, which can greatly improve performance for complex queries.

**PostgreSQL**:
```sql
CREATE MATERIALIZED VIEW my_mv AS
SELECT column1, COUNT(*) FROM my_table GROUP BY column1;
REFRESH MATERIALIZED VIEW my_mv;
```

**SQL Server**:
```sql
CREATE VIEW my_mv WITH SCHEMABINDING AS
SELECT column1, COUNT_BIG(*) AS count
FROM dbo.my_table
GROUP BY column1;

CREATE UNIQUE CLUSTERED INDEX idx_my_mv ON my_mv (column1);
```

**MySQL**: While MySQL does not natively support materialized views, similar functionality can be achieved through manual methods.
