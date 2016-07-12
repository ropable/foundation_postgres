# Performance benchmarking

Benchmarking measures the performance of a database and helps DBA in
preparing a system. It can evaluate changes and identify problem areas.

``pgbench`` is installed with Postgres. *BenchmarkSQL* is an OS tool
that can be used, another is *HammerDB*. It connects multiple DB
sessions and runs the same SQL multiple times, then calculates the
average transaction rate. Workload involves fice SELECT, UPDATE and
INSERT commands per transaction.

Invoke ``pgbench`` with ``-i`` to create the required tables in a
database. Other options:

* -c: # clients (default 1).
* -C: Establish a new connection for each transaction.
* -t: # of transactions that each client runs (default 10).
* -j: # of threads (default 1).

Example: ``pgbench -p 5444 -c 10 -T 30``
