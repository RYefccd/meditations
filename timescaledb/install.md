# TimescaleDB配置安装

> 本次安装配置环境为Ubuntu

Ubuntu的默认软件包存储库中没有TimescaleDB，因此需要从TimescaleDB Personal Packages Archive(PPA)安装。

添加Timescale的APT存储库，通过按ENTER键来确认此操作

```
sudo add-apt-repository ppa:timescale/timescaledb-ppa
```

刷新APT缓存以更新软件包列表

```
sudo apt-get update
```

本示例中使用的是使用PostgreSQL版本11，如果使用的是其他版本的PostgreSQL(例如12)，请替换以下命令中的值并运行它
```
sudo apt install timescaledb-2-postgresql-11
```

使用timescaledb-tune工具来启动配置向导，-conf-path 后面的参数为数据库实例正在使用的配置文件路径
```
sudo timescaledb-tune -conf-path /data/timescaledb_test/postgresql.conf
```
输入命令后会提示将timescaledb模块添加到shared_preload_libraries列表中，输入y确认进行调整
```
Using postgresql.conf at this path:
/data/timescaledb_test/postgresql.conf

Writing backup to:
/tmp/timescaledb_tune.backup202202091014

shared_preload_libraries needs to be updated
Current:
#shared_preload_libraries = ''
Recommended:
shared_preload_libraries = 'timescaledb'
Is this okay? [(y)es/(n)o]:
```
timescaledb-tune会根据服务器的特性和PostgreSQL版本来提示调整信息，输入y确认进行调整
```
Tune memory/parallelism/WAL and other settings? [(y)es/(n)o]: y
Recommendations based on 7.80 GB of available memory and 2 CPUs for PostgreSQL 11

Memory settings recommendations
Current:
shared_buffers = 128MB
#effective_cache_size = 4GB
#maintenance_work_mem = 64MB
#work_mem = 4MB
Recommended:
shared_buffers = 1995MB
effective_cache_size = 5987MB
maintenance_work_mem = 1021883kB
work_mem = 10218kB
Is this okay? [(y)es/(s)kip/(q)uit]: y
success: memory settings will be updated
```
timescaledb-tune会根据服务器的cpu来给出并行相关的设置建议，输入y确认进行调整
```
Parallelism settings recommendations
Current:
missing: timescaledb.max_background_workers
#max_worker_processes = 8
#max_parallel_workers_per_gather = 2
#max_parallel_workers = 8
Recommended:
timescaledb.max_background_workers = 8
max_worker_processes = 13
max_parallel_workers_per_gather = 1
max_parallel_workers = 2
Is this okay? [(y)es/(s)kip/(q)uit]: y
success: parallelism settings will be updated
```
timescaledb-tune会给出预写日志(WAL)相关的设置建议，输入y确认进行调整
```
WAL settings recommendations
Current:
#wal_buffers = -1
min_wal_size = 80MB
Recommended:
wal_buffers = 16MB
min_wal_size = 512MB
Is this okay? [(y)es/(s)kip/(q)uit]: y
success: WAL settings will be updated
```
timescaledb-tune会给出一些其他设置建议来提高使用性能，输入y确认进行调整
```
Miscellaneous settings recommendations
Current:
#default_statistics_target = 100
#random_page_cost = 4.0
#checkpoint_completion_target = 0.5
#max_locks_per_transaction = 64
#autovacuum_max_workers = 3
#autovacuum_naptime = 1min
#effective_io_concurrency = 1
Recommended:
default_statistics_target = 500
random_page_cost = 1.1
checkpoint_completion_target = 0.9
max_locks_per_transaction = 64
autovacuum_max_workers = 10
autovacuum_naptime = 10
effective_io_concurrency = 200
Is this okay? [(y)es/(s)kip/(q)uit]: y
success: miscellaneous settings will be updated
Saving changes to: /data/timescaledb_test/postgresql.conf
```
重启数据库实例来使修改后的配置参数生效
```
sudo -i -u postgres /usr/lib/postgresql/11/bin/pg_ctl -D /data/timescaledb_test/ restart
```
登录数据库实例，添加TimescaleDB扩展
```
psql -U postgres -p 5434
CREATE EXTENSION IF NOT EXISTS timescaledb;
```
将会出现以下输出
```
2022-02-09 10:24:38.002 CST [3685] CONTEXT:  PL/pgSQL function inline_code_block line 12 at RAISE
WARNING:  
WELCOME TO
 _____ _                               _     ____________  
|_   _(_)                             | |    |  _  \ ___ \ 
  | |  _ _ __ ___   ___  ___  ___ __ _| | ___| | | | |_/ / 
  | | | |  _ ` _ \ / _ \/ __|/ __/ _` | |/ _ \ | | | ___ \ 
  | | | | | | | | |  __/\__ \ (_| (_| | |  __/ |/ /| |_/ /
  |_| |_|_| |_| |_|\___||___/\___\__,_|_|\___|___/ \____/
               Running version 2.3.1
For more information on TimescaleDB, please visit the following links:

 1. Getting started: https://docs.timescale.com/timescaledb/latest/getting-started
 2. API reference documentation: https://docs.timescale.com/api/latest
 3. How TimescaleDB is designed: https://docs.timescale.com/timescaledb/latest/overview/core-concepts

Note: TimescaleDB collects anonymous reports to better understand and assist our users.
For more information and how to disable, please see our docs https://docs.timescale.com/timescaledb/latest/how-to-guides/configuration/telemetry.

CREATE EXTENSION
```
可以使用\dx命令来检查TimescaleDB扩展是否安装成功
```
postgres=# \dx
                                      List of installed extensions
    Name     | Version |   Schema   |                            Description                            
-------------+---------+------------+-------------------------------------------------------------------
 plpgsql     | 1.0     | pg_catalog | PL/pgSQL procedural language
 timescaledb | 2.3.1   | public     | Enables scalable inserts and complex queries for time-series data
(2 rows)
```
**至此，TimescaleDB安装配置已全部完成。**