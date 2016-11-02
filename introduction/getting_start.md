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

##使用表达式浏览器