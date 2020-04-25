
## 一、Prometheus 介绍

Prometheus 是一款开源时序数据监控系统，SoundCloud 2012年使用Go语言开发出来，该项目受到很多公司和组织的欢迎，于2016年加入CNCF（云计算本地计算基金会），
作为继Kubernetes之后的第二个托管项目。CNCF（Cloud Native Computing Foundation）于 2015 年 7 月成立，是一个非营利组织，为了统一云计算接口和相关标准而成立。

## 二、Influxdb 介绍
InfluxDB 是一款开源的时序数据库，是[InfluxData](https://www.influxdata.com/about/)（是一个公司，提供时序平台）下的一个产品，使用Go语言开发。

## 三、什么是时序数据库？
Time Series Database（TSDB）时序数据库，是自带时间戳索引的数据库，每一条数据都带了采样时间，适用于做监控图表，点击，交易图表（横轴都是时间）。

## 四、Prometheus 如何工作？

### 1 架构图
![架构图](https://prometheus.io/assets/architecture.png)

### 2 组件说明
- exporter：一个http api，负责采集暴露metrics。
- prometheus server：负责通过配置文件发现抓取目标，然后抓取目标metrics，加上时间戳存储到TSDB，因为自身也实现了TSDB，所以默认序列化后存储到本地磁盘。
- alertmanager：负责处理Prometheus发送的告警，默认Prometheus不处理告警，只是做规则判断，只要是触发就发送给alertmanager，
alertmanager负责分组、抑制、静默、多态处理渠道。
- grafana：数据可视化平台，可以使用PromQL，查询Prometheus数据源做绘图。

## 四、Prometheus 本地存储遇到的问题？

### 1 高可用
Prometheus的高可用很简单，复制一台一模一样的机器，配置一模一样，数据采集双份，存储双份到本地，然后用lb做负载查询，但是这样将会迎来新的问题，假如一台Prometheus故障，
10分钟没有采集到数据，那么用lb查询后端落到这台的时候也是没有数据的，因为Prometheus没有做到无状态，依赖本地数据。

### 2 水平扩容
如果Prometheus按照类型拆分，将抓取目标分为，基础指标和业务指标，再加上高可用就要部署两台基础Prometheus，两台业务Prometheus，这样又会迎来另外一个问题，数据分散在不同的地方，
没有一个全局的查询api。

### 3 联邦集群的问题
Prometheus fedoration联邦集群虽然是一台Prometheus可以抓取其他Prometheus的数据，但是两个相同的数据源，选择拉哪一个？拉lb的也会出现拉到坏的那一台。
采用fedoration如果写`match[]={__name__=".+"}`的话单次拉取的http body可能会非常大有可能导致拉取失败，然后又要把job拆分，分多次拉取，配置会越来越乱。


## 五、Prometheus 使用influxdb做存储

Prometheus 远程读写支持很多组件，[官方文档](https://prometheus.io/docs/operating/integrations/#remote-endpoints-and-storage)有详细说明

### 1 influxdb 准备工作

安装
```bash
wget https://dl.influxdata.com/influxdb/releases/influxdb-1.7.4.x86_64.rpm
sudo yum localinstall influxdb-1.7.4.x86_64.rpm
```
修改配置文件，启动

```ini
[meta]
  # Where the metadata/raft database is stored
  dir = "/data/influxdb/var/lib/influxdb/meta"
[data]
  # The directory where the TSM storage engine stores TSM files.
  dir = "/data/influxdb/var/lib/influxdb/data"
  wal-dir = "/data/influxdb/var/lib/influxdb/wal"
```
```bash
mkdir /data/influxdb
chown influxdb:influxdb /data/influxdb
systemctl start influxdb

```

创建数据库

```bash
# influx
Connected to http://localhost:8086 version 1.7.4
InfluxDB shell version: 1.7.4
Enter an InfluxQL query
> create database prometheus

```
### 2 修改Prometheus写入influxdb的Prometheus库

```yaml
remote_write:
  - url: "http://influxdb:8086/api/v1/prom/write?db=prometheus"
```
```bash
kill -1 pid
```
### 3 查看influxdb的Prometheus库
influxdb库下面是measurement，类似mysql的table

```bash
> show measurements
name: measurements
name
----
ALERTS
ALERTS_FOR_STATE
go_gc_duration_seconds
go_gc_duration_seconds_count
go_gc_duration_seconds_sum
go_goroutines
go_memstats_alloc_bytes
go_memstats_alloc_bytes_total
go_memstats_buck_hash_sys_bytes
go_memstats_frees_total
go_memstats_gc_sys_bytes
go_memstats_heap_alloc_bytes
go_memstats_heap_idle_bytes
go_memstats_heap_inuse_bytes
go_memstats_heap_objects
go_memstats_heap_released_bytes
go_memstats_heap_sys_bytes
go_memstats_last_gc_time_seconds
go_memstats_lookups_total
go_memstats_mallocs_total
go_memstats_mcache_inuse_bytes
go_memstats_mcache_sys_bytes
go_memstats_mspan_inuse_bytes
go_memstats_mspan_sys_bytes
go_memstats_next_gc_bytes
go_memstats_other_sys_bytes
go_memstats_stack_inuse_bytes
```
可以看到Prometheus里面的指标名被转换成measurement
```bash
> select * from go_gc_duration_seconds limit 1
name: go_gc_duration_seconds
time                __name__               cvm_id env group hostname            instance        ip         job monitor_group object qcloud_id    quantile replica role  value
----                --------               ------ --- ----- --------            --------        --         --- ------------- ------ ---------    -------- ------- ----  -----
1552111190174000000 go_gc_duration_seconds 613    dev xx    xx x.x.x.x:9100 x.x.x.x cvm empty         cvm    xxx-xxxxx 0        A       basic 0.000062953

```
prometheus的label，被转为influxdb的tags，值转为fields，value=000062953

一个例子
```conf
# Prometheus metric
example_metric{queue="0:http://example:8086/api/v1/prom/write?db=prometheus",le="0.005"} 308

# Same metric parsed into InfluxDB
measurement
  example_metric
tags
  queue = "0:http://example:8086/api/v1/prom/write?db=prometheus"
  le = "0.005"
  job = "prometheus"
  instance = "localhost:9090"
  __name__ = "example_metric"
fields
  value = 308
```
**配置了远程写的Prometheus，也会在本地存储一份**

## 相关资料

- [Prometheus Doc](https://prometheus.io/)
- [Prometheus Github](https://github.com/prometheus/prometheus)
- [Influxdb Doc](https://docs.influxdata.com/influxdb/v1.7/introduction/getting-started/)
- [Influxdb Github](https://github.com/influxdata/influxdb)
- [Prometheus endpoints support in InfluxDB](https://docs.influxdata.com/influxdb/v1.7/supported_protocols/prometheus)