# Tools

**psql** is the command line tool to interact with database clusters.
Will use the environment variables ``PGDATABASE``, ``PGHOST``,
``PGPORT`` and ``PGUSER`` first.

On startup, psql will execute commands in ``$HOME/.psqlrc` unless the
option -X is specified.

Options:

* -h <host>: will default to PGHOST or localhost
* -p <port>: will default to PGPORT or 5432
* -d <database_name>: will default to same as the logged-on user or
  ``PGUSER``
* -U <username>: will default to same as the username provided or
  ``PGDATABASE``
* -W: force password challenge.
* -f <filename>: execute the commands in filename.
* -c <command>: execute <command> and exit.

```bash
psql -h dbhostname -d databasename -U username  # Connects on port 5432
sudo -U postgres psql -p 5433  # Connect as postgres user to localhost:5433
```
# psql meta-commands

**psql** has a number of commands that all start with backslash (\). SQL
commands must always end with semicolon (;). Some commands accept a
pattern (modified regex). * and ? and wildcards, double-quotes are used
to specify an exact name/case.

Variables can be set with ``\set``. Some special variables also exist:

* AUTOCOMMIT
* ENCODING
* HISTFILE
* ON_ERROR_ROLLBACK
* ON_ERROR_STOP
* PROMPT1
* VERBOSITY

```sql
\?  # Help
\h [command]  # Shows info about syntax of a command
\h ALTER TABLE
\s  # Show command history
\s FILENAME  # Save the command history
\e  # Open the current query buffer in an editor then execute it
\w FILENAME  # Save the query buffer to file
\o FILENAME  # Start sending query output excl. STDERR to file (repeat to stop)
\set variable value  # Set variable to value
\echo :variable
\unset variable
\set AUTOCOMMIT off  # Set a special variable value
\conninfo  # Current connection info
\t  # Turn column headers off/on (tuples only)
\q  # Quit
\! [command]  # Execute an OS command on the command line

\l[+]  # List databases
\d(i|s|t|v|b|S)[+] [pattern]
\dt  # List database tables
\d+ table_name  # List detailed information about table_name
\di  # Info about Indexes
\ds  # Sequences
\dv  # Views
\dn  # Schemas (namespaces)
\df  # Functions
```
