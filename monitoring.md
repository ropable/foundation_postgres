# Monitoring

## Database monitoring

DB monitoring consists of capturing and recording db event. This info
helps a DBA to detect, identify and fix potential db performance issues.
Monitoring statistics make it easier for a DBA to monitori the health
and performance of DBs. Several monitoring tools are available.

## Database statistics

Database statistics catalog table store db activity information. This
information is captured by the **Stats Collector** process, and
includes:

* Current running sessions
* Running SQL
* Locks
* DML counts
* Row counts
* Index usage

The Stats Collector process collects and reports the information, and
some add some overhead to query execution. This process can be disabled
by changing the ``track_counts`` and ``track_activities`` parameters.
The stats collector uses temp files stored in subdir ``pg_stat_tmp``.
Permanent stats are stored in the ``pg_catalog`` schema in the global
subdirectory.

**Performance tip**: point the ``stats_temp_directory`` at a RAM-based
file system.

## DB statistics tables

* ``pg_catalog`` schema contains a set of tables, views and functions
  which store and report db stats.
* ``pg_class`` and ``pg_stats`` catalog tables store all statistics.
* ``pg_stat_database`` view can be used to view stats information about
  a database.
* ``pg_stat_user_tables`` shows info about activities on a table like
  inserts, update, deletes, vacuums, etc.
* ``pg_stat_user_indexes`` shows info about index usage for all user
  tables.

## Operating System processes

On Unix/Linux:

* ``ps``: info about current processes (eg. ``ps aux | grep postgres``)
* ``netstat``: info about current network connections (eg. ``netstat -an
  | grep LISTEN``)
* ``top`` or ``htop``: process list and system load

## Current sessions and locks

* ``pg_locks``: system table that stores info about outstanding locks in
  the lock manager.
* ``pg_stat_activity``: shows the process ID, database, user, query and
  start time of current query (eg. ``SELECT * FROM pg_stat_activity;``)
* Terminate a session gracefully using the ``pg_terminate_backend``
  function.
* Rollback a transaction gracefully using function ``pg_cancel_backend``.

Don't terminate a process using OS command ``kill``, as this can leave
the database in a corrupted state.

## Logging slow-running queries

``log_min_duration_statement`` parameter sets a minimum statement
execution time (ms) that causes a statement to be logged. This can be
used to log all SQL that take longer than e.g. 10s to be logged.

## Disk usage

Use ``pg_database_size(name)`` and ``pg_tablespace_size(name)`` to
display how much space is being used by a database or tablespace.

Example:

```sql
select datname, pg_size_pretty(pg_database_size(datname)) from pg_database;
```

## Tracking execution statistics

Track query execution stats using the **pg_stat_statements** extension.
The extension tracks stats across all databases of a cluster. The view
``pg_stat_statements`` shows the query statistics. Use
``pg_stat_statements_reset`` function to reset the statistics.

Set up:

1. Add ``pg_stat_statements`` to ``shared_preload_libraries`` parameter.
2. Configure: ``pg_stat_statements.max`` (default 5000),
  ``pg_stat_statements.track`` (top, all or none),
  ``pg_stat_statements.track_utility``, ``pg_stat_statements.save``.
3. Restart the database cluster.
4. Connect with a database and enable the ``pg_stat_statements``
   extension.

# Postgres Enterprise Manager (PEM)

PEM is an enterprise class software tool for administering, monitoring
and tuning PostgreSQL and EnterpriseDB Postgres Plus database servers.
PEM is architected to manage and monitor multiple servers from a single
console. Consists of PEM Agents that are installed on DB servers and send
information to the PEM Server, which is accessed via PEM Client (a web
application).
