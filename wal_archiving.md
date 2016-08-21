# WAL Archiving

## Transaction logging (WAL)

WAL Writer writes at each commit. WAL protects the database
(durability). WAL logs are stored as a set of 16 MB segments. The process of
switching from one WAL segment to another is called a _log switch_.

## WAL Archiving

The **Archiver** is an optional background process that invokes the
``archive_command`` when ``archive_mode`` is set to **on**. It copies the
WAL segment when full and ready to be switched. Archive logs are copies
of WAL segments. The name of the archive log is the same as the WAL
segment, and are used for PITR and replication.

Configured in two ways: Archiver process, or ``pg_receivexlog`` tool
and streamed to the prepared location.

## Archiver process

This process can be started by changing the WAL level to ``archive``,
and setting the ``archive_mode`` parameter to on. The process executes
``archive_command`` whenever:

* A WAL segment is switched.
* Checkpoint is forced.
* ``archive_timeout`` seconds pass.
* ``pg_switch_xlog()`` function is called.

A checkpoint is forced if ``max_wal_size`` (default 1 GB) is to be
exceeded, ``checkpoint_timeout`` seconds pass or whenever the
``CHECKPOINT`` command is executed. The ``archive_command`` can be
anything, but will typically copy the logfile to a location.

## ``pg_receivexlog``

``pg_receivexlog`` streams WAL segments from a running Postgres cluster.
It connects with a cluster using streaming replication protocol and
streams transactions to an archive location. Streamed files are PITR
ready. WAL segments can be streamed in real time and this can be used to
avoid log shipping delays.

It runs using the ``pg_receivexlog`` command, options:

* -D: archive directory
* -v: verbose mode
* -?: help
* -d: database name
* -h: hostname
* -p: port
* -u: user

``pg_receivexlog`` needs superuser access (or a user with REPLICATION
permissions). ``max_wal_senders`` must be set to 1 or more. The default
interval check is 10 seconds and can be changed with **-s** option.

``pg_hba.conf`` must be explicitly configured to permit to replication
connection. This can be done by adding a ``replication`` entry in
``pg_hba.conf`` and reloading the cluster.

The ``pg_stat_archiver`` view can be used to display statistics and
information about the archiving process.
