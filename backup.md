# Backups

There are three different approaches to backing up Postgres data: SQL
dump, file system backup and continuous archiving.

# SQL dump

Generate a text file using ``pg_dump``. This does not block reading &
writing, and dumps are internally consistent for the time of the dump.

Format is ``pg_dump <options> <database>``. Options:

* -a: data only
* -s: structure
* -n <schema>: specific schema(s) only
* -t <table>: specific table(s)
* -f <path>: send dump to file path
* -Fc: dump in compressed custom format
* -v: verbose output

To reduce large outputs, you can compress or split the output. E.g.:

```
pg_dump dbname | gzip > filename.gz
pg_dump dbname | split -b 100m - filename
```

If the db dump is output in plain text, it can be read in by ``psql``.
For example:

```
psql dbname < infile
```

Use ``pg_restore`` to read in a database dump of compressed custom
output format and restore it. Format is ``pg_restore [options]
<filename>``. Options:

* -d <database name>: connect to the specified database.
* -C: create the database named in the dump and restore into it.
* -a: restore data only, not the schema.
* -s: restore schema only, not the data.
* -n <schema>: restore only objects from specified schema.
* -t <table>: restore only the specified table.
* -v: verbose output.

``pg_dumpall`` is used to dump an entire database cluster in plain text
SQL format. Dumps all global objects: users, groups and permissions. Use
``pgsql`` to restore the dump.

```
pg_dumpall [options] > <filename>
```

* -a: data only (no schema)
* -s: schema only
* -g: global objects only, not databases
* -r: roles only
* -c: clean (drop) databases before recreating
* -O: skip restoration of object ownership
* -x: do not dump privileges
* --disable-triggers: disable triggers during dump process
* -v: verbose output

# File system backups

An alternative backup strategy is the copy the files on the server (the
data directory). The database must be shut down in order to obtain a
usable backup. File backups only work for complete backup and
restoration of an entire database cluster. File system snapshots can be
used for live servers. Example file system backup:

1. Stop database cluster: ``pg_ctlclust 9.5 clustername stop``
2. Copy the /var/lib/postgresql/9.5/<clustername> directory to another
   location (e.g. tar -cvf backup.tar /var/lib/postgresql/9.5/<clustername>)


# Continuous archiving

Postgres maintains WAL files for all transactions in the **pg_xlog**
directory. Postgres automatically maintains the WAL logs that are full
and switches them out. Continuous archiving can be set up to keep a copy
of switched WAL logs which can be later used for recovery. This also
enables online file system backup of a database cluster. Requirements:

* ``wal_level`` must be set to archive
* ``archive_mode`` must be set to on
* ``archive_command`` must be set in postgresql.conf

**pg_receivexlog** streams transaction logs from a running Postgres
cluster. The tool uses streaming replication protocol to stream
transaction logs to a local directory. Example:

```
pg_receivelog -h localhost -D /mnt/archivedir
```

# Base backup using ``pg_basebackup``

``pg_basebackup`` supports both plain and tar output formats. Example:

```
pg_basebackup -h localhost -D /mnt/archive
```

# Point-in-time recovery

PITR is the ability to restore a database cluster up to the present or
to a specified point of time in the past. Uses a full database cluster
backup and the write-ahead logs in the ``pg_xlog`` subdirectory. Must be
configured before it is needed (write-ahead log archiving must be
enabled). Steps:

1. Stop the server.
2. Copy the data directory and transaction logs.
3. Remove all directories and files from the cluster data dir.
4. Restore the db files from your file system backup.
5. Verify the ownership of restored backup directories (must not be root
   user).
6. Remove any files present in ``pg_xlog``.
7. If you have unarchived WAL segment files recovered from crashed
   cluster, copy them into ``pg_xlog``.
8. Create a recovery command file **recovery.conf** in the cluster data
   directory.
9. Start the server.
10. Upon completion, the server will rename recovery.conf to recovery.done.

```recovery.conf``` file:

```
# Unix:
restore_command = 'cp /mnt/archive/%f "%p"'
recovery_target_name
recovery_target_time
recovery_target_xid
recovery_target_inclusive
recovery_target_action  # Can be pause (default), promote or shutdown
```
