# TimescaleDB简介

TimescaleDB是一个建立在PostgreSQL之上的时间序列关系型数据库。

## 时间序列数据的特征

TimescaleDB这样的时间序列数据库通常利用了一些重要的特征:

以时间为中心:数据记录总是有一个时间字段。

仅追加:数据几乎完全是仅追加(INSERTs)。

最近的:新数据通常是关于最近的时间间隔的，我们很少更新或回填关于旧时间间隔的缺失数据。

TimescaleDB官方文档地址:https://docs.timescale.com/
