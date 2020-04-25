
## 一、Prometheus 时间序列介绍

time series 时间序列，由指标名(metric name)和标签组成(key value pairs， labels)

比如
```
<metric name>{<label name>=<label value>, ...}
api_http_requests_total{method="POST", handler="/messages"}
```

标签代表指标维度，通过不同组合描述不同的场景，常用做筛选条件，标签值一般为可枚举的字符串。

一个存入Prometheus里面的时间序列的标签由用户配置的抓取目标labels和exporter暴露的合并而成。

比如下面的一个Prometheus序列瞬时值

```
api_http_requests_total{method="POST", handler="/messages",app_name="service_name",env="prd",instance="1.1.1.1:9999"} 10
```
`method`，`handler` 是exporter暴露的

`app_name`,`env` 是targets labels配置的

`instance`是怎么来的？

## 二、标签是如何被抓取合并的

一个抓取配置是怎么配置的

```yaml
job_name: <job_name>
[ scrape_interval: <duration> | default = <global_config.scrape_interval> ]
[ scrape_timeout: <duration> | default = <global_config.scrape_timeout> ]
[ metrics_path: <path> | default = /metrics ]
[ honor_labels: <boolean> | default = false ]
[ scheme: <scheme> | default = http ]
params:
  [ <string>: [<string>, ...] ]
basic_auth:
  [ username: <string> ]
  [ password: <secret> ]
  [ password_file: <string> ]
[ bearer_token: <secret> ]
[ bearer_token_file: /path/to/bearer/token/file ]
tls_config:
  [ <tls_config> ]
[ proxy_url: <string> ]
azure_sd_configs:
  [ - <azure_sd_config> ... ]
consul_sd_configs:
  [ - <consul_sd_config> ... ]
dns_sd_configs:
  [ - <dns_sd_config> ... ]
ec2_sd_configs:
  [ - <ec2_sd_config> ... ]
openstack_sd_configs:
  [ - <openstack_sd_config> ... ]
file_sd_configs:
  [ - <file_sd_config> ... ]
gce_sd_configs:
  [ - <gce_sd_config> ... ]
kubernetes_sd_configs:
  [ - <kubernetes_sd_config> ... ]
marathon_sd_configs:
  [ - <marathon_sd_config> ... ]
nerve_sd_configs:
  [ - <nerve_sd_config> ... ]
serverset_sd_configs:
  [ - <serverset_sd_config> ... ]
triton_sd_configs:
  [ - <triton_sd_config> ... ]
static_configs:
  [ - <static_config> ... ]
relabel_configs:
  [ - <relabel_config> ... ]
metric_relabel_configs:
  [ - <relabel_config> ... ]
[ sample_limit: <int> | default = 0 ]
```

Prometheus提供多种监控目标配置,一般情况我们使用静态配置或者一种发现配置就可以了


比如使用文件发现，动态添加删除虚拟机监控
```yaml
  - job_name: 'cvm'
    file_sd_configs:
      - files:
        - "targets/cvm/*.yml"
```

dev.yml

```yaml
- targets: ['10.0.0.1:9100']
  labels:
    cvm_id: "2"
    env: dev
    hostname: cvm-test1
    ip: 10.0.0.1
- targets: ['10.0.0.2:9100']
  labels:
    cvm_id: "1"
    env: dev
    hostname: cvm-test2
    ip: 10.0.0.2
```
Prometheus加载了配置文件之后并不是直接去http请求targets的地址抓取的而是经过了一些列的转换

1 加载targets配置

2 一个抓取目标被转为类似如下结构

```json
{
    "__address__": "10.0.0.1:9100",
    "__param_配置文件的<params key>": "配置文件的<params value>"(默认无),
    "__metrics_path":"配置文件的metrics_path(默认/metrics",
    "__schema__":"配置文件的schema(默认http)",
    "__meta_xxxx":"某种发现机制的元数据，文件发现就是__meta_filepath",
    "job":"配置文件的job_name", //这里不太确定是在这里加还是存储的时候加
}
```
3 `relabel_config`,标签重写

用户修改上面的label
```json
{
    "__address__": "10.0.0.1:9100",
    "__param_配置文件的<params key>": "配置文件的<params value>"(默认无),
    "__metrics_path":"配置文件的metrics_path(默认/metrics",
    "__schema__":"配置文件的schema(默认http)",
    "__meta_xxxx":"某种发现机制的元数据，文件发现就是__meta_filepath",
    "job":"配置文件的job_name", //这里不太确定是在这里加还是存储的时候加
    "instance": "__address__"（如果不存在）,
}
```
`__address__` 不存在就会删除抓取目标


4 生成http抓取目标 `__schema__://__address____metrics_path?<params key>=<params value>&<params key>=<params value>`

5 开始抓取http请求 如果失败添加up指标 = 0

6 判断targets labels是否和exporer暴露的重复

7 如有重复   判断 `honor_labels` ? 以exporter为准 : exporter的标签加上前缀 exported_

8 没有重复 合并targets labels和exporter的lables，去掉__前缀开头的

9 `metric_relabel_config`,指标标签重写

10 添加up =1和 scrape_duration_seconds 新指标

## 三、relabel_config配置


- [Life of a Label](https://www.robustperception.io/life-of-a-label)