# Postgres configuration

For a cluster, on Debian/Ubuntu ``/etc/postgresql/9.x/<clustername>``.

There are many config parameters available, these are case-insensitive.
All take one of five value types:

* boolean
* integer
* floating point
* string
* enum

# Server parameter file ``postgresql.conf``

Holds parameters used by a cluster. Some parameters only take effect on
server restart. ``#`` is used for comments, one parameter per line. Can
use the ``include`` directive to read from another file. We can also change
parameters using command line tools. Usually located in the cluster data
directory.

* Some params can be changed per session using ``SET``.
* Some params can be changed at user level using ``ALTER USER``.
* Some params can be changed at db level using ``ALTER DATABASE``.
* ``SHOW`` can be used to see settings.
* ``pg_settings`` and ``pg_file_settings`` catalog tables list settings.

``ALTER SYSTEM`` command is available in 9.4+, edits cluster settings
without changing ``postgresql.conf`` (writes to postgresql.auto.conf).
This is always read last on start/reload.

Hierarchy of settings: Session > User > Database > Cluster.

# Command examples

```sql
ALTER SYSTEM SET work_mem=20480;
ALTER SYSTEM SET work_mem=DEFAULT;
SHOW data_directory;
SHOW work_mem;
SELECT name, setting FROM pg_settings;
SELECT pg_reload_conf();
ALTER USER username SET work_mem=10240;
SHOW ALL;
```

# Connection settings

* ``listen_addresses`` - specifies the addresses on which the server is
  to listen for connections. Use * for all IP addresses.
* ``port`` (default 5432) - the port that the server listens on.
* ``max_connections`` (default 100) - max number of concurrent
  connections the server can support.
* ``superuser_reserved_connections`` (default 3) - number of connection
  slots reserved for superusers.
* ``unix_socket_directory`` (default /tmp) - directory to be used for
  Unix socket connections to the server.
* ``unix_socket_permissions`` (default 0777) - access permissions of the
  Unix socket.

# Security and authentication settings

* ``authentication_timeout`` (default 60 sec) - max time to complete
  client authentication, in seconds.
* ``ssl`` (default off) - enables SSL connections.
* ``ssl_ca_file`` - specifies the name of the file containing the SSL
  server certificate authority.
* ``ssl_cert_file`` - specifies the name of the file containing the SSL
  server certificate.
* ``ssl_key_file`` - specifies the name of the file containing the SSL
  server private key.
* ``ssl_ciphers`` - list of SSL ciphers that may be used for secure
  connections.

# Memory settings

* ``shared_buffers`` (default: 128MB) - size of Postgres shared buffer
  pool for a cluster.
* ``temp_buffers`` (default: 8MB) - amount of memory user by each
  backend for caching temporary table data.
* ``work_mem`` (default: 4MB) - amount of memory used for each sort of
  hash operation before switching to temporary disk files.
* ``maintenance_work_mem`` (default: 64MB) - amount of memory used for
  each index build of VACUUM.
* ``temp_file_limit`` (default -1) - amount of disk that a session can
  use for temp files. A transaction attempting to exceed this limit can
  be cancelled. Default is unlimited space.

# Query planner settings

* ``random_page_cost`` (default 4.0) - estimated cost of a random page
  fetch in cost units. May need to reduce to account for caching.
* ``seq_page_cost`` (default 1.0) - estimate cost of a sequential page
  fetch in cost units. May need to be reduced to account for caching.
  Must always set ``random_page_cost`` > ``seq_page_cost``.
* ``effective_cache_size`` (default 4GB) - used to estimate the cost of
  an index scan. Rule of thumb is 75% of system memory.

There are many ``enable_*`` parameters which influence the planner in
choosing an optimal plan.

# Write Ahead Log (WAL) settings

* ``wall_level`` (default minimal) - how much info is written to the
  WAL. Change this to enable replication. Other values: archive, logical
  and hot_standby.
* ``fsync`` (default: on) - turn off to make database faster, however
  may cause arbitrary corruption in case of system crash.
* ``wal_buffers`` (default -1, autotune) - amount of memory used in
  shared memory for WAL data. Default of -1 equals 3% of shared_buffers.
* ``min_wal_size`` (default 80MB) - WAL size to start recycling the WAL
  files.
* ``max_wal_size`` (default 1GB) - WAL size to start checkpoint.
* ``checkpoint_timeout`` (default 5 min)
* ``wal_compression`` (default off)

# Logging

* ``log_destination``
* ``logging_collector`` (default off) - enables advanced logging features.
  Allows the following to be set.
* ``log_directory``
* ``log_filename``
* ``log_file_mode``
* ``log_rotation_age``
* ``log_rotation_size``

When to log:

* ``client_min_messages`` (default NOTICE) - messages this severity and
  above are sent to the client.
* ``log_min_messages`` (default WARNING) - messages of this severity and
  above are sent to the server.
* ``log_min_error_statement`` (default ERROR) - when a message of this
  severity or higher is logged, the statement that caused it is also
  logged.
* ``log_min_duration_statement`` (default -1, disabled) - when a
  statement runs for at least this long (ms) is written to the server
  log with its duration. Allows long-running statements to be analysed.

What to log:

* ``log_connections`` (default off) - log successful connections to the
  server.
* ``log_disconnections`` (default off) - log info each time a session
  disconnections, including duration.
* ``log_error_verbosity``
* ``log_duration`` (default off)
* ``log_line_prefix`` - additional details to log with each line.
* ``log_statement`` (default none) - can be: none, ddl, mod, all.
* ``log_temp_files`` (default -1) - log temporary files of this size or
  larger in kilobytes.
* ``log_checkpoints`` (default off) - causes checkpoints and restart
  points to be logged.

# Background writer settings

* ``bgwriter_delay`` (default 200 ms) - specifies time between activity
  rounds for the background writer.
* ``bgwriter_lru_maxpages`` (default 100) - maximum no or pages that the
  background writer may clean per activity round.
* ``bgwriter_lru_multiplier`` (default 2.0)

A primary tuning technique is to lower **bgwriter_delay**.

# Statement behaviour

* ``statement_timeout`` - abort any statement that takes > the specified
  no of milliseconds.
* ``default_tablespace`` - name of the tablespace in which objects are
  created by default.
* ``temp_tablespace``
* ``search_path`` - specifies the order in which schemas are searched.

# Vacuum cost settings

* ``vacuum_cost_delay`` (default 0 ms) - length of time that the process
  will wait when the cost limit is exceeded.
* ``vacuum_cost_limit`` (default 200) - the accumulated cost that will
  cause the vacuuming process to sleep.
* ``vacuum_cost_page_hist`` (def 1)
* ``vacuum_cost_page_miss`` (def 10)
* ``vacuum_cost_page_dirty`` (def 200)

# Autovacuum settings

* ``autovacuum`` (def on) - control whether the autovacuum launcher runs
  and starts worker processes to vacuum & analyse tables.
* ``autovacuum_max_workers`` (def 3) - max no of processes to run.
* ``autovacuum_work_mem`` (def -1) - max amt of memory used by each
  worker.
* ``log_autovacuum_min_duration`` (def -1) - autovacuum tasks longer
  than this duration in ms are logged.

# Include directives

The configuration file can include directives from other files, allows
config to be divided in separate files. Usage:

* include 'filename'
* include_dir 'path/to/dir'
