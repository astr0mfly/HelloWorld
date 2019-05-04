---
date: "2019-04-25T09:06:07-07:00"
draft: false
title: "Elasticsrh-Introduce"
tags: [elk]
#series: []
#categories: []
toc: true
---

## ELK技术栈ElasticSearch，Logstash，Kibana

调研了ELK技术栈，发现新一代的logstash-forward即Filebeat，使用了golang，性能超logstash，部署简单，占用资源少，可以很方便的和logstash和ES对接，作为日志文件采集组件。所以决定使用ELK+Filebeat的架构进行平台搭建。

Filebeat是Beats家族的一员，后续可以使用Packetbeat进行网络数据采集、Winlogbeat进行Windosw事件采集、Heartbeat进行心跳采集、Metricbeat进行系统指标采集。这种架构解决了 Logstash 在各服务器节点上占用系统资源高的问题。相比 Logstash，Beats 所占系统的 CPU 和内存几乎可以忽略不计。另外，Beats 和 Logstash 之间支持 SSL/TLS 加密传输，客户端和服务器双向认证，保证了通信安全。

各组件承担的角色和功能：

**Elasticsearch**：

分布式搜索和分析引擎，具有高可伸缩、高可靠和易管理等特点。基于 Apache Lucene 构建，能对大容量的数据进行接近实时的存储、搜索和分析操作。通常被用作某些应用的基础搜索引擎，使其具有复杂的搜索功能； 

**Logstash**：数据处理引擎，它支持动态的从各种数据源搜集数据，并对数据进行过滤、分析、丰富、统一格式等操作，然后存储到 ES； 
Kibana：数据分析和可视化平台。与 Elasticsearch 配合使用，对数据进行搜索、分析和以统计图表的方式展示； 

**Filebeat**：ELK 协议栈的新成员，一个轻量级开源日志文件数据搜集器，使用 golang 基于 Logstash-Forwarder 源代码开发，是对它的替代。在需要采集日志数据的 server 上安装 Filebeat，并指定日志目录或日志文件后，Filebeat 就能读取数据，迅速发送到 Logstash 进行解析。 

## 常用架构
在这个章节中，我们将介绍几种常用架构及使用场景。

在这种架构中，只有一个 Logstash、Elasticsearch 和 Kibana 实例，集中部署于一台服务器。Logstash 通过输入插件从多种数据源（比如日志文件、标准输入 Stdin 等）获取数据，再经过滤插件加工数据，然后经 Elasticsearch 输出插件输出到 Elasticsearch，通过 Kibana 展示。

- All-in-One 
    这种架构非常简单，使用场景也有限。初学者可以搭建这个架构，了解 ELK 如何工作。
- Logstash 分布式采集
    这种架构是对上面架构的扩展，把一个 Logstash 数据搜集节点扩展到多个，分布于多台机器，将解析好的数据发送到 Elasticsearch server 进行存储，最后在 Kibana 查询、生成日志报表等。
- Logstash 分布式采集 
    这种结构因为需要在各个服务器上部署 Logstash，而它比较消耗 CPU 和内存资源，所以比较适合计算资源丰富的服务器，否则容易造成服务器性能下降，甚至可能导致无法正常工作。
- Beats 分布式采集
    这种架构引入 Beats 作为日志搜集器。目前 Beats 包括四种：
    
    - Packetbeat（搜集网络流量数据）； 
    - Topbeat（搜集系统、进程和文件系统级别的 CPU 和内存使用情况等数据）；
    - Filebeat（搜集文件数据）； 
    - Winlogbeat（搜集 Windows 事件日志数据）。 
Beats 将搜集到的数据发送到 Logstash，经 Logstash 解析、过滤后，将其发送到 Elasticsearch 存储，并由 Kibana 呈现给用户。
 
这种架构解决了 Logstash 在各服务器节点上占用系统资源高的问题。相比 Logstash，Beats 所占系统的 CPU 和内存几乎可以忽略不计。另外，Beats 和 Logstash 之间支持 SSL/TLS 加密传输，客户端和服务器双向认证，保证了通信安全。
因此这种架构适合对数据安全性要求较高，同时各服务器性能比较敏感的场景。

- 引入消息队列机制的 Logstash 分布式架构
    这种架构使用 Logstash 从各个数据源搜集数据，然后经消息队列输出插件输出到消息队列中。目前 Logstash 支持 Kafka、Redis、RabbitMQ 等常见消息队列。然后 Logstash 通过消息队列输入插件从队列中获取数据，分析过滤后经输出插件发送到 Elasticsearch，最后通过 Kibana 展示。详见图 4。

    这种架构适合于日志规模比较庞大的情况。但由于 Logstash 日志解析节点和 Elasticsearch 的负荷比较重，可将他们配置为集群模式，以分担负荷。引入消息队列，均衡了网络传输，从而降低了网络闭塞，尤其是丢失数据的可能性，但依然存在 Logstash 占用系统资源过多的问题。

- 引入消息队列机制的 Filebeat + Logstash 分布式架构
    截至到我们调研为止，Filebeat 已经支持 kafka 作为 ouput，5.2.1 版本的 Logstash 已经支持 Kafka 作为 Input，和上面的架构不同的地方仅在于，把 Logstash 日志搜集发送替换为了 Filebeat。这种架构是当前最为完美的，有极低的客户端采集开销，引入消息队列，均衡了网络传输，从而降低了网络闭塞，尤其是丢失数据的可能性。
    引入消息队列机制的 Filebeat + Logstash 分布式架构 
    对于绿湾的架构选型来说，高质量的数据传输、日志采集的低资源开销都需要考虑，同时，也需要logstash强大的插件支持灵活的日志数据过滤处理。确定采用引入消息队列机制的 Filebeat + Logstash 分布式架构。

## 什么是日志分析平台
场景分析：
除了排错之外，日志本身也能给我们提供非常有价值的信息，比方说服务器提供了 100 个对外接口（假设这些接口是并行的，即关闭哪个都无所谓），忽然老板说我们不能提供这么多，只能保留 50 个。那怎么确定要关闭哪五十个呢？其中一个方法就是把访问次数最少的那些给干掉。这时候我们就可以把过去一个星期的日志找出来，统计一下各个接口的使用情况（假设每次接口被访问都会生成一条日志），然后就能排个序，确定需要去掉的接口了。

- 我们提供一些服务，这些服务每被访问一次都会生成一条日志
- 一般来说我们会把程序产生的日志按日切割，也就是每天会生成一个新的日志文件
- 有的时候我们需要对大量日志进行统计以得到某些数据

日志分析平台应运而生，一般来说套路分三步：

- 把分散在各个机器的日志汇总到一个地方(Shipper, Broker, Indexer)
- 把这些日志用某种方式保存并索引起来(Search & Storage)
- 需要的时候直接在汇总的日志中查询(Web Interface)


logs -Shipper-> Kafka(Broker) -> LogStash(Indexer) -> ElasticSearch(Search&Storage) -> Kibana(WebInterface)
### 为什么选择 ElasticStack
ElasticStack 最初的核心是 ELK(Elasticsearch, Logstash, Kibana) 三兄弟。其中 Logstash 收集数据，Elasticsearch 索引数据，Kibana 展示数据。

- Elasticsearch 背靠 Lucene 这一老牌劲旅做到了准实时全文索引
- Logstash 的配置直接是 Ruby DSL，非常灵活简单
- Kibana 则自带各种查询聚合以及生成报表功能。

再加上查询简单、扩展容易之类的特点，大受欢迎其实也在情理之中。官方也在不断吸收社区精华的同时开发了安全、报警、监控、报告等一系列功能，再加上能够轻松和 Hadoop 这类分布式计算框架配合，怎么看都是非常不错的选择。

环境配置：
```shell
# 添加源 
sudo add-apt-repository -y ppa:webupd8team/java
# 更新地址 
sudo apt-get update
# 安装 
sudo apt-get -y install oracle-java8-installer


```
### 安装ELK
打开浏览器，访问 localhost:5601，如果看到如下所示的页面，Elasticsearch 和 Kibana 基本就没问题了。

体验一下 Logstash
```shell
# 进入文件夹
cd logstash-5.0.1/
# 启动 logstash，输入和输出均为命令行
bin/logstash -e 'input { stdin { } } output { stdout {} }'

#然后我们随意输入一些内容，显示为：
parallels@ubuntu:~/logstash-5.0.1$ bin/logstash -e 'input { stdin { } } output { stdout {} }'
wdxtub.com updated
Sending Logstash's logs to /home/parallels/logstash-5.0.1/logs which is now configured via log4j2.properties
The stdin plugin is now waiting for input:[2016-11-20T22:54:20,014][INFO ][logstash.pipeline        ] Starting pipeline {"id"=>"main", "pipeline.workers"=>2, "pipeline.batch.size"=>125, "pipeline.batch.delay"=>5, "pipeline.max_inflight"=>250}

[2016-11-20T22:54:20,038][INFO ][logstash.pipeline        ] Pipeline main started
[2016-11-20T22:54:20,088][INFO ][logstash.agent           ] Successfully started Logstash API endpoint {:port=>9600}
2016-11-21T06:54:20.036Z ubuntu wdxtub.com updated
wdxtub.com is a personal blog
2016-11-21T06:54:41.187Z ubuntu wdxtub.com is a personal blog
wdxtub.com is created in 2015
2016-11-21T06:54:55.190Z ubuntu wdxtub.com is created in 2015

```
### 入门Elasticsearch 
和 Mongodb 的思路类似，Elasticsearch 中保存的是整个文档(document)，并且还会根据文档的内容进行索引，于是我们得以进行搜索、排序和过滤等操作。在 Elasticsearch 中，利用 JSON 来表示文档。

在 Elasticsearch 中存储数据的行为就叫做索引(indexing)，而前面提到的文档，属于一种类型(type)，这里类型会存在索引(index)中，如果列一个表来和传统数据库比较，大概是这样的：

|关系型数据|Elasticsearch|
|-|-|
|Databases|Indices|
|Tables|Types|
|Rows|Documents|
|Columns|Fields|

一个 Elasticsearch 集群可以包含多个索引(indices，对应于『数据库』)，每个索引可以包含多个类型(types，对应于『表』)，每个类型可以包含多个文档(document，对应于『行』)，每个文档可以包含多个字段(fields，对应于『列』)

### 启动 Logstash

Logstash 使用一个名叫 FileWatch 的 Ruby Gem 库来监听文件变化。这个库支持 glob 展开文件路径，而且会记录一个叫 .sincedb 的数据库文件来跟踪被监听的日志文件的当前读取位置。通过记录下来的 inode, major number, minor number 和 pos 就可以保证不漏过每一条日志。

配置：
```
# 我的习惯是把配置文件统一放到名为 confs 的文件夹中
# 本配置文件名为 syslog.conf
input {
  file {
    # 确定需要检测的文件
    path => [ "/var/log/*.log", "/var/log/messages", "/var/log/syslog", "/var/log/apt", "/var/log/fsck", "/var/log/faillog"]
    # 日志类型
    type => "syslog"
  }
}
 
output {
  # 输出到命令行，一般用于调试
  stdout { 
    codec => rubydebug 
  }
  # 输出到 elasticsearch，这里指定索引名为 system-log
  elasticsearch { 
    hosts => "localhost:9200"
    index => "system-log" 
  }
}

```
这里说一下 File rotation 的情况，为了处理被 rotate 的情况，最好把 rotate 之后的文件名也加到 path 中（如上面所示），这里注意，如果 start_position 被设为 beginning，被 rotate 的文件因为会被认为是新文件，而重新导入。如果用默认值 end，那么在最后一次读之后到被 rotate 结束前生成的日志不会被采集。
```
# -f 表示从文件中读取配置
bin/logstash -f confs/syslog.conf

```

### 使用 Kibana

Kibana 相当于是 Elasticsearch 的一个可视化插件，所以我们需要在 Management 页面中告诉 Kibana 我们刚才创建的 system-log 索引，完成之后可以看到具体的条目及对应的信息（包括是否可被检索，是否能聚合，是否被分词等）

- Discover: 探索数据
- Visualize: 可视化统计
- Dashboard: 仪表盘
- Timelion: 时序，这里我们暂时不用
- Management: 设置
- Dev Tools: 开发工具，可以方便的测试内置接口

###  Kafka 缓冲区 
作为云计算大数据的套件，Kafka 是一个分布式的、可分区的、可复制的消息系统。该有的功能基本都有，而且有自己的特色：

- 以 topic 为单位进行消息归纳
- 向 topic 发布消息的是 producer
- 从 topic 获取消息的是 consumer
- 集群方式运行，每个服务叫 broker
- 客户端和服务器通过 TCP 进行通信

在Kafka集群中，没有“中心主节点”的概念，集群中所有的服务器都是对等的，因此，可以在不做任何配置的更改的情况下实现服务器的的添加与删除，同样的消息的生产者和消费者也能够做到随意重启和机器的上下线。

对每个 topic 来说，Kafka 会对其进行分区，每个分区都由一系列有序的、不可变的消息组成，这些消息被连续的追加到分区中。分区中的每个消息都有一个连续的序列号叫做 offset,用来在分区中唯一的标识这个消息。

发布消息通常有两种模式：队列模式(queuing)和发布-订阅模式(publish-subscribe)。队列模式中，consumers 可以同时从服务端读取消息，每个消息只被其中一个 consumer 读到；发布-订阅模式中消息被广播到所有的 consumer 中。更常见的是，每个 topic 都有若干数量的 consumer 组，每个组都是一个逻辑上的『订阅者』，为了容错和更好的稳定性，每个组由若干 consumer 组成。这其实就是一个发布-订阅模式，只不过订阅者是个组而不是单个 consumer。

通过分区的概念，Kafka 可以在多个 consumer 组并发的情况下提供较好的有序性和负载均衡。将每个分区分只分发给一个 consumer 组，这样一个分区就只被这个组的一个 consumer 消费，就可以顺序的消费这个分区的消息。因为有多个分区，依然可以在多个 consumer 组之间进行负载均衡。注意 consumer 组的数量不能多于分区的数量，也就是有多少分区就允许多少并发消费。

Kafka 只能保证一个分区之内消息的有序性，在不同的分区之间是不可以的，这已经可以满足大部分应用的需求。如果需要 topic 中所有消息的有序性，那就只能让这个 topic 只有一个分区，当然也就只有一个 consumer 组消费它。

配置：
```
# 在 zookeeper.properties 中
/data/home/logger/kafka-data/zookeeper

# 在 server.properties 中
log.dirs=/data/home/logger/kafka-data/kafka-logs

# advertised.listerners 改为对外服务的地址
# 比如对外的 ip 地址是 xx.xx.xx.xx，端口是 8080，那么
advertised.listeners=PLAINTEXT://xx.xx.xx.xx:8080

# 允许删除 topic
delete.topic.enable=true

# 不允许自动创建 topic，方便管理
auto.create.topics.enable=false

# 设定每个 topic 的分区数量，这里设为 100
num.partitions=100

# 设定日志保留的时间，这里改为 72 小时
log.retention.hours=72

# 可以使用 tmux 或 nohup & 等方式来进行后台运行，这里使用 tmux
# 启动 Zookeeper
bin/zookeeper-server-start.sh config/zookeeper.properties

# 启动 Kafka
bin/kafka-server-start.sh config/server.properties

# 创建 topic
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic wdxtub

bin/kafka-topics.sh --list --zookeeper localhost:2181

# 创建一个向 topic wdxtub 发送消息的 producer
# 按回车发送，ctrl+c 退出
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic wdxtub

# 创建一个从 topic wdxtub 读取消息的 consumer
bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic wdxtub --from-beginning

# producer 窗口内容
$> ~/kafka_2.11-0.10.1.0$ bin/kafka-console-producer.sh --broker-list localhost:9092 --topic wdxtub
abcdefu
dalkdjflka^H^H^H^H^H^H^H
wdxtub.com
wdxtub.com is good

# consumer 窗口内容
$> ~/kafka_2.11-0.10.1.0$ bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic wdxtub --from-beginning
Using the ConsoleConsumer with old consumer is deprecated and will be removed in a future major release. Consider using the new consumer by passing [bootstrap-server] instead of [zookeeper].
abcdefu
dalkdjflka
wdxtub.com
wdxtub.com is good

:nignx
upstream mq_pool{
server ip1:9092 weight=1 max_fails=3 fail_timeout=30s;
server localhost:9092 weight=1 max_fails=3 fail_timeout=30s;
}

server{
listen 8080;
allow all;
proxy_pass mq_pool;
proxy_connect_timeout 24h;
proxy_timeout 24h;
}

nignx


## 测试kafka

# 名为 kafka-test.py
from kafka import KafkaProducer
# 设置 Kafka 地址
producer = KafkaProducer(
    bootstrap_servers='your.host.name:8080')

# 设置需要发送的 topic 及内容
producer.send('wdxtub', 'Hello World! This is wdxtub.com.')

```

**常用操作**：
```
#创建 topic
bin/kafka-topics.sh --zookeeper zk_host:port/chroot --create --topic my_topic_name --partitions 20 --replication-factor 3 --config x=y

#修改 topic
bin/kafka-topics.sh --zookeeper zk_host:port/chroot --alter --topic my_topic_name --partitions 40

#增加配置
bin/kafka-topics.sh --zookeeper zk_host:port/chroot --alter --topic my_topic_name --config x=y

#移除配置
bin/kafka-topics.sh --zookeeper zk_host:port/chroot --alter --topic my_topic_name --delete-config x

#删除 topic
bin/kafka-topics.sh --zookeeper zk_host:port/chroot --delete --topic my_topic_name

#优雅关闭
Kafka 会自动检测 broker 的状态并根据机器状态选举出新的 leader。但是如果需要进行配置更改停机的时候，我们就需要使用优雅关闭了，好处在于：

会把所有的日志同步到磁盘上，避免重启之后的日志恢复，减少重启时间
会在关闭前把以这台机为 leader 的分区数据迁移到其他节点，会减少不可用的时间
但是这个需要开启 controlled.shutdown.enable=true。
刚重启之后的节点不是任何分区的 leader，所以这时候需要进行重新分配：
bin/kafka-preferred-replica-election.sh --zookeeper zk_host:port/chroot
这里需要开启 auto.leader.rebalance.enable=true
然后可以使用脚本 bin/kafka-server-stop.sh



```

性能优化
线性读写的情况下影响磁盘性能问题大约有两个方面：
太多的琐碎的 I/O 操作和太多的字节拷贝。
I/O 问题发生在客户端和服务端之间，也发生在服务端内部的持久化的操作中。

**消息集(message set)**
为了避免这些问题，Kafka 建立了消息集(message set)的概念，将消息组织到一起，作为处理的单位。以消息集为单位处理消息，比以单个的消息为单位处理，会提升不少性能。Producer 把消息集一块发送给服务端，而不是一条条的发送；服务端把消息集一次性的追加到日志文件中，这样减少了琐碎的 I/O 操作。consumer 也可以一次性的请求一个消息集。

另外一个性能优化是在字节拷贝方面。在低负载的情况下这不是问题，但是在高负载的情况下它的影响还是很大的。为了避免这个问题，Kafka 使用了标准的二进制消息格式，这个格式可以在 producer, broker 和 producer 之间共享而无需做任何改动。

**zero copy**
Broker 维护的消息日志仅仅是一些目录文件，消息集以固定队的格式写入到日志文件中，这个格式 producer 和 consumer 是共享的，这使得 Kafka 可以一个很重要的点进行优化：消息在网络上的传递。现代的 unix 操作系统提供了高性能的将数据从页面缓存发送到 socket 的系统函数，在 linux 中，这个函数是 sendfile
为了更好的理解 sendfile 的好处，我们先来看下一般将数据从文件发送到 socket 的数据流向：
- 操作系统把数据从文件拷贝内核中的页缓存中
- 应用程序从页缓存从把数据拷贝自己的内存缓存中
- 应用程序将数据写入到内核中 socket 缓存中
- 操作系统把数据从 socket 缓存中拷贝到网卡接口缓存，从这里发送到网络上。

这显然是低效率的，有 4 次拷贝和 2 次系统调用。sendfile 通过直接将数据从页面缓存发送网卡接口缓存，避免了重复拷贝，大大的优化了性能。

在一个多consumers的场景里，数据仅仅被拷贝到页面缓存一次而不是每次消费消息的时候都重复的进行拷贝。这使得消息以近乎网络带宽的速率发送出去。这样在磁盘层面你几乎看不到任何的读操作，因为数据都是从页面缓存中直接发送到网络上去了。

**数据压缩**

很多时候，性能的瓶颈并非CPU或者硬盘而是网络带宽，对于需要在数据中心之间传送大量数据的应用更是如此。当然用户可以在没有 Kafka 支持的情况下各自压缩自己的消息，但是这将导致较低的压缩率，因为相比于将消息单独压缩，将大量文件压缩在一起才能起到最好的压缩效果。

Kafka 采用了端到端的压缩：因为有『消息集』的概念，客户端的消息可以一起被压缩后送到服务端，并以压缩后的格式写入日志文件，以压缩的格式发送到 consumer，消息从 producer 发出到 consumer 拿到都被是压缩的，只有在 consumer 使用的时候才被解压缩，所以叫做『端到端的压缩』。Kafka支持GZIP和Snappy压缩协议。



### 示例：日志平台

- /var/log/apport.log 应用程序崩溃记录
- /var/log/apt/ 用 apt-get 安装卸载软件的信息
- /var/log/auth.log 登录认证的日志
- /var/log/boot.log 系统启动时的日志。
- /var/log/dmesg 包含内核缓冲信息(kernel ringbuffer)。在系统启动时，显示屏幕上的与硬件有关的信息
- /var/log/faillog 包含用户登录失败信息。此外，错误登录命令也会记录在本文件中
- /var/log/fsck 文件系统日志
- /var/log/kern.log 包含内核产生的日志，有助于在定制内核时解决问题
- /var/log/wtmp 包含登录信息。使用 wtmp 可以找出谁正在登陆进入系统，谁使用命令显示这个文件或信息等



## 版本控制

| Version | Action                   | Time       |
| ------- | ------------------------ | ---------- |
| 1.0     | Init                     | 2019-04-25T09:06:07-07:00|
