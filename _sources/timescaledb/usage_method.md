# TimescaleDB使用

>本示例是以线上3.0秒级别数据统计业务诉求为例

创建hypertable需要两个步骤。首先使用CREATE TABLE语句创建一个常规关系表，
然后使用SELECT语句和create hypertable函数将表转换为超表。
SELECT语句需要转换的表的名称，以及表中时间列的名称。
```
create table captcha_detail_log(
iso_time  TIMESTAMPTZ     NOT NULL,
captcha_id    varchar(256)    NOT NULL,
method    varchar(128),
type    varchar(128),
result    varchar(128),
drag_count    smallint
);

select create_hypertable('captcha_detail_log', 'iso_time', , if_not_exists => TRUE);
```
查看hypertable元信息
```
SELECT * FROM timescaledb_information.hypertables WHERE hypertable_name = 'captcha_detail_log';
```
查看hypertable维度的信息
```
SELECT * from timescaledb_information.dimensions ORDER BY hypertable_name, dimension_number;
```
根据实际的业务诉求来设置块的大小，块的大小按照时间间隔来区分。
```
SELECT set_chunk_time_interval('captcha_detail_log',600000000);
SELECT set_chunk_time_interval('captcha_detail_log',INTERVAL '10 minutes');
```
查看与hypertable关联的块的信息
```
SELECT show_chunks('captcha_detail_log');
SELECT show_chunks('captcha_detail_log', older_than => INTERVAL '3 hours');
SELECT show_chunks('captcha_detail_log', newer_than => INTERVAL '3 hours');
SELECT show_chunks('captcha_detail_log', older_than => DATE '2022-02-09');
```
查看hypertable块的元信息
```
SELECT * FROM timescaledb_information.chunks WHERE hypertable_name = 'captcha_detail_log';
```
删除指定时间范围内的数据快
```
SELECT drop_chunks('captcha_detail_log', INTERVAL '3 hours');
SELECT drop_chunks('captcha_detail_log', newer_than => now() + interval '3 hours');
SELECT drop_chunks('captcha_detail_log', '2022-02-08'::date);
SELECT drop_chunks('captcha_detail_log', older_than => INTERVAL '3 hours', newer_than => INTERVAL '4 hours')
```
hypertable开启压缩并设置压缩选项
```
ALTER TABLE captcha_detail_log SET (timescaledb.compress, timescaledb.compress_segmentby = 'captcha_id', timescaledb.compress_orderby = 'iso_time');
```
hypertable关闭压缩
```
ALTER TABLE captcha_detail_log SET (timescaledb.compress=false);
```
查看hypertable压缩相关的设置信息
```
SELECT * FROM timescaledb_information.compression_settings WHERE hypertable_name = 'captcha_detail_log';
```
参数说明
:::{note}
|  Name  | Type  |  Description  |
|  ----  | ----  |  ----  |
| timescaledb.compress  |  BOOLEAN  |  开启或关闭压缩  |
| timescaledb.compress_orderby  | TEXT | 压缩使用的顺序，与SELECT查询中的Order by子句指定的方式相同。默认值是hypertable的时间列的降序。 |
| timescaledb.compress_segmentby  | TEXT | 表示数据来源的标识符，例如device_id或tags_id通常是一个很好的选择。默认为无segment by列。 |  
:::

查看所有hypertable策略和策略状态
```
select * from timescaledb_information.jobs;
select * from timescaledb_information.job_stats;
```
更改策略触发计划
```
SELECT alter_job(1047, scheduled => false);
SELECT alter_job(1047, schedule_interval => INTERVAL '2 hours');
SELECT alter_job(1047, next_start => '2022-02-10 09:00:00.0+00');
```
参数说明
:::{note}
|  Name  | Type  |  Description  |
|  ----  | ----  |  ----  |
| schedule_interval  |  INTERVAL  |  策略运行的时间间隔，默认为24小时。  |
| max_runtime  | INTERVAL | 在策略停止之前，后台工作调度器允许作业运行的最长时间。 |
| max_retries  | INTEGER | 如果策略运行失败，重试的次数。 |  
| retry_period  | INTERVAL | 在失败时，调度程序在策略重试之间等待的时间。 |  
| scheduled  | BOOLEAN | 如果设置为FALSE，则不允许该策略作为后台策略运行。 |  
| config  | JSONB | 特定于策略的配置。 |  
| next_start  | TIMESTAMPTZ | 下次运行策略的时间。可以通过将此值设置为无穷大来暂停策略，然后使用now()的值重新启动。 |  
| if_exists  | BOOLEAN | 如果任务不存在，设置为true则发出一个通知而不是一个错误。默认值为false。。 |  
:::

设置hypertable压缩策略
```
SELECT add_compression_policy('captcha_detail_log', INTERVAL '10 minutes', if_not_exists => true);
```
删除hypertable压缩策略
```
SELECT remove_compression_policy('captcha_detail_log');
```
压缩指定的块
```
SELECT compress_chunk('_timescaledb_internal._hyper_1_2_chunk');
```
解压缩指定的块
```
SELECT decompress_chunk('_timescaledb_internal._hyper_1_2_chunk');
```
查看hypertable压缩相关的指定块的统计信息
```
SELECT * FROM chunk_compression_stats('captcha_detail_log');
SELECT chunk_name, compression_status, pg_size_pretty(before_compression_total_bytes) as before_total, pg_size_pretty(after_compression_total_bytes) as after_total, node_name FROM chunk_compression_stats('captcha_detail_log') order by chunk_name;
```
查看hypertable的块所使用的磁盘空间的信息
```
SELECT * FROM chunks_detailed_size('captcha_detail_log') ORDER BY chunk_name, node_name;
SELECT chunk_schema, chunk_name, pg_size_pretty(table_bytes) as table_bytes, pg_size_pretty(index_bytes) as index_bytes, pg_size_pretty(toast_bytes) as toast_bytes, pg_size_pretty(total_bytes) as total_bytes FROM chunks_detailed_size('captcha_detail_log') ORDER BY chunk_name, node_name;
```
查看hypertable中指定索引的大小
```
SELECT hypertable_index_size('captcha_detail_log_iso_time_idx');
SELECT pg_size_pretty(hypertable_index_size('captcha_detail_log_iso_time_idx')) as hypertable_index_size;
```
查看hypertable压缩相关的统计信息
```
SELECT * FROM hypertable_compression_stats('captcha_detail_log');
SELECT total_chunks, number_compressed_chunks, pg_size_pretty(before_compression_table_bytes) as before_compression_table_bytes, pg_size_pretty(before_compression_index_bytes) as before_compression_index_bytes, pg_size_pretty(before_compression_toast_bytes) as before_compression_toast_bytes, pg_size_pretty(before_compression_total_bytes) as before_compression_total_bytes, pg_size_pretty(after_compression_table_bytes) as after_compression_table_bytes, pg_size_pretty(after_compression_index_bytes) as after_compression_index_bytes, pg_size_pretty(after_compression_toast_bytes) as after_compression_toast_bytes, pg_size_pretty(after_compression_total_bytes) as after_compression_total_bytes, node_name FROM hypertable_compression_stats('captcha_detail_log');
```
查看hypertable使用的磁盘空间的详细信息
```
SELECT * FROM hypertable_detailed_size('captcha_detail_log');
SELECT total_chunks, number_compressed_chunks, pg_size_pretty(before_compression_table_bytes) as before_compression_table_bytes, pg_size_pretty(before_compression_index_bytes) as before_compression_index_bytes, pg_size_pretty(before_compression_toast_bytes) as before_compression_toast_bytes, pg_size_pretty(before_compression_total_bytes) as before_compression_total_bytes, pg_size_pretty(after_compression_table_bytes) as after_compression_table_bytes, pg_size_pretty(after_compression_index_bytes) as after_compression_index_bytes, pg_size_pretty(after_compression_toast_bytes) as after_compression_toast_bytes, pg_size_pretty(after_compression_total_bytes) as after_compression_total_bytes, node_name FROM hypertable_compression_stats('captcha_detail_log');
```

创建连续聚合视图
```
CREATE MATERIALIZED VIEW captcha_detail_second
WITH (timescaledb.continuous) AS
SELECT time_bucket(INTERVAL '1 second', iso_time) AS second,
       captcha_id,
       method,
       type,
       result,
       count(*)
FROM captcha_detail_log where not (method = 'Ajax' and type like '%_slide' and drag_count < 4 and result = 'fail')
GROUP BY 1, 2, 3, 4, 5 with no data;
```
:::{admonition} Tip
:class: tip
如果没有给出with no data，连续聚合视图将自动刷新
:::

查看连续聚合视图的元信息
```
SELECT * FROM timescaledb_information.continuous_aggregates;
```

修改连续聚合视图
```
连续聚合禁用实时聚合
ALTER MATERIALIZED VIEW captcha_detail_second SET (timescaledb.materialized_only = true);
```
ALTER MATERIALIZED VIEW 其他用法
* RENAME TO  重命名连续聚合视图
* SET SCHEMA  设置连续聚合视图新的SCHEMA
* SET TABLESPACE  将连续聚合视图的实体化移动到新的表空间
* OWNER TO  设置连续聚合视图新的所有者

删除连续聚合视图
```
DROP MATERIALIZED VIEW captcha_detail_second;
```
创建自动刷新连续聚合的策略
```
SELECT add_continuous_aggregate_policy('captcha_detail_second',
    start_offset => INTERVAL '1 minutes',
    end_offset => INTERVAL '1 second',
    schedule_interval => INTERVAL '3 seconds');
```
手动刷新刷新连续聚合
```
CALL refresh_continuous_aggregate('captcha_detail_second', '2022-02-09', '2020-02-10');
```
删除连续聚合的刷新策略
```
SELECT remove_continuous_aggregate_policy('captcha_detail_second');
```
创建自动清除块数据的策略
```
SELECT add_retention_policy('captcha_detail_log', INTERVAL '2 days');
```
删除自动清除块数据的策略
```
SELECT remove_retention_policy('captcha_detail_log');
```