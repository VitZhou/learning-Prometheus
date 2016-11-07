安装
======
##使用预编译的二进制文件

我们为大多数官方Prometheus组件提供预编译的二进制文件。 查看[下载页面](https://prometheus.io/download)以获取所有可用版本的列表

##使用docker
所有Prometheus服务都可以作为Dock组织下的Docker镜像。

在Docker上运行Prometheus就像:docker run -p 9090：9090 prom / prometheus一样简单。 这将启动Prometheus配置示例，并将其显示在端口9090上。

Prometheus映像使用卷存储实际指标。 对于生产部署，强烈建议使用数据卷容器模式来简化对Prometheus升级的数据管理。

要提供您自己的配置，有几个选项。 这里有两个例子。

####Volumes & bind-mount
通过运行以下步骤从主机绑定安装您的prometheus.yml：
```shell
	docker run -p 9090:9090 -v /tmp/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus
```
或者为配置使用额外的卷：
```shell
	docker run -p 9090:9090 -v /prometheus-data prom/prometheus -config.file=/prometheus-data/prometheus.yml
···

####自定义镜像
为了避免在主机上管理文件并对其进行绑定安装，配置可以嵌入到映像中。 如果配置本身是静态的，并且在所有环境中都是相同的，这样做是有好处的。

为此，使用Prometheus配置和Dockerfile创建一个新目录，如下所示：
···shell
    FROM prom/prometheus
    ADD prometheus.yml /etc/prometheus/
```
构建并且运行它:
```shell
    docker build -t my-prometheus .
    docker run -p 9090:9090 my-prometheus
```

一个更高级的选项是动态地渲染配置开始一些工具，甚至有守护程序定期更新。

