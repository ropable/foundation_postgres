# PostgreSQL Installation

Postgres runs as a daemon (Unix/Linux) or service (Windows). During
installation a ``postgres`` locked user will be created on Linux. It
is advisable to run Postgres under a separate user account, which
should only own the data directory.

Installation options include a wizard (Windows), OS package (deb, yum,
etc.), or from source.

Default installation locations (Debian/Ubuntu, installed via ``apt``):

* ``/usr/lib/postgresql/9.x/bin`` - shared program executables
* ``/var/lib/postgresql/9.x/`` - cluster data directories (e.g. main)
* ``/var/log/postgresql/`` - log files
* ``/etc/postgresql/9.x/`` - configuration files

## Environment variables

* ``PATH`` - should include the correct bin directory.
* ``PGDATA`` - should point to correct data cluster directory.
* ``PGPORT`` - should point to correct port on which db cluster is
  running (will define default port used by psql).
* ``PGUSER``
* ``PGHOST``

Environment variables on a client PC can be set in ``.profile`` or
``.bash_profile`` (Linux).
