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
use the include directive to read from another file. We can also change
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
```
