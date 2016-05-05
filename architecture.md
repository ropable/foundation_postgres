# Architecture

Postgres uses processes, not threads. One backend process per user
session. The ``postmaster`` process acts as supervisor, monitoring
utility processes.

![Process and memory architecture](media/images/postgres_process_memory_architecture.jpg)
