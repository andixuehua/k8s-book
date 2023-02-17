https://yunlzheng.gitbook.io/prometheus-book/parti-prometheus-ji-chu/quickstart



# 理解时间序列

https://yunlzheng.gitbook.io/prometheus-book/parti-prometheus-ji-chu/promql/what-is-prometheus-metrics-and-labels



时间戳转换工具　https://tool.lu/timestamp/

Prometheus会将所有采集到的样本数据以时间序列（time-series）的方式保存在内存数据库中，并且定时保存到硬盘上。time-series是按照时间戳和值的序列顺序存放的，我们称之为向量(vector). 每条time-series通过指标名称(metrics name)和一组标签集(labelset)命名。如下所示，可以将time-series理解为一个以时间为Y轴的数字矩阵：



```
^
  │   . . . . . . . . . . . . . . . . .   . .   
  │     . . . . . . . . . . . . . . . . . . .   
  │     . . . . . . . . . .   . . . . . . . .   node_load1{}
  │     . . . . . . . . . . . . . . . .   . .  
  v
    <------------------ 时间 ---------------->
    
    node_load1@1668000264
    
    1668000264是秒数,可以用上面的时间戳转换工具获得某个时间的秒数
```

在time-series中的每一个点称为一个样本（sample），样本由以下三部分组成：

- 指标(metric)：metric name和描述当前样本特征的labelsets;

- 时间戳(timestamp)：一个精确到毫秒的时间戳;

- 样本值(value)： 一个float64的浮点型数据表示当前样本的值。

  

  如 node_load1就是metric,

  
  
  下面是node_load1添加了labelsets的metric,就是具体到某个对像的metric,
  
  node_load1{container="node-exporter", endpoint="http-metrics", instance="192.168.3.75:9100", job="node-exporter", namespace="monitoring", pod="pm-prometheus-node-exporter-fglgq", service="pm-prometheus-node-exporter"}
  
  
  
  用以上metric会查出的结果就是当前时间戳　value,　　->table方式显示
  
  把各个时间戳的value关联起来就可以形成graph,图形　->graph方式显示　
  
  
  
  ```
  node_load1@1668000264 or node_load1
  
  node_load1{container="node-exporter", endpoint="http-metrics", instance="192.168.3.75:9100", job="node-exporter", namespace="monitoring", pod="pm-prometheus-node-exporter-fglgq", service="pm-prometheus-node-exporter"}
  2.28
  node_load1{container="node-exporter", endpoint="http-metrics", instance="192.168.3.76:9100", job="node-exporter", namespace="monitoring", pod="pm-prometheus-node-exporter-59426", service="pm-prometheus-node-exporter"}
  3.73
  node_load1{container="node-exporter", endpoint="http-metrics", instance="192.168.3.77:9100", job="node-exporter", namespace="monitoring", pod="pm-prometheus-node-exporter-v8qwd", service="pm-prometheus-node-exporter"}
  0.81
  node_load1{container="node-exporter", endpoint="http-metrics", instance="192.168.3.78:9100", job="node-exporter", namespace="monitoring", pod="pm-prometheus-node-exporter-8bq2r", service="pm-prometheus-node-exporter"}
  1.27
  node_load1{container="node-exporter", endpoint="http-metrics", instance="192.168.3.79:9100", job="node-exporter", namespace="monitoring", pod="pm-prometheus-node-exporter-nw9bd", service="pm-prometheus-node-exporter"}
  1.55
  node_load1{container="node-exporter", endpoint="http-metrics", instance="192.168.3.80:9100", job="node-exporter", namespace="monitoring", pod="pm-prometheus-node-exporter-tk82s", service="pm-prometheus-node-exporter"}
  ```



Prometheus采集抓取间隔时间30秒: scrape_interval,可以这里查看,

https://prometheus.monitoring.apps.taikang1.local/config



# Metrics类型

https://yunlzheng.gitbook.io/prometheus-book/parti-prometheus-ji-chu/promql/prometheus-metrics-types

从存储上来讲所有的监控指标metric都是相同的，但是在不同的场景下这些metric又有一些细微的差异。 例如，在Node Exporter返回的样本中指标node_load1反应的是当前系统的负载状态，随着时间的变化这个指标返回的样本数据是在不断变化的。而指标node_cpu所获取到的样本数据却不同，它是一个持续增大的值，因为其反应的是CPU的累积使用时间，从理论上讲只要系统不关机，这个值是会无限变大的。

为了能够帮助用户理解和区分这些不同监控指标之间的差异，Prometheus定义了4种不同的指标类型(metric type)：Counter（计数器）、Gauge（仪表盘）、Histogram（直方图）、Summary（摘要）

## Counter：只增不减的计数器

Counter类型的指标其工作方式和计数器一样，只增不减（除非系统发生重置）。常见的监控指标，如http_requests_total，node_cpu都是Counter类型的监控指标。 一般在定义Counter类型指标的名称时推荐使用_total作为后缀。

Counter是一个简单但有强大的工具，例如我们可以在应用程序中记录某些事件发生的次数，通过以时序的形式存储这些数据，我们可以轻松的了解该事件产生速率的变化。 PromQL内置的聚合操作和函数可以让用户对这些数据进行进一步的分析：

如:

```
apiserver_request_total

apiserver_request_total{code="200", component="apiserver", endpoint="https", group="admissionregistration.k8s.io", instance="192.168.3.75:6443", job="apiserver", namespace="default", resource="mutatingwebhookconfigurations", scope="cluster", service="kubernetes", verb="WATCH", version="v1"}

```



## Gauge：可增可减的仪表盘

与Counter不同，Gauge类型的指标侧重于反应系统的当前状态。因此这类指标的样本数据可增可减。常见指标如：node_memory_MemFree（主机当前空闲的内容大小）、node_memory_MemAvailable（可用内存大小）都是Gauge类型的监控指标。

通过Gauge指标，用户可以直接查看系统的当前状态：

如:

```
machine_cpu_cores
node_memory_MemFree_bytes
node_memory_MemTotal_bytes
```



## 使用Histogram和Summary分析数据分布情况



Histogram和Summary主用用于统计和分析样本的分布情况。

在大多数情况下人们都倾向于使用某些量化指标的平均值，例如CPU的平均使用率、页面的平均响应时间。这种方式的问题很明显，以系统API调用的平均响应时间为例：如果大多数API请求都维持在100ms的响应时间范围内，而个别请求的响应时间需要5s，那么就会导致某些WEB页面的响应时间落到中位数的情况，而这种现象被称为长尾问题。

为了区分是平均的慢还是长尾的慢，最简单的方式就是按照请求延迟的范围进行分组。例如，统计延迟在0~10ms之间的请求数有多少而10~20ms之间的请求数又有多少。通过这种方式可以快速分析系统慢的原因。Histogram和Summary都是为了能够解决这样问题的存在，通过Histogram和Summary类型的监控指标，我们可以快速了解监控样本的分布情况。



```
summary:

prometheus_rule_group_duration_seconds
```



如果需要聚合（aggregate），选择histograms。
如果比较清楚要观测的指标的范围和分布情况，选择histograms。如果需要精确的分为数选择summary。



----------------------------

**Histogram(柱状图)**

https://zhuanlan.zhihu.com/p/448969449

**histogram，是柱状图，在Prometheus系统中的查询语言中，有三种作用：**

- 对每个采样点进行统计，打到各个分类值中(bucket)。
- 对每个采样点值累计和(sum)。
- 对采样点的次数累计和(count)。

**度量指标名称: [basename]的柱状图, 上面三类的作用度量指标名称：**



- [basename]_bucket{le=“上边界”}, 这个值为小于等于上边界的所有采样点数量。
- [basename]_sum
- [basename]_count



**小结：所以如果定义一个度量类型为Histogram，则Prometheus系统会自动生成三个对应的指标。**

```
apiserver_response_sizes_bucket, 或　apiserver_response_sizes_bucket{le="1000"}
apiserver_response_sizes_sum
apiserver_response_sizes_count

#以上三个都是counter,组成一个apiserver_response_sizes的Histogram

```

apiserver_response_sizes就是一种Histogram指标



---------------------------------

**类似histogram柱状图，summary是采样点分位图统计。**(通常的使用场景：请求持续时间和响应大小)。它也有三种作用：

- 对于每个采样点进行统计，并形成分位图。（如：正态分布一样，统计低于60分不及格的同学比例，统计低于80分的同学比例，统计低于95分的同学比例）

  

  - 统计班上所有同学的总成绩(sum)。
  - 统计班上同学的考试总人数(count)。
  - 带有度量指标的[basename]的summary 在抓取时间序列数据展示。

  

  观察时间的φ-quantiles (0 ≤ φ ≤ 1), 显示为[basename]{分位数="[φ]"}。

  

  - [basename]_sum， 是指所有观察值的总和。
  - [basename]_count, 是指已观察到的事件计数值。



-----------------------------------

```
#summary
prometheus_rule_group_duration_seconds
prometheus_rule_group_duration_seconds{quantile='0.9'}
prometheus_rule_group_duration_seconds{quantile='0.9'}
```

prometheus_rule_group_duration_seconds就是一种summary指标



##### 如何区分prometheus中Histogram和Summary类型的metrics？

https://www.shuzhiduo.com/A/mo5kARGvzw/

要理解它们的区别，关键还是分业务应用。

但如何在学习时，如何区分呢？

有以下几个维度：

**histogram有bucket，summary在quatile(百分位)。**  

```
#summary
prometheus_rule_group_duration_seconds
prometheus_rule_group_duration_seconds{quantile='0.9'}
prometheus_rule_group_duration_seconds{quantile='0.9'}

#
```

**summary分位数是客户端计算上报，histogram中位数涉及服务端计算。**



##### 与Summary类型的指标相似之处在于Histogram类型的样本同样会反应当前指标的记录的总数(以_count作为后缀)以及其值的总量（以_sum作为后缀）。

##### 不同在于Histogram指标直接反应了在不同区间内样本的个数，区间通过标签len进行定义。



```
prometheus_rule_group_duration_seconds_sum
prometheus_rule_group_duration_seconds_count

prometheus_rule_group_duration_seconds_sum{le=0.00069531}
prometheus_rule_group_duration_seconds_count

```



使用histogram_quantile()函数，计算直方图或者是直方图聚合计算的分位数阈值。一个直方图计算Apdex值也是合适的, 当在buckets上操作时，记住直方图是累计的。

##### prometheus两种分位值histogram和summary对比histogram线性插值法原理

https://zhuanlan.zhihu.com/p/348863302









# PromQL

PromQL是Prometheus内置的数据查询语言，其提供对时间序列数据丰富的查询，聚合以及逻辑运算能力的支持。并且被广泛应用在Prometheus的日常应用当中，包括对数据查询、可视化、告警处理当中。可以这么说，PromQL是Prometheus所有应用场景的基础，理解和掌握PromQL是Prometheus入门的第一课。



Prometheus通过指标名称（metrics name）以及对应的一组标签（labelset）唯一定义一条时间序列。



#### 函数



## 计算Counter指标增长率

我们知道Counter类型的监控指标其特点是只增不减，在没有发生重置（如服务器重启，应用重启）的情况下其样本值应该是不断增大的。为了能够更直观的表示样本数据的变化剧烈情况，需要计算样本的增长速率。

```
increase(node_cpu_seconds_total[2m]) / 120

```

这里通过node_cpu[2m]获取时间序列最近两分钟的所有样本，increase计算出最近两分钟的增长量，最后除以时间120秒得到node_cpu样本在最近两分钟的平均增长率。并且这个值也近似于主机节点最近两分钟内的平均CPU使用率。

除了使用increase函数以外，PromQL中还直接内置了rate(v range-vector)函数，rate函数可以直接计算区间向量v在时间窗口内平均增长速率。因此，通过以下表达式可以得到与increase函数相同的结果：



```
rate(node_cpu_seconds_total[2m])
```



需要注意的是使用rate或者increase函数去计算样本的平均增长速率，容易陷入“长尾问题”当中，其无法反应在时间窗口内样本数据的突发变化。 例如，对于主机而言在2分钟的时间窗口内，可能在某一个由于访问量或者其它问题导致CPU占用100%的情况，但是通过计算在时间窗口内的平均增长率却无法反应出该问题。

为了解决该问题，PromQL提供了另外一个灵敏度更高的函数irate(v range-vector)。irate同样用于计算区间向量的计算率，但是其反应出的是瞬时增长率。irate函数是通过区间向量中最后两个样本数据来计算区间向量的增长速率。这种方式可以避免在时间窗口范围内的“长尾问题”，并且体现出更好的灵敏度，通过irate函数绘制的图标能够更好的反应样本数据的瞬时变化状态。



```
irate(node_cpu_seconds_total[2m])
```

irate函数相比于rate函数提供了更高的灵敏度，不过当需要分析长期趋势或者在告警规则中，irate的这种灵敏度反而容易造成干扰。因此在长期趋势分析或者告警中更推荐使用rate函数。



## 预测Gauge指标变化趋势

在一般情况下，系统管理员为了确保业务的持续可用运行，会针对服务器的资源设置相应的告警阈值。例如，当磁盘空间只剩512MB时向相关人员发送告警通知。 这种基于阈值的告警模式对于当资源用量是平滑增长的情况下是能够有效的工作的。 但是如果资源不是平滑变化的呢？ 比如有些某些业务增长，存储空间的增长速率提升了高几倍。这时，如果基于原有阈值去触发告警，当系统管理员接收到告警以后可能还没来得及去处理问题，系统就已经不可用了。 因此阈值通常来说不是固定的，需要定期进行调整才能保证该告警阈值能够发挥去作用。 那么还有没有更好的方法吗？

PromQL中内置的predict_linear(v range-vector, t scalar) 函数可以帮助系统管理员更好的处理此类情况，predict_linear函数可以预测时间序列v在t秒后的值。它基于简单线性回归的方式，对时间窗口内的样本数据进行统计，从而可以对时间序列的变化趋势做出预测。例如，基于2小时的样本数据，来预测主机可用磁盘空间的是否在4个小时候被占满，可以使用如下表达式：



```
predict_linear(node_filesystem_free_bytes{device="/dev/sda1"}[2h], 4 * 3600) < 0
predict_linear(node_filesystem_free_bytes{device="/dev/sda1"}[2h], 4 * 3600) < 0

predict_linear(node_filesystem_free_bytes{device="/dev/sda1"}[2h], 24 * 3600)/1024/1024
```



## 统计Histogram指标的分位数

在本章的第2小节中，我们介绍了Prometheus的四种监控指标类型，其中Histogram和Summary都可以用于统计和分析数据的分布情况。区别在于Summary是直接在客户端计算了数据分布的分位数情况。而Histogram的分位数计算需要通过histogram_quantile(φ float, b instant-vector)函数进行计算。其中φ（0<φ<1）表示需要计算的分位数，如果需要计算中位数φ取值为0.5，以此类推即可。



```
histogram_quantile(0.5, apiserver_response_sizes_bucket)
```



如果想要查询每台[服务器](https://cloud.tencent.com/product/cvm?from=10680)每 1 分钟的 CPU 负载是多少，



```
(1-((sum(increase(node_cpu_seconds_total{mode="idle"}[1m])) by (instance)) /(sum(increase(node_cpu_seconds_total[1m])) by (instance)))) * 100
```



可参考以下:

#有些不能用

https://github.com/infinityworks/prometheus-example-queries





https://sysdig.com/blog/prometheus-query-examples/



这里比较全,与alert结合使用:

https://github.com/samber/awesome-prometheus-alerts

https://awesome-prometheus-alerts.grep.to/rules#kubernetes



#### PromSQL参考:



https://blog.csdn.net/weixin_56175092/article/details/125554841

标签匹配操作符

也可以对标签值进行负匹配，或者将标签值与正则表达式进行匹配，存在以下标签匹配操作符:

- **`=`**: 选择与提供的字符串完全相等的标签；
- **`!=`**: 选择与提供的字符串不相等的标签；
- **`=~`**: 选择正则表达式与提供的字符串匹配的标签；
- **`!~`**: 选择与提供的字符串不匹配的标签；

正则表达式匹配是完全对应的，匹配`env=~"foo"`被视为`env=~"^foo$"`。
