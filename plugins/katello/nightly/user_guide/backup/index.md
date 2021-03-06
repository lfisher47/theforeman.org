---
layout: plugins/katello/documentation
title: Backup
version: nightly
---

# Backup

In the following sections, these assumptions are being made with respect to making the backup:

 * `/tmp/backup` will be used as the target for backups
 * All commands are executed as `root`

## Option One: Offline repositories backup

By default, the whole Katello instance will be turned off completely for the entire backup.

```
# katello-backup /tmp/backup
```

## Option Two: Online repositories backup

Backing up the repositories can take an extensive amount of time. You can perform a backup while online. In order for this procedure to succeed, you must not change or update the repositories database until the backup procedure is complete. Thus, you must avoid publishing, adding, or deleting content views, promoting content view versions, adding, changing, or deleting sync-plans, and adding, deleting, or syncing repositories during this time. To perform an online-backup of the repositories, run:

```
# katello-backup --online-backup /tmp/backup
```

## Option Three: Skip repositories backup

There may be situations in which you want to see a system without its repository information. You can skip backing up the Pulp database with the following option:

```
# katello-backup --skip-pulp /tmp/backup
```

Please note you would not be able to restore a Katello instance from a directory where the Pulp database was skipped.


## Option Four: Incremental backup

Incremental backups can be used to only store the changes since the last backup:

First take a full backup:
```
# katello-backup /tmp/backups
```
(This will create a new directory, /tmp/backups/katello-backup-YEAR-MONTH-DAYTHOUR:MIN:SEC[+-]TIMEZONE)

Take 1st incremental backup (this will create a new directory under /tmp/backups to house the new backup, just like the full backup directory):
```
# katello-backup /tmp/backups --incremental /tmp/backup/FULL_BACKUP_DIR
```

Take 2nd incremental backup (again, this will create a new directory under /tmp/backups to house this second incremental backup):
```
# katello-backup /tmp/backups --incremental /tmp/backup/FIRST_INCREMENTAL_BACKUP_DIR
```

### Branching/rebasing incremental backups

Should you choose to take a new incremental backup from, say, the full backup so you don't need too many files when restoring those backups, simply point the command to the full backup directory, and this newest backup directory will be incremental in relation to the full backup not to the 2nd incremental backup:
```
# katello-backup /tmp/backups --incremental /tmp/backup/FULL_BACKUP_DIR
```

## Final check-up

After a successful backup, the backup directory should have the following files:

```
# ls /tmp/backup
config_files.tar.gz
mongo_data.tar.gz
pgsql_data.tar.gz
```

Additionally, if you ran the backup without skipping the Pulp database, you will see the additional file:

```
pulp_data.tar
```

Katello instance should be up and running. Next chapter is dedicated to restoring a backup.

# Restore

## Full restore

All the following commands are executed under `root` system account.

Please note only backups that include the Pulp database can be restored. To verify that your backup directory is usable, make sure it has the following files:

```
# ls /tmp/backup
config_files.tar.gz
mongo_data.tar.gz
pgsql_data.tar.gz
pulp_data.tar
```

Once verified, simply run:

```
# katello-restore /tmp/backup
```

This command will require verification in order to proceed, as the method will destruct all databases before restoring them. Once the procedure is finished, all processes will be online, and all databases and system configuration will be reverted to the state and the time of the backup.

Check log files for errors, such as `/var/log/foreman/production.log` and `/var/log/messages`.

## Incremental restore

Incremental backups need to be restored sequentially starting with the oldest:

```
# katello-restore /tmp/backups/FULL_BACKUP
# katello-restore /tmp/backups/FIRST_INCREMENTAL
# katello-restore /tmp/backups/SECOND_INCREMENTAL
(...)
```
In case you have multiple "branches" of incremental backups, you need to pick your full backup and each incremental one for the "branch" you wish to restore, in the order they were taken.
