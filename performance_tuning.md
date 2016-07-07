# Performance tuning

These are activities used to optimise a database, to correct
poorly-written SQL, poor session management, misconfigured db
parameters, I/O issues, etc. Starts with information collection, then
analysis, followed by implementation of solution(s).

Identify the info relevant to diagnosing problems, collect that info on
a regular basis, conduct analysis to understand and correlate all
relevant stats together. There may be multiple solutions. Judgement is
req'd to prioritise and quantify the solutions by impact.

PEM simplifies collection of performance data, with automatic analysis
of performance and diagnostics data. Provides a performance dashboard to
view I/O, memory usage, session activity and wait stats. Includes SQL
Profiler to optimise slow SQL, Index Advisor and setup alerts and
thresholds.

# Operating System considerations

OS issues:

* Memory - increased memory demand can lead to 100% usage and disk
  swapping. Check usage. Reduce usage or increase system RAM.
* CPU - may be the bottleneck when load or process wait is high. Check
  usage for the db connections.
* Disk - high wait times or request rates are symptoms of an I/O
  problem. Check for I/O spikes. Solution is to reduce demand or
  increase capacity.
* ``vmstat``, ``sar`` and ``iostat`` can also be useful.

Hardware: focus on disk speed and RAM over CPU speed (Postgres is IO
intensive like many databases). Separate the transaction log and
indexes: put the transaction log (pg_xlog) on a dedicated disk resource,
tablespaces can be used to create indexes on separate drives.

The file system makes a difference, a journaling file system is not
required for the trans log. Multiple options for Linux:

* EXT2 with sync enabled
* EXT3 with write-back
* EXT4 and XFS

Recommended: RAID controllers with battery-backed cache and configured
for write-back.

# Server parameter tuning

Default *postgresql.conf* server parameters are configured for wide
compatibility, must be tuned to optimal values for best performance.
Parameters can be evaluated using difference methods. Basic info needed
for tuning: database size, largest table size, type and frequency of
queries, available RAM, no of concurrent connections. PEM has Postgres
Expert which can perform and analysis for your server and database.

``max_connections`` - sets the max no of concurrent connections. Each
user connection has a backend process on the server. Terminated when a
user logs off. Connection pooling can decrease the overhead on
postmaster by reusing existing user backend processes.

``shared_buffers`` - sets the number of shared memory buffers used by
the db server. Each buffer is 8KB. Minimum value must be 16 and at least
twice ``max_connections``. Default is 128MB. 6-25% of available memory
is a good general guideline.

``work_mem`` - amount of memory in KB to be used by internal sorts and
hash tables before switching to temporary disk files. Min allowed value
is 64KB. Set in KB, default in 4MB. Increasing the work_mem often helps in
faster sorting (can also be changed on a per-session basis).

``maintenance_work_mem`` - max memory in KB to be used in maintenance
operations such as vacuum, create index, alter table (add FK). Minimum
allowed value is 1024 KB. Set in KB. Performance for vacuuming and
restoring db dumps can be improved by increasing this value.

``autovacuum_work_mem`` - max amount of memory to be used by each
autovacuum worker process. Default value is -1, which indicates that
``maintenance_work_mem`` is to be used instead.

``huge_pages`` - enables.disables the use of huge memory pages. Valid
values are try, on and off. Param is only supported on Linux. May help
in increasing performance by using smaller page tables and less CPU time
on memory management.

``effective_cache_size`` - size of memory available for disk cache for a
single query. A higher value favours index scans. This param is used
only for estimation purposes. Half of RAM in a conservative setting, 3/4
is more aggressive. Find the optimal value looking at OS stats after
changing this parameter.

``temp_file_limit`` - max amount of disk space that a session can use
for temporary files. A transaction attempting to exceed this will be
cancelled. Default is -1 (no limit). This settings constrains the total
space used at any instant by all temporary files used by a given
session.

``wal_level`` - WAL level determines how much information is written to
the WA. The default value is minimal, which writes only the info needed
to recover from a crash or shutdown. The value archive adds logging
required for WAL archiving. The value host_standby adds information
required to run read-only queries on a standby server.

``wal_buffers`` - number of disk-page buffers allocated in shared memory
for WAL data. Each buffer is 8KB. Needs to be only large enough to hold
the amount of WAL data created by a typical transaction. Default setting
is -1 (auto-tuned).

Checkpoints and ``max_wal_size`` - checkpoints writes the current
in-memory modified pages (known as dirty pages) to disk. An automatic
checkpoint occurs each time the ``max_wal_size`` is reached. Each log
file is 16 MB. A larger settings results in fewer checkpoints.

``checkpoint_timeout`` is the max time between automatic WAL checkpoints
in seconds before a checkpoint is forced. Range is 30-3600 seconds
(default is 300s).

``fsync`` - ensures that all the WAL buffers are written to the WAL logs
at each COMMIT. Turning this off will boost performance, but there is a
risk of data corruption - DON'T DO IT!!! Can be turned off during
initial loading of a new db cluster load from a backup (bulk load). Note
that ``synchronous_commit=off`` can provide similar benefits for
noncritical transactions without risk of data corruption.
