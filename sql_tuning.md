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
command is used to view a query plan.

Syntax: ``EXPLAIN [option] statement;``

Plan components:

* Cardinality (row estimates)
* Access method (sequential or index)
* Join method (hash, nested loop, etc.
* Join type & order
* Sort and aggregates

Example:

```sql
# Show a plan:
explain select * from edbstore.emp;
# Show the pages used by the planner:
select relpages, reltuples from pg_class where relname='emp';
# Show a cost:
show seq_page_cost;
```
