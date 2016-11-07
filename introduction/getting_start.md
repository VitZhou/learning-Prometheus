入门
======
本指南是一个“Hello World”风格的教程，显示如何在一个简单的示例设置中安装，配置和使用Prometheus。 您将在本地下载并运行Prometheus，将其配置为自己和示例应用程序，然后使用查询，规则和图形来利用收集的时间序列数据。
##Downloading and running Prometheus
在[这里](https://prometheus.io/download)下载最新的版本,然后运行它:
```shell
tar xvfz prometheus-*.tar.gz
cd prometheus-*
```

启动之前配置

##配置Prometheus监视自己
Prometheus通过收集这些目标上的HTTP端点的指标来收集受监视目标的指标。 由于Prometheus也以相同的方式暴露自己的数据，它也可以收集和监测自己的健康。
虽然只收集有关自身的数据的Prometheus服务器在实践中不是非常有用，但它是一个很好的开始示例。 将以下基本Prometheus配置保存为名为prometheus.yml的文件：
```xml
global:
  scrape_interval:     15s # 默认每15s抓取一次数据.

  # 将这些标签附加到与外部系统通信时的任何时间序列或警报（federation, remote storage, Alertmanager），
  external_labels:
    monitor: 'codelab-monitor'

# 包含一个要收集的重点的scrape配置:
# Here it's Prometheus itself.
scrape_configs:
  # `job=<job_name>`表情作为一个作业名，添加到从此配置中剪切的任何时间序列
  - job_name: 'prometheus'

    #覆盖全局默认值，并且每5秒从此作业中收集目标。
    scrape_interval: 5s

    static_configs:
      - targets: ['localhost:9090']
```
有关配置选项的完整规范,请参考[配置文档](https://prometheus.io/docs/operating/configuration)

##启动Prometheus
要使用新创建的配置文件启动Prometheus，请切换到您的Prometheus构建目录并运行：
```shell
# Start Prometheus.
# By default, Prometheus stores its database in ./data (flag -storage.local.path).
./prometheus -config.file=prometheus.yml
```
Prometheus启动，http://localhost:9090访问状态页面。 等待几秒钟，从自己的HTTP指标端点收集有关自己的数据。
您还可以通过导航到其度量标准端点来验证Prometheus是否提供有关自身的指标：http：//localhost：9090/metrics

Prometheus执行的操作系统线程数由GOMAXPROCS环境变量控制。 截至Go 1.5，默认值是可用的核心数。

将GOMAXPROCS盲目设置为高值可能会适得其反。 参考相关[Go常见问题](http://golang.org/doc/faq#Why_no_multi_CPU)。

注意，Prometheus默认情况下使用大约3GB的内存。 如果你有一台较小的机器，你可以调整Prometheus使用更少的内存。 有关详细信息，请参考[内存使用文档](https://prometheus.io/docs/operating/storage/#memory-usage)。

##使用表达式筛选
想尝试查看Prometheus的关于自己的一些数据。 要使用Prometheus的内置表达式，请导航到http：//localhost：9090/graph，然后在“graph”选项卡中选择“console”视图。

您可以从http://localhost:9090/metrics中收集，Prometheus自身导出的一个指标称为prometheus_target_interval_length_seconds（目标抓取之间的实际时间量）。 继续并输入到表达式控制台
```shell
prometheus_target_interval_length_seconds
```
这里可以返回很多不同的时间序列(以及为每个记录的最新值)，所有的具有度量名称prometheus_target_interval_length_seconds，但具有不同的标签。 这些标签指定不同的延迟百分位数和目标组间隔。

如果我们只对quantile为99的数据感兴趣，我们可以使用此查询来检索该信息：
```shell
prometheus_target_interval_length_seconds{quantile="0.99"}
```
如果想要统计时间序列,可以这样:
```shell
count(prometheus_target_interval_length_seconds)
```

更多的查询表达式,你可以查看[这里](https://prometheus.io/docs/querying/basics/)

##使用图形界面
要绘制表达式，请导航到http:/localhost:90/graph，然后使用“graph”选项卡。
例如:输入以下表达式以图形化的Prometheus中发生的所有操作的存储每秒速率：
```shell
rate(prometheus_local_storage_chunk_ops_total[1m])
```

##启动一些实例target
启动一些示例做为Prometheus的收集目标

这里的例子是用go语言做得客户端,模拟了用于导出具有不同延迟分布的三个服务的虚构RPC延迟。

确保你的机器已经安装好了正确的[go编译器](https://golang.org/doc/install)和[go构建环境](https://golang.org/doc/code.html)

下载Prometheus的go语言客户端,并且运行以下三个示例:
```shell
# Fetch the client library code and compile example.
git clone https://github.com/prometheus/client_golang.git
cd client_golang/examples/random
go get -d
go build

# Start 3 example targets in separate terminals:
./random -listen-address=:8080
./random -listen-address=:8081
./random -listen-address=:8082
```

现在，您应该有示例目标：http://localhost:8080/metrics, http://localhost:8081/metrics, 和 http://localhost:8082/metrics.

##配置Prometheus来监视样本目标
现在我们将配置Prometheus来抓取这些新的目标。 让我们将所有三个端点组合成一个名为example-random的作业。 然而，假设前两个端点是生产目标，而第三个端点表示Canary实例。 要在Prometheus中对其进行建模，我们可以将多个端点组添加到单个作业，为每组目标添加额外的标签。 在此示例中，我们将group =“production”标签添加到第一组目标，同时向第二组添加group =“canary”。

要实现此目的，请将以下作业定义添加到prometheus.yml中的scrape_configs部分，然后重新启动Prometheus实例：
```xml
scrape_configs:
  - job_name:       'example-random'

    # Override the global default and scrape targets from this job every 5 seconds.
    scrape_interval: 5s

    static_configs:
      - targets: ['localhost:8080', 'localhost:8081']
        labels:
          group: 'production'

      - targets: ['localhost:8082']
        labels:
          group: 'canary'
```

##配置要收集聚合到新时间序列的规则
虽然在我们的示例中不是问题，但是当计算机是时时收集数据，累积了上千个时间序列的查询可能变慢。为了提高效率，Prometheus允许您通过配置的记录规则将表达式预先记录为完全新的持久时间序列。 假设我们有兴趣记录所有实例（但保留作业和服务维度）上平均的示例RPC（rpc_durations_seconds_count）的每秒速率，在5分钟的窗口上测量,可以这样设置:
```xml
avg(rate(rpc_durations_seconds_count[5m])) by (job, service)
```

要将上述表达式生成的时间序列记录到名为job_service：rpc_durations_seconds_count：avg_rate5m的新度量标准中，请使用以下记录规则创建一个文件，并将其保存为prometheus.rules：
```xml
job_service:rpc_durations_seconds_count:avg_rate5m = avg(rate(rpc_durations_seconds_count[5m])) by (job, service)
```

要使Prometheus选择此新规则，请将rule_files语句添加到prometheus.yml中的全局配置部分。 配置应该现在看起来像这样：
```xml
global:
  scrape_interval:     15s # By default, scrape targets every 15 seconds.
  evaluation_interval: 15s # Evaluate rules every 15 seconds.

  # Attach these extra labels to all timeseries collected by this Prometheus instance.
  external_labels:
    monitor: 'codelab-monitor'

rule_files:
  - 'prometheus.rules'

scrape_configs:
  - job_name: 'prometheus'

    # Override the global default and scrape targets from this job every 5 seconds.
    scrape_interval: 5s

    static_configs:
      - targets: ['localhost:9090']

  - job_name:       'example-random'

    # Override the global default and scrape targets from this job every 5 seconds.
    scrape_interval: 5s

    static_configs:
      - targets: ['localhost:8080', 'localhost:8081']
        labels:
          group: 'production'

      - targets: ['localhost:8082']
        labels:
          group: 'canary'
```

使用新配置重新启动Prometheus，并验证具有度量标准名称job_service：rpc_durations_seconds_count：avg_rate5m的新时间系列现在可通过表达式浏览器查询或通过图形查看。