# Upgrading best practices

Postgres uses a minor and major version upgrade system. For minor
versions (0.0.x) storage format will not change and executable can be
replaced with existing data directory. For major versions (0.x), the
storage format may change and upgrade is more complicated. Upgrade can
be formed using ``pg_dumpall`` or ``pg_upgrade``.

Postgres is an actively developed OSS database. Major releases include
new features, capabilities and performance enhancements. Minor releases
include security and data corruption bug fixes. Not upgrading is riskier
than upgrading. Postgres community aims to fully support a major release
for five years.

Plan an upgrade: plan, upgrade, test, maintain. Read the release notes
and manual. Check for user-visible incompatibilities. Test an
application on the new version before completing the switch. Plan for
failure. Take a full backup before and after upgrading. Retain the old
database until the upgrading database is verified as correct and working
correctly.

``pg_upgrade`` can be used for 8.4 or later to upgrade a database to the
latest version. Upgrade steps:

1. Verify the source version and data directory.
2. Install newer version (same server or new server).
3. Initialise a new data cluster (not started).
4. Stop the source database cluster.
5. Run ``pg_upgrade``.
6. Start the new data cluster.
7. Connect and verify the new data cluster.

For streaming replication environments, the standby server also needs to
be upgraded to the same new version.
