# Maintenance

Data files become fragmented as data is modified/deleted. Maintenance
should be proactive. Improves performance and saves db from transaction
ID wraparound failures. An update/delete of a row does not immediately
remove the row from the disk page. Eventually this row space becomes
obsolete and causes fragmentation/bloat.

Maintenance commands: **ANALYZE, VACUUM, CLUSTER**. Maintenance command
``vacuumdb`` can be run from the command prompt.

Optimizer statistics play a vital role in query planning, but are not
updated in real time. Collect information for relations inluding size,
row counts, average row size and row sampling. Stats are stored
permanently in catalog tables. The maintenance command ``ANALYZE``
updates the stats (e.g. after a bulk operation). To update:

```sql
ANALYZE tablename;
```

# Routine vacuuming

Obsolete rows can be removed/reused using vacuuming. Helps to shrink
data file size when required. Vacuuming can be automated using
autovacuum. Vacuum can recover disk space, updates statistics, updates
the visibility map (for index-only scans). Vacuuming also prevents
transaction ID wraparound failure. Row transaction IDs are 32-bit numbers,
maximum of ~4 billion, behaviour is to recycle IDs when maximum is
reached (wraparound).

The visibility map simply stores one bit per heap page.

By default, autovacuuming is enable for databases (``postgres.conf``).
The behaviour of autovacuum workers can be altered. Per-table settings
can be set using ``ALTER TABLE`` (e.g. autovacuum_enabled).

* ``VACUUM`` - removes dead rows and makes space available for reuse.
* ``VACUUM FULL`` - more agressive algorithm, compacts table, takes
  more time, requires extra space during operation. Requires an
  exclusive lock on each table.

```sql
# Show the size of a table:
SELECT pg_size_pretty(pg_relation_size('tablename'));
# Show autovacuum parameters:
SELECT name, settings FROM pg_settings WHERE name LIKE 'autovacuum%';
```

The ``VACUUM`` command has a command line executable called
``vacuumdb``. Can vacuum all databases easily, e.g.:

```
sudo -u postgres vacuumdb -p 5433 --all --full
```

# Routine reindexing

Indexes are used for faster database access and query planning. Indexes
are stored on data pages and become fragmented over time. ``REINDEX``
rebuilds an index using current data in the table. Syntax:

```sql
REINDEX {INDEX|TABLE|DATABASE|SYSTEM} name [FORCE];
```

# Clustering

Clustering sorts data physically according to the specified index.
Requires an ``ACCESS EXCLUSIVE`` lock on the database. Clustering lowers
disk accesses and speeds up queries when accessing a range of indexes
values. Recommended to set ``maintenance_work_mem`` parameter to a large
value before running ``CLUSTER``, and always run ``ANALYZE`` afterwards.

Example:

```sql
CLUSTER tablename USING indexname;
```
