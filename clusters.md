# Database clusters

A database cluster is a collection of databases managed by a single
server instance. Clusters are comprised of a data dir and a port on
which it is listening for requests.

# Debian/Ubuntu commands

```bash
# List database clusters
pg_lsclusters
# Create and start a database cluster (defaults to next free port)
pg_createcluster 9.5 <clustername> --start
# Control a cluster
pg_ctlcluster 9.5 <clustername> start|stop|restart|reload|status|promote
# Stop a cluster (can use -f option to force shutdown)
pg_ctlcluster 9.5 <clustername> stop
# Stop and delete a cluster
pg_dropcluster --stop 9.5 <clustername>
```

# Other Linux OS commands

```bash
# Create a cluster
initdb -D /path/to/data/dir  # Defaults to PGDATA if not provided.
# Choose a unique port in postgresql.conf
# Start a cluster
pg_ctl start -D /path/to/data/dir
# Reload a cluster
pg_ctl reload -D /path/to/data/dir
```
