# 简介

## 为什么需要普罗米修斯？

普罗米修斯官网的首页简单的对普罗米修斯做了定义：**从指标到洞察力** ，普罗米修斯通过领先的开源监控解决方案为用户的指标和告警提供强大的支持。

![image20230114203332341.png](/img/chapter_post/img_32.png)

可以看到普罗米修斯是领先的、开源的、也是一种监控解决方案、支持用户指标和告警等需求。使用普罗米修斯可以有效的解决在云原生时代下的指标埋点，服务异常监控等需求，比如:

- 借助时序数据库来**存储海量多维度指标数据** ，使用PromQL数据查询，聚合分析指标数据或者Grafana这样的图形化页面展示指标数据。
- 传统监控的**异常监控** 需求，也就是监控那些我们知道某个地方可能会出现问题但是又不知道何时会出现问题(Know-Unknow)的地方。
- 云原生时代服务快速的重启发布，自动弹性扩缩容，面对海量容器POD频繁变化，每次地址发生了变化修改配置肯定是不现实的，普罗米修斯通过**及时感知的服务发现模型** 来解决云原生时代大规模服务发现问题。

当然作为云原生优秀的监控系统，并不仅仅可以解决这里罗列的问题，普罗米修斯生态庞大，在云原生时代为可观测性的指标埋点提供了足够的铺垫。后续通过一些可观测性技术深度串联分析链路和日志数据通过故障预测，根因分析可以有效的解决我们不知道会出现问题的地方和不知道何时会出现问题的地方（Unknow-Unknow）。普罗米修斯不仅仅可以洞察主机层的指标信息，也可以深度通过系统指标埋点深度洞察系统内部的健康状态，那具体怎么做呢？可以继续往下看。

## 起源

普罗米修斯是由SoundCloud开发的开源监控告警系统，是Google BorgMon监控系统的开源版本。2016年，由Google发起的Linux基金会旗下的原生云基金会CNCF（Cloud Native Computing Foundation）将Prometheus纳入其第二大开源项目(第一大开源项目是kunernetes)。

![prometheus-release-roadmaps (2)](/img/chapter_post/img_33.png)


2012年开源的普罗米修斯监控系统从开源到现在经过了数十年的打磨具备哪些特性呢？从官方文档参考到的内容如下所示：

![image20230115173801018.png](/img/chapter_post/img_34.png)

可以看到普罗米修斯在多维度指标监控告警等方面拥有强大的支持，下面就进入正题，从普罗米修斯的架构到入门案例来看下如何使用普罗米修斯进行服务指标监控。

# 架构

下面就直接来看下Prometheus 的架构及其一些生态系统组件：

![普罗米修斯架构](/img/chapter_post/img_35.png)


这个图完整的体现了普罗米修斯从发现服务，采集数据，到监控告警分析数据的整个过程：

![image.png](/img/chapter_post/img_36.png)

初步了解了普罗米修斯的一些概念，想要优雅的使用普罗米修斯监控还需要我们了解一些常见术语，下面可以看下。

## 常见术语

下面列举一些我们常用的术语说明：

- **Meters（指标）**

用外行的话来说，*指标*是数字测量。*时间序列*意味着随着时间的推移记录变化。用户想要测量的内容因应用程序而异。对于 Web 服务器，它可能是请求时间，对于数据库，它可能是活动连接数或活动查询数等。

- **Collector（收集器）**

收集器是代表一组指标的导出器的一部分。如果它是直接检测的一部分，则它可能是单个指标；如果它是从另一个系统提取指标，则它可能是多个指标。

- **Endpoint（端点）**

  可以抓取的指标来源，通常对应于单个进程。

- **Exporter（导出器）**

  导出器是与您要从中获取指标的应用程序一起运行的二进制文件。导出器公开 普罗米修斯 指标，通常是将以非 普罗米修斯 格式公开的指标转换为 普罗米修斯 支持的格式。

- **PromQL（普罗米修斯查询语言）**

  PromQL是普罗米修斯查询语言。它允许进行广泛的操作，包括聚合、切片和切块、预测和连接。

- **Pushgateway（推送网关）**

  Pushgateway保留来自批处理作业的最新指标推送。这允许 普罗米修斯 在它们终止后抓取它们的指标（实时性较高可以先缓存在推送网关中后续由普罗米修斯拉取。

- **Sample（样本）**

  样本是时间序列中某个时间点的单个值。在 普罗米修斯 中，每个样本都包含一个 float64 值和一个毫秒精度的时间戳。

- **The Four Golden Signals（四大黄金信号）**
  ![image20230115183536075.png](/img/chapter_post/img_37.png)


Google SRE中提到的概念，监控的四个黄金信号是延迟、流量、错误和饱和度。如果您只能衡量面向用户的系统的四个指标，请关注这四个指标。原文链接如下

[https://sre.google/sre-book/monitoring-distributed-systems/](https://sre.google/sre-book/monitoring-distributed-systems/)

- **Black-Box Versus White-Box（黑盒与白盒）**

![image20230115183829944.png](/img/chapter_post/img_37.png)

- **Metric names and labels（指标名称和标签）**

    - **指标名称：** 指定了被测系统的一般特征（例如`http_requests_total`- 接收到的 HTTP 请求总数

    - **标签：** 启用 Prometheus 的维度数据模型：相同指标名称的任何给定标签组合标识该指标的**特定维度** 实例（例如：所有使用处理程序方法`POST`的HTTP 请求`/api/tracks`）。查询语言允许基于这些维度进行过滤和聚合。更改任何标签值，包括添加或删除标签，都将创建一个新的时间序列。

- **METRIC TYPES（指标类型）**
  ![image20230115184057358.png](/img/chapter_post/img_38.png)

Prometheus 客户端库提供四种核心指标类型，用来解决不同指标差异区分，帮助用户理解和区分这些不同监控指标之间的差异，Prometheus 定义了 4 种不同的指标类型：`Counter（计数器）`、`Gauge（仪表盘）`、`Histogram（直方图）`、`Summary（摘要）`。

这里常见术语列举的相对还是比较多的，不过慢慢消化，下面就开始通过一个简单的案例来入门普罗米修斯的使用来实现对普罗米修斯自身的一些指标的暴漏与抓取。

# 入门示例

## 普罗米修的安装

这里演示环境为Centos7系统

### 下载

登录服务器后，直接输入如下命令,从官方仓库下载压缩文件到本地，并解压。

```bash
[root@iZ8lgm9icspkthZ ~] wget https://github.com/prometheus/prometheus/releases/download/v2.41.0/prometheus-2.41.0.linux-amd64.tar.gz

[root@iZ8lgm9icspkthZ ~] tar -zxvf ./prometheus-2.41.0.linux-amd64.tar.gz 
```

### 配置

打开prometheus.yml配置文件，可以看到配置文件里面默认文件如下所示：

```yaml
# my global config 全局配置（如果有内部单独设定，会覆盖这个参数）
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute. 默认15s 全局每次数据收集的间隔
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).  规则扫描时间间隔是15秒，

# Alertmanager configuration  告警插件定义。这里会设定alertmanager这个报警插件。
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.  告警规则。 按照设定参数进行扫描加载，用于自定义报警规则，其报警媒介和route路由由alertmanager插件实现。
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.  采集配置。配置数据源，包含分组job_name以及具体target。又分为静态配置和服务发现
  - job_name: "prometheus"  #任务目标名，可以理解成分组，每个分组包含具体的target组员。

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]
```

### 启动服务

指定配置文件，同时后台运行服务

```bash
./prometheus --config.file=prometheus.yml &
```

启动服务成功后可以看到如下info日志，普罗米修斯启动后监听了一个9090端口。

[//]: # (![image20230115160933693.png]&#40;/img/chapter_post/img_39.png&#41;)

### 访问Dashboard

浏览器打开地址 `http://当前服务器IP:9090`   即可，可以看到如下可视化页面：
![image20230115161304856.png](/img/chapter_post/img_39.png)

在菜单栏中找到服务发现地址如下：
![image20230115161356691.png](/img/chapter_post/img_40.png)

### 指标查询

**指标解析**

指标查询这里提供两种方式，一种是直接在服务器上访问地址如下命令：

```bash
curl http://localhost:9090/metrics
```

查询后会得到普罗米修斯提供的指标类型，如下图：
![image20230115162004831.png](https://i-blog.csdnimg.cn/blog_migrate/23b40e43e653077397f98370131280c8.png)

可以看到这个图中存在一个指标名称为promhttp_metric_handler_requests_total的指标，#HELP中的内容为当前指标的描述，#TYPE中的内容是描述当前指标的类型，指标的详细格式为给定一个指标名称和一组标签，时间序列通常使用这种表示法来识别：

```bash
<metric name>{<label name>=<label value>, ...}
```

关于指标的命名：前缀通常是指标类型名称，后缀必须有一个单位，更详细的指标命名规范可以参考如下链接：

[https://prometheus.io/docs/practices/naming/](https://prometheus.io/docs/practices/naming/)

**PromQL查询**

让监控的数据会说话。日常数据查询、可视化及告警配置这三大功能模块都是依赖PromQL实现的，PromQL (Prometheus Query Language) 是 Prometheus 自己开发的数据查询 DSL 语言，语言表现力非常丰富，内置函数很多，在日常数据可视化以及rule 告警中都会使用到它。

这里我们介绍一些简单的语法。首先来看下表达式语言数据类型。

在 Prometheus 的表达式语言中，表达式或子表达式可以计算为四种类型之一：

![image20230115184520607.png](/img/chapter_post/img_41.png)
PromQL 查询结果主要有 3 种类型：

- **瞬时数据 (Instant vector):**
    - 包含一组时序，每个时序只有一个点，例如：`http_requests_total` `http_requests_total{}`
- **区间数据 (Range vector):**
    - 包含一组时序，每个时序有多个点，例如：`http_requests_total[5m]`
- **纯量数据 (Scalar):**
    - 纯量只有一个数字，没有时序，例如：`count(http_requests_total)`

了解了基本语法之后可以重新打开dashboard输入以下命令来查询筛选标签中状态码为200的请求数据：

`promhttp_metric_handler_requests_total{code="200"}` 查询结果如下图所示：

第一个图为表格展示列表数据

![image20230115163635316.png](/img/chapter_post/img_42.png)
第二个图以图表形式展示
![image20230115163703836.png](/img/chapter_post/img_43.png)

# 总结

完善的监控系统能够引导技术人员快速定位问题并解决，让监控告警先于用户发现问题的最佳手段，Prometheus是基于指标的监控系统，是打造一站式通用监控架构的最佳方案之一，借助普罗米修斯监控系统可以尝试在开发之初就想好要需要为业务埋下哪些监控埋点，当然也有人提出指标驱动开发（MDD）的开发理念，**通过实时指标来驱动快速、精确和细粒度的软件迭代，**  帮助我们更早地 **发现问题** 和 **明确目标**

当然普罗米修斯也不是万能的，使用时也需要注意很多的注意事项，比如：

- 如果Pushgateway从许多不同的来源收集指标时宕机，用户将失去对所有这些来源的监控，可能会触发许多不必要的告警。

- Alertmanager是独立于Prometheus的一个告警组件，需要单独安装部署，Prometheus可以将多个Alertmanager配置为一个集群，通过服务发现动态发现告警集群中节点的上下，如下图：
  ![image20230115165102377.png](/img/chapter_post/img_44.png)


- 另外存储方面普罗米修斯并不是为了解决大容量存储问题，TB级以上数据建议保存到远端TSDB中，通常来说，InfluxDB在集群方面的表现更佳，但是InfluxDB的单机版本免费，而集群版本是收费的。

- 作为时序数据库普罗米修斯不仅仅对系统时间的准确性要求很高，必须保证本机时间实时同步。另外还需要注意监控的高可用搭建，如果监控挂了一切系统将成为黑盒，即便系统处理问题也无法及时发现，这里可以通过Prometheus中有3种常见的HA架构来保证高可用，分别是简单HA、基本HA+远程存储、基本HA+远程存储+联邦集等方式 配置。




