# telegraf-influxdb-grafana
利用 telegraf  influxdb grafana 搭建服务器监控平台


# 一、架构思想
要搭建服务器监控平台，总共分三步：
第一步，数据采集；
第二步，数据存储；
第三步，数据可视化。
作者经过大量阅读相关资料后，最终选择了Telegraf+InfluxDB+Grafana这套方案。接下来，作者就对这套监控系统方案进行简要的介绍。
![在这里插入图片描述](https://img-blog.csdnimg.cn/aa124365831a479a90bc29e87ef50dcd.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAZ21IYXBweQ==,size_20,color_FFFFFF,t_70,g_se,x_16)

# 二、组件介绍
## 2.1 Telegraf 数据采集部分
`Telegraf`是一个用`Go`语言编写的代理程序，可采集系统和服务的统计数据，并写入`InfluxDB`数据库。`Telegraf`具有内存占用小的特点，通过插件系统开发人员可轻松添加支持其他服务的扩展。目前，最新版`Telegraf`支持的插件主要有：

- Apache
- DNS query time
- Docker
- http Listener
- MySQL
- Network Response
- Tomcat
- Zookeeper
- TCP Listener
## 2.2 InfluxDB 数据存储部分
`InfluxDB` 是一个开源分布式时序、事件和指标数据库。使用 `Go` 语言编写，无需外部依赖。其设计目标是实现分布式和水平伸缩扩展。

>InfluxDB有以下特性：

- Time Series （时间序列）：可以使用与时间有关的相关函数（如最大，最小，求和等）

- Metrics（度量）：你可以实时对大量数据进行计算

-  Eevents（事件）：它支持任意的事件数据

 

>InfluxDB有以下特特点：

- Schemaless（无结构），可以是任意数量的列

- Scalable（可扩展）：min, max, sum, count, mean, median 一系列函数，方便统计

- Native HTTP API, 内置http支持，使用http读写

- Powerful Query Language 类似sql

- 自带压力测试工具等，功能强大

 

## 2.3 Grafana 数据展示部分

`Grafana` 是实现 数据展示（数据可视化） 的工具。 `Grafana`是一个跨平台的开源的度量分析和可视化工具，可以通过将采集的数据查询然后可视化的展示，并及时通知。

>Grafana有以下特点：

- 展示方式：快速灵活的客户端图表，面板插件有许多不同方式的可视化指标和日志，官方库中具有丰富的仪表盘插件，比如热图、折线图、图表等多种展示方式；

- 数据源：Graphite，InfluxDB，OpenTSDB，Prometheus，Elasticsearch，CloudWatch和KairosDB等；

- 通知提醒：以可视方式定义最重要指标的警报规则，Grafana将不断计算并发送通知，在数据达到阈值时通过Slack、PagerDuty等获得通知；

- 混合展示：在同一图表中混合使用不同的数据源，可以基于每个查询指定数据源，甚至自定义数据源；

- 注释：使用来自不同数据源的丰富事件注释图表，将鼠标悬停在事件上会显示完整的事件元数据和标记；

- 过滤器：Ad-hoc过滤器允许动态创建新的键/值过滤器，这些过滤器会自动应用于使用该数据源的所有查询

# 三、组件安装
## 3.1 部署influxdb grafana
> influxdb grafana 部署本文基于docker-compose

docker-compose.yml

```yml
version: '3'

services:

  influxdb:
    image: influxdb:1.8
    container_name: influxdb
    networks:
      influxdb:
    volumes:
      - influxdb-storage:/var/lib/influxdb
    restart: unless-stopped

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    depends_on:
      - influxdb
    ports:
      - 3000:3000
    networks:
      - influxdb
    environment:
      - GF_INSTALL_PLUGINS=grafana-piechart-panel
    volumes:
      - grafana-storage:/var/lib/grafana
    restart: unless-stopped

volumes:
  grafana-storage:
  influxdb-storage:

networks:
  influxdb:
```
grafana docker版本官方部署文档：

[https://grafana.com/docs/grafana/latest/installation/docker/](https://grafana.com/docs/grafana/latest/installation/docker/)

[https://grafana.com/docs/grafana/latest/administration/configure-docker/](https://grafana.com/docs/grafana/latest/administration/configure-docker/)

## 3.2 influxdb 创建数据库并开启认证
1. 进入influxdb

```
docker exec -it influxdb /bin/bash
influx
```

2.  展示用户

```
show users
```

3.  创建用户

```
CREATE USER "username" WITH PASSWORD 'password'
```

4.  创建管理员权限的用户

```
CREATE USER "username" WITH PASSWORD 'password' WITH ALL PRIVILEGES
```
> CREATE USER "telegraf" WITH PASSWORD 'telegraf' WITH ALL PRIVILEGES
5. 删除用户

```
DROP USER "username"
```

6.  展示数据库、

```
show databases
```

7.  新建数据库

```
create database "dbname"
```
> create database telegraf_metrics
> 
![在这里插入图片描述](https://img-blog.csdnimg.cn/0b2fec6bc16446e4a64c2a98280abab2.png)

8. 查看influxdb当前状态信息

```
influxd
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/0e897d1655464478901f2bd57b2ff913.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAZ21IYXBweQ==,size_20,color_FFFFFF,t_70,g_se,x_16)

9. 查看influxdb目前配置

```
influxd config
```

```config
root@e97f0d91ee06:/# influxd config
Merging with configuration at: /etc/influxdb/influxdb.conf
reporting-disabled = false
bind-address = "127.0.0.1:8088"

[meta]
  dir = "/var/lib/influxdb/meta"
  retention-autocreate = true
  logging-enabled = true

[data]
  dir = "/var/lib/influxdb/data"
  index-version = "inmem"
  wal-dir = "/var/lib/influxdb/wal"
  wal-fsync-delay = "0s"
  validate-keys = false
  strict-error-handling = false
  query-log-enabled = true
  cache-max-memory-size = 1073741824
  cache-snapshot-memory-size = 26214400
  cache-snapshot-write-cold-duration = "10m0s"
  compact-full-write-cold-duration = "4h0m0s"
  compact-throughput = 50331648
  compact-throughput-burst = 50331648
  max-series-per-database = 1000000
  max-values-per-tag = 100000
  max-concurrent-compactions = 0
  max-index-log-file-size = 1048576
  series-id-set-cache-size = 100
  series-file-max-concurrent-snapshot-compactions = 0
  trace-logging-enabled = false
  tsm-use-madv-willneed = false

[coordinator]
  write-timeout = "10s"
  max-concurrent-queries = 0
  query-timeout = "0s"
  log-queries-after = "0s"
  max-select-point = 0
  max-select-series = 0
  max-select-buckets = 0

[retention]
  enabled = true
  check-interval = "30m0s"

[shard-precreation]
  enabled = true
  check-interval = "10m0s"
  advance-period = "30m0s"

[monitor]
  store-enabled = true
  store-database = "_internal"
  store-interval = "10s"

[subscriber]
  enabled = true
  http-timeout = "30s"
  insecure-skip-verify = false
  ca-certs = ""
  write-concurrency = 40
  write-buffer-size = 1000

[http]
  enabled = true
  bind-address = ":8086"
  auth-enabled = false
  log-enabled = true
  suppress-write-log = false
  write-tracing = false
  flux-enabled = false
  flux-log-enabled = false
  pprof-enabled = true
  pprof-auth-enabled = false
  debug-pprof-enabled = false
  ping-auth-enabled = false
  prom-read-auth-enabled = false
  https-enabled = false
  https-certificate = "/etc/ssl/influxdb.pem"
  https-private-key = ""
  max-row-limit = 0
  max-connection-limit = 0
  shared-secret = ""
  realm = "InfluxDB"
  unix-socket-enabled = false
  unix-socket-permissions = "0777"
  bind-socket = "/var/run/influxdb.sock"
  max-body-size = 25000000
  access-log-path = ""
  max-concurrent-write-limit = 0
  max-enqueued-write-limit = 0
  enqueued-write-timeout = 30000000000

[logging]
  format = "auto"
  level = "info"
  suppress-logo = false

[[graphite]]
  enabled = false
  bind-address = ":2003"
  database = "graphite"
  retention-policy = ""
  protocol = "tcp"
  batch-size = 5000
  batch-pending = 10
  batch-timeout = "1s"
  consistency-level = "one"
  separator = "."
  udp-read-buffer = 0

[[collectd]]
  enabled = false
  bind-address = ":25826"
  database = "collectd"
  retention-policy = ""
  batch-size = 5000
  batch-pending = 10
  batch-timeout = "10s"
  read-buffer = 0
  typesdb = "/usr/share/collectd/types.db"
  security-level = "none"
  auth-file = "/etc/collectd/auth_file"
  parse-multivalue-plugin = "split"

[[opentsdb]]
  enabled = false
  bind-address = ":4242"
  database = "opentsdb"
  retention-policy = ""
  consistency-level = "one"
  tls-enabled = false
  certificate = "/etc/ssl/influxdb.pem"
  batch-size = 1000
  batch-pending = 5
  batch-timeout = "1s"
  log-point-errors = true

[[udp]]
  enabled = false
  bind-address = ":8089"
  database = "udp"
  retention-policy = ""
  batch-size = 5000
  batch-pending = 10
  read-buffer = 0
  batch-timeout = "1s"
  precision = ""

[continuous_queries]
  log-enabled = true
  enabled = true
  query-stats-enabled = false
  run-interval = "1s"

[tls]
  min-version = ""
  max-version = ""

root@e97f0d91ee06:/# 

```
10. InfluxDB可视化工具

InfluxDB在0.13版本以后，就默认关闭了web管理页面

下载地址：[https://github.com/CymaticLabs/InfluxDBStudio/releases/tag/v0.2.0-beta.1](https://github.com/CymaticLabs/InfluxDBStudio/releases/tag/v0.2.0-beta.1)


## 3.3 部署Telegraf
`Telegraf`下载地址：[https://portal.influxdata.com/downloads/](https://portal.influxdata.com/downloads/)

`Telegraf`配置参照：[https://grafana.com/grafana/dashboards/928](https://grafana.com/grafana/dashboards/928)


新建 `telegraf.conf`
```config
# Global tags can be specified here in key="value" format.
[global_tags]
  # dc = "us-east-1" # will tag all metrics with dc=us-east-1
  # rack = "1a"
  ## Environment variables can be used as tags, and throughout the config file
  # user = "$USER"


# Configuration for telegraf agent
[agent]
  interval = "10s"
  round_interval = true
  metric_batch_size = 1000
  metric_buffer_limit = 10000
  collection_jitter = "0s"
  flush_interval = "10s"
  flush_jitter = "0s"
  precision = ""
  debug = false
  quiet = false
  hostname = ""
  omit_hostname = false


### OUTPUT

# Configuration for influxdb server to send metrics to
[[outputs.influxdb]]
  urls = ["http://192.168.3.105:8086"]
  database = "telegraf_metrics"

  ## Retention policy to write to. Empty string writes to the default rp.
  retention_policy = ""
  ## Write consistency (clusters only), can be: "any", "one", "quorum", "all"
  write_consistency = "any"

  ## Write timeout (for the InfluxDB client), formatted as a string.
  ## If not provided, will default to 5s. 0s means no timeout (not recommended).
  timeout = "5s"
  username = "grafana"
  password = "grafana"
  ## Set the user agent for HTTP POSTs (can be useful for log differentiation)
  # user_agent = "telegraf"
  ## Set UDP payload size, defaults to InfluxDB UDP Client default (512 bytes)
  # udp_payload = 512


# Read metrics about cpu usage
[[inputs.cpu]]
  ## Whether to report per-cpu stats or not
  percpu = true
  ## Whether to report total system cpu stats or not
  totalcpu = true
  ## Comment this line if you want the raw CPU time metrics
  fielddrop = ["time_*"]


# Read metrics about disk usage by mount point
[[inputs.disk]]
  ## By default, telegraf gather stats for all mountpoints.
  ## Setting mountpoints will restrict the stats to the specified mountpoints.
  # mount_points = ["/"]

  ## Ignore some mountpoints by filesystem type. For example (dev)tmpfs (usually
  ## present on /run, /var/run, /dev/shm or /dev).
  ignore_fs = ["tmpfs", "devtmpfs"]


# Read metrics about disk IO by device
[[inputs.diskio]]
  ## By default, telegraf will gather stats for all devices including
  ## disk partitions.
  ## Setting devices will restrict the stats to the specified devices.
  # devices = ["sda", "sdb"]
  ## Uncomment the following line if you need disk serial numbers.
  # skip_serial_number = false


# Get kernel statistics from /proc/stat
[[inputs.kernel]]
  # no configuration


# Read metrics about memory usage
[[inputs.mem]]
  # no configuration


# Get the number of processes and group them by status
[[inputs.processes]]
  # no configuration


# Read metrics about swap memory usage
[[inputs.swap]]
  # no configuration


# Read metrics about system load & uptime
[[inputs.system]]
  # no configuration

# Read metrics about network interface usage
[[inputs.net]]
  # collect data only about specific interfaces
  # interfaces = ["eth0"]


[[inputs.netstat]]
  # no configuration

[[inputs.interrupts]]
  # no configuration

[[inputs.linux_sysctl_fs]]
  # no configuration
```
>注意对以下配置进行修改：

```config
urls = ["http://192.168.3.105:8086"]
database = "telegraf_metrics"
username = "grafana"
password = "grafana"
```
执行程序 `telegraf --config telegraf.conf`

以后台方式启动 `nohup telegraf --config telegraf.conf > /dev/null 2>&1 &`

利用可视化工具查看`telegraf`采集回的数据
![在这里插入图片描述](https://img-blog.csdnimg.cn/28be08fbcdf94d4090f555a587d9c2b7.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAZ21IYXBweQ==,size_20,color_FFFFFF,t_70,g_se,x_16)
## 3.4 配置grafana 数据源
浏览器访问地址：http://192.168.3.105:3000，登录并重新设置密码
默认用户密码：admin/admin
![在这里插入图片描述](https://img-blog.csdnimg.cn/41ac1cd5c6bd4279b9d305f4c471bf41.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAZ21IYXBweQ==,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/b2fa23f9fe9f49688567d43a9d8f7b53.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAZ21IYXBweQ==,size_20,color_FFFFFF,t_70,g_se,x_16)
配置数据源，选择`InfluxDB`，配置数据库地址、数据库名称、用户名、密码
![在这里插入图片描述](https://img-blog.csdnimg.cn/eb69ab5164f24b6787503bdd280bcb4f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAZ21IYXBweQ==,size_20,color_FFFFFF,t_70,g_se,x_16)
## 3.5 配置grafana 仪表
效果图展示
![在这里插入图片描述](https://img-blog.csdnimg.cn/2dc724e654014e98818f6111aa2e76e9.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAZ21IYXBweQ==,size_20,color_FFFFFF,t_70,g_se,x_16)
仪表盘配置：
[https://github.com/gm19900510/telegraf-influxdb-grafana/blob/main/Telegraf_%20system%20dashboard-1640155877711.json](https://github.com/gm19900510/telegraf-influxdb-grafana/blob/main/Telegraf_%20system%20dashboard-1640155877711.json)

