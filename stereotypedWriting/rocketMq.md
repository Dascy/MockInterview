## RocketMQ学习

### MQ简介

MQ, Message Queue, 是一种提供消息队列服务的中间件，也称为消息中间件，是提供消息生成、存储、消费全过程的API系统

### MQ的作用

- 削峰限流    MQ可以将系统的超量请求暂存其中，以便系统后期可以慢慢进行处理，从而避免了请求的丢失或系统被压垮。
- 异步解耦   
- 数据收集

### 基本概念

- Producer： 消息生产者，负责产生消息，一般由业务系统负责产生消息
- Producer Group：消息生产者组，简单来说就是多个发送同一类消息的生产者称之为一个生产者
- Consumer：消息消费者，负责消费消息，一般是后台系统负责异步消费
- Consumer Group：消费者组，和生产者类似，消费同一类消息的多个 Consumer 实例组成一个消费者组
- Topic：主题，用于将消息按主题做划分，Producer将消息发往指定的Topic，Consumer订阅该Topic就可以收到这条消息
- Message：消息，每个message必须指定一个topic，Message 还有一个可选的 Tag 设置，以便消费端可以基于 Tag 进行过滤消息
- Tag：标签，子主题（二级分类）对topic的进一步细化,用于区分同一个主题下的不同业务的消息
- Broker：Broker是RocketMQ的核心模块，负责接收并存储消息，同时提供Push/Pull接口来将消息发送给Consumer。Broker同时提供消息查询的功能，可以通过MessageID和MessageKey来查询消息。Borker会将自己的Topic配置信息实时同步到NameServer
- Queue：Topic和Queue是1对多的关系，一个Topic下可以包含多个Queue，主要用于负载均衡，Queue数量设置建议不要比消费者数少。发送消息时，用户只指定Topic，Producer会根据Topic的路由信息选择具体发到哪个Queue上。Consumer订阅消息时，会根据负载均衡策略决定订阅哪些Queue的消息
- Offset：RocketMQ在存储消息时会为每个Topic下的每个Queue生成一个消息的索引文件，每个Queue都对应一个Offset记录当前Queue中消息条数
- NameServer：NameServer可以看作是RocketMQ的注册中心，它管理两部分数据：集群的Topic-Queue的路由配置；Broker的实时配置信息。其它模块通过Nameserv提供的接口获取最新的Topic配置和路由信息；各 NameServer 之间不会互相通信， 各 NameServer 都有完整的路由信息，即无状态。

### RocketMQ安装

#### 安装步骤

- 去[官网](https://rocketmq.apache.org/)下载压缩包。

<img src="/picture/rocketMq/officalWebsite.jpg">

<img src='/picture/rocketmq/download.jpg'>

- 上传到Linux服务器
- unzip命令解压
- 修改启动配置，原文件设置的jvm配置高，适当缩小。服务器资源重组可以忽略。

修改目录bin下的配置文件： runserver.sh、runbroker.sh,不然会报insufficient memory
修改runserver.sh 中原有内存配置，更改为

```shell
JAVA_OPT="${JAVA_OPT} -server -Xms256m -Xmx256m"
```

#### RocketMQ目录介绍

- bin   常用的启动脚本
- config  配置文件


- lib  依赖jar包

#### MQ命令

##### 启动NameServer

```shell
#启动NameServer
nohup sh mqnamesrv &
#查看启动日志
tail -f ~/logs/rocketmqlogs/namesrv.log
```

##### 启动Broker

```shell
#启动Broker
nohup sh mqbroker -n localhost:9876 autoCreateTopicEnable=true -c ../conf/broker.conf &
#查看启动日志
tail -f ~/logs/rocketmqlogs/broker.log 
```

##### 发送消息

```shell
#设置环境变量
export NAMESRCV_ADDR=192.168.1.102:9876
#使用安装包demo发送消息
sh tools.sh org.apache.rocketmq.example.quickstart.Producer
```

##### 消费消息

```shell
#设置环境变量
export NAMESRCV_ADDR=192.168.1.102:9876
#使用安装包demo接收
sh tools.sh org.apache.rocketmq.example.quickstart.Consumer
```

##### 关闭服务

```shell
sh mqshutdown broker
sh mqshutdown namesrv
```

