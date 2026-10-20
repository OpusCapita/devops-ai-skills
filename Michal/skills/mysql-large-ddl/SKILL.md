---
name: mysql-large-ddl
description: Plan and monitor schema changes (ALTER TABLE / ADD COLUMN / migrations) on large MySQL 5.7 InnoDB tables in production, and judge whether an in-flight ALTER is progressing or stuck. Use when a migration runs long, disk fills during DDL, or someone asks "is the ALTER stuck?".
---

# Large-table DDL on MySQL 5.7

## Before running
- MySQL 5.7 has no `ALGORITHM=INSTANT`. `ADD COLUMN` on a big table is a full rebuild and needs about 1x the table's `.ibd` size free on the disk that holds the table data.
- Check `df -h` on the **table-data mount** for at least 1.5x the target `.ibd` size. The datadir, tmpdir, binlogs and table data can be on different disks. Purging binlogs does not free the table-data disk if they're on another mount.
- For big tables, prefer `gh-ost` or `pt-online-schema-change` off-hours over a release-gating ORM migration (e.g. Sequelize `addColumn`).
- Some apps track applied migrations in their own table, not `SequelizeMeta`. Check which table the app actually uses.

## Is the running ALTER alive?
A temp file `#sql-ib*.ibd` whose size has stopped growing, or a flat `df`, does **not** mean it's stuck. Check these instead:
- `stat -c %Y <tempfile>`: if mtime keeps moving while the size stays the same, it's writing into pre-allocated extents (index build or row-log apply).
- `SHOW FULL PROCESSLIST` → `State`: `altering table` means it's working. `Waiting for table metadata lock` means it's blocked, and it's blocking others.
- `ls -l /proc/<mysqld pid>/fd | grep '#sql'`: if the fd is still held, the DDL thread is alive and the file isn't an orphan.
- On the app side, grep pods for `lock wait timeout` / `metadata lock` to prove nothing is queued behind it.

Watch for minutes before you call a trend. Take timestamps from the observed host (`date -u`), not your workstation.
