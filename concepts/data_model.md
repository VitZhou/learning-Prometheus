数据模型
======
Prometheus是以时间序列存储所有数据：属于相同度量和相同标记维度集合的时间戳的流。 除了存储的时间序列，Prometheus可以生成作为查询结果的临时导出的时间序列。

##指标名称和标签(Metric names and labels)
每个时间序列都由其指标名称和一组键值对（也称为标签）唯一标识组成.
度量标准名称指定所测量的系统的一般功能（例如，http_requests_total - 收到的HTTP请求的总数）。 它可以包含ASCII字母和数字，以及下划线和冒号。 它必须匹配正则表达式[a-zA-Z _：] [a-zA-Z0-9 _：] *。

标签（labels）可以启用Prometheus的维度数据模型:对于同一指标名称的标签，任何给定的标签组合标志都会实例化（any given combination of labels for the same metric name identifies a particular dimensional instantiation of that metric）(例如:使用POST方法处理/api/tracks的所有请求).查询语言允许基于这些维度进行过滤和聚合,更改任何标签值（包括删除,添加）都会创建新的时间序列.

标签名称可以包括ASCLL字母,数字以及下划线.他们必须匹配正则表达式[a-zA-Z _] [a-zA-Z0-9 _]*.以_开头的标签名保留供内部使用


##样品(Samples)
来自实际时间序列数据的样本都会包括：
- 一个float64的值
- 一个精度为毫秒的时间戳

##符号(Notation)
给定一个标识名称和一组标签，时间序列通常使用以下表示法标识：
```xml
<metric name>{<label name>=<label value>, ...}
```

例如，具有标识名称api_http_requests_total和标签method =“POST”和handler =“/ messages”的时间序列可以写成这样：
```xml
api_http_requests_total{method="POST", handler="/messages"}
```