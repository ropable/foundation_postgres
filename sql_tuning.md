# SQL Tuning

Lifecycle of a SQL query: parse, optimise (note for DDL), execute. The
parser checks syntax and tokenises the query. The optimiser generates a
plan (cost-based). The executer carries out the plan. Common performance
issues:

* Full table scans
* Bad SQL
* Sorts using disk
* Join order
* Old/missing statistics
* I/O issues
* Bad connection management

SQL tuning goals:

* Identity slow or bad SQL
* Find possible performance issue
* Reduce total execution time
* Reduce the resource usage of a query
* Determine the most efficient execution plan (optimiser hints, etc.)
* Balance/parallelise the workload

SQL tuning steps (no fixed order):

1. Identify slow queries.
2. Review the query execution plan.
3. Optimiser statistics and behaviour.
4. Restructure SQL statements.
5. Add/remove indexes.
6. Review final execution plan.

# Identify slow queries

``log_min_duration_statement`` parameter can be used to track long
running SQL. PEM's Log Manager Wizard can configure log min duration
statements, also has the Server Log Analysis Dashboard, Log Analysis
Expert Wizard and SQL Profiler (to find, troubleshoot and optimise slow
SQL queries). ``log_min_duration_statement`` is set in postgres.conf,
and all queries that take longer that the defined number of milliseconds
will be logged.

# Review the query execution plan

An execution plan shows the steps necessary to execute a SQL statement.
Planner is responsible for generating the plan. The Optimizer determines
the most efficient plan. Cost estimates rely on accurate table
statistics, gathered with ANALYZE (plus other parameters). The EXPLAIN
command is used to view a query plan, while ANALYZE will execute the
query also and display some statistics about the execution. Syntax:

    EXPLAIN [ANALYZE] <SQL statement>;

Plan components:

* Cardinality (row estimates)
* Access method (sequential or index)
* Join method (hash, nested loop, etc.
* Join type & order
* Sort and aggregates

The ``relpages`` and ``reltuples`` from the ``pg_class`` table are used
by the planner to generate the plan. The param ``seq_page_cost`` is the
cost for scanning a page sequentially, the param ``cpu_tuple_cost`` is
the cost for scanning a tuple. Plan cost for scanning a table is:
(number of pages * seq_page_cost) + (number of tuples * cpu_tuple_cost).
Choice of query plans are only as good as table statistics.

Example:

```sql
# Show a plan:
EXPLAIN SELECT * FROM edbstore.emp;
# Show the pages & tuples used by a table:
SELECT relpages, reltuples FROM pg_class WHERE relname='customer';
# Show cost params:
SHOW seq_page_cost;
SHOW cpu_tuple_cost;
```

Reviewing explain plans:

* Examine the various costs at different levels in the plan.
* Look for seq scans on large tables (all seq scans are not
  inefficient).
* Check whether join types are appropriate for the number of rows
  returned.
* Check for indexes used in the plan.
* Examine the cost of sorting and aggregation (in-memory/disc IO).
* Review if views are being used efficiently.

# Optimizer statistics and behaviour

Table stats are critical to ensure optimal query plans. They are not
updated in realtime, so should be manually updated after bulk
operations. Can be updated using ``ANALYZE`` or OS command ``vacuumdb``
with the -z option. Stats are stored in ``pg_class`` and
``pg_statistics``.

Syntax: ``ANALYZE [VERBOSE] <table_name>|<column_name>;``

Statistics can be controlled using ``ALTER TABLE name ALTER COLUMN column
SET STATISTICS x`` (where **x** is 1 < x < 10000). A higher
number will signal the server to gather and update more stats, but may
slow autovacuum and analyze operations on stat tables.

The ``TABLESAMPLE`` clause can be used to retrieve a random sample of
tables from a table. Supports SYSTEM and BERNOULLI sample methods.
System uses random IO, is faster but sampling will vary more wildly.
Bernoulli used seq IO, scans full table picking tuples randomly. Example:

    SELECT * FROM pg_class TABLESAMPLE SYSTEM(20);

# Restructuring SQL statements

* Avoid implicit type conversions.
* Avoid expressions as the optimizer may ignore indexes on such columns.
* Use equijoins wherever possible to improve SQL efficiency.
* Try changing the access path and join order with hints.
* Avoid full table scans if using an index is more efficient.
* Try disabling sequential scans with ``SET enable_seqscan TO off;``
* Join order can have a significant effect on performance.
* Use views/materialized views for complex queries.

# Indexes

Create indexes when needed, remove unused indexes. Adding an index for
one query can slow other queries. Verify index usage with the EXPLAIN
command. Different types include B-tree (default), Hash, GiST, GIN, BRIN
(PG 9.5+).

You can adjust the ordering of a B-tree index by including the options
ASC, DESC, NULLS FIRST and/or NULLS LAST. This may save time on sorting.

Indexes can also be used to enforce uniqueness (on B-tree). Postgres
automatically creates a unique index when a unique constraint or primary
key is defined for a table. Example:

    CREATE UNIQUE INDEX name ON table (column1, [, column2]);

An index can be created on a computed value from the table column
values. These are expensive to maintain. They are useful when retrieval
speed is more important than insert or update speed. Example:

    CREATE INDEX name ON table (lower(column1));

A partial index can be built on a subset of table rows. The index
contains entries only for those rows that satisfy the predicate.
Example:

    CREATE INDEX index ON table(column1) WHERE column1='VALUE';

BRIN (Block Range Index) store metadata on range (min and max) of pages.
Block range are by default 1 MB and ``pages_per_range`` storage
parameter determines the size of the block range. This index type is
small in size, easier to maintain and remove disk IO. Designed to handle
large table columns, which correlate to the physical location within the
table.

Examining index usage: it is difficult to form a general procedure for
determining which indexes to create. Always run ANALYZE first. Use real
data for experimentation. When indexes are not used, it can be useful for
testing to force their usage with hints.

# Review the final plan

* Check again for missing indexes.
* Check table stats are correct.
* Check for large table sequential scans.
* Compare the cost of first and final plans.
