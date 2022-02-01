# HSA L20: Backups

## Overview
This is an example project to show how to set up a backup system.

### Task:
* Implement all kinds of repository models: Full, Incremental, Differential, Reverse Delta, CDP.
* Compare their parameters: size, ability to roll back at a specific time point, speed of roll back, cost.


## Getting Started

### Preparation

1. Prepare sample [omdb](https://github.com/credativ/omdb-postgresql) database. Run next script to download data.
```bash
  ./docker/db_dump/download
```
2. Run the docker containers.
```bash
  docker-compose up -d
```

Be sure to use ```docker-compose down -v``` to clean up after you're done with tests.

3. Check if `ppgbackrest` is working correctly.
```bash
docker exec -it fid_backup pgbackrest --stanza=main --log-level-console=info check
```
if you have an error, run the next command and check again:
```bash
docker exec -it fid_backup pgbackrest --stanza=main stanza-create
```

## Backup Models

### Full
```bash
$ docker exec -it fid_backup pgbackrest --stanza=main --type=full --log-level-console=info backup

stanza: main
    status: ok
    cipher: none

    db (current)
        wal archive min/max (14): 00000001000000000000001A/00000001000000000000001E

        full backup: 20220201-173616F
            timestamp start/stop: 2022-02-01 17:36:16 / 2022-02-01 17:36:53
            wal start/stop: 00000001000000000000001E / 00000001000000000000001E
            database size: 367.9MB, database backup size: 367.9MB
            repo1: backup set size: 91.4MB, backup size: 91.4MB
```

### Incremental
```bash
$ docker exec -it fid_backup pgbackrest --stanza=main --type=incr --log-level-console=info backup

stanza: main
    status: ok
    cipher: none

    db (current)
        wal archive min/max (14): 00000001000000000000001A/00000001000000000000001D
        ...
        incr backup: 20220201-182001F_20220201-191308I
            timestamp start/stop: 2022-02-01 19:13:08 / 2022-02-01 19:13:13
            wal start/stop: 00000001000000000000001D / 00000001000000000000001D
            database size: 368.0MB, database backup size: 24.3KB
            repo1: backup set size: 91.4MB, backup size: 530B
            backup reference list: 20220201-182001F
```

### Differential
```bash
$ docker exec -it fid_backup pgbackrest --stanza=main --type=diff --log-level-console=info backup

stanza: main
    status: ok
    cipher: none

    db (current)
        wal archive min/max (14): 00000001000000000000001A/00000001000000000000001D
        ...
        diff backup: 20220201-193403F_20220201-194159D
            timestamp start/stop: 2022-02-01 19:41:59 / 2022-02-01 19:42:29
            wal start/stop: 000000010000000000000043 / 000000010000000000000043
            database size: 701.9MB, database backup size: 340.6MB
            repo1: backup set size: 178.6MB, backup size: 88.2MB
            backup reference list: 20220201-193403F
```

### Reverse Delta
The PostgreSQL + Wal-G playground was taken from the [https://github.com/stephane-klein/playground-postgresql-walg]
Build playground:
```bash
$ cd docker/reverse_delta
$ ./scripts/build-postgres-with-wal-g-docker-image.sh
$ docker-compose up -d postgres1 s3
$ ./scripts/pg1/load-seed.sh
$ ./scripts/pg1/insert-fixtures.sh
$ ./scripts/pg1/query.sh
```

Create and check backup:
```bash
$ ./scripts/pg1/make-basebackup.sh
$ ./scripts/pg1/show-pg-stats-archiver.sh

select * from pg_stat_archiver;
-[ RECORD 1 ]------+------------------------------
archived_count     | 14
last_archived_wal  | 00000001000000000000000E
last_archived_time | 2022-02-01 20:53:45.085259+00
failed_count       | 0
last_failed_wal    | 
last_failed_time   | 
stats_reset        | 2022-02-01 20:47:54.590468+00
```

### CDP
Backup:
```bash

```

## Comparing All Of Them

|  | Full | Incremental | Differential | Reverse Delta | CDP |
|---|:---:|:---:|:---:|:---:|:---:|
| Size | 91.4MB | 1.2MB | 88.2MB | 102.9MB |  |
| Point in specific time recovery | - | + | + | + | + |
| Speed of roll back | 38.76s | 5.23s | 30.86s | 31.54s |  |
