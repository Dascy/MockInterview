## RocketMQ学习

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

