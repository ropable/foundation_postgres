# Architecture

Postgres uses processes, not threads. One backend process per user
session. The ``postmaster`` process acts as supervisor, monitoring
utility processes.

Postmaster is master process called ``postgres``, listens on 1 only TCP
port (receives client connection requests). Default port: 5432. The
postmaster must be configured to accept a connection from an IP
address.

![Process and memory architecture](media/images/postgres_process_memory_architecture.jpg)

## Utility processes

* Background writer - writes dirty data blocks to disk.
* WAL writer - flushes write ahead log to disk.
* Checkpointer process - auto. performs a checkpoint based on config
  parameters.
* Autovacuum launcher - starts Autovacuum workers as needed.
* Autovacuum workers - recovers free space for reuse.
* Logging collector - routes log messages to syslog, eventlog or files.
* Stats collector - collects usage statistics by relation and block.
* Archiver - archives write-ahead log files.
