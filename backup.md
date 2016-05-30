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

# Stop database cluster: ``pg_ctlclust 9.5 clustername stop``
# Copy the /var/lib/postgresql/9.5/<clustername> directory to another
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
