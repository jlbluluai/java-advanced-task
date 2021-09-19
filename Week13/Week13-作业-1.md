# Table of Contents

* [Week13-作业-1](#week13-作业-1)
    * [题目](#题目)
    * [解题](#解题)
        * [环境准备](#环境准备)
        * [安装](#安装)
        * [查看集群情况](#查看集群情况)
        * [测试](#测试)
        * [Spring Boot 集成 kafka](#spring-boot-集成-kafka)

# Week13-作业-1

## 题目

> 搭建一个 3 节点 Kafka 集群，测试功能和性能；实现 spring kafka 下对 kafka 集群的操作

## 解题


### 环境准备

[官网下载地址](http://kafka.apache.org/downloads)

我这里下载了[kafka_2.12-2.8.0.tgz](https://www.apache.org/dyn/closer.cgi?path=/kafka/2.8.0/kafka_2.12-2.8.0.tgz)

### 安装

解压后在kafka根目录准备三份配置文件，内容如下：

```
## 集群做区分
broker.id=1
num.network.threads=3
num.io.threads=8
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600
## 集群做区分
log.dirs=/Users/zhuweijie/logs/kafka/kafka-logs-1
num.recovery.threads.per.data.dir=1
offsets.top1c.replication.factore=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1
log.retentlon.hours=168
log.segnent.bytes=1073741824
1og.retention.check.interval.ms=300000
zookeeper.connection.timeout.ms=6000000
delete.topic.enable=true
group.initial.rebalance.delay.ms=0
message.max.bytes=5000000
replica.fetch.max.bytes=5000000
#三个文件分别改为9001，9002，9003 
listeners=PLAINTEXT://localhost:9001
broker.list=localhost:9001,localhost:9002,localhost:9003
zookeeper.connect=localhost:2181
```

然后启动三个终端页执行下面三个启动命令（确保有zookeeper已经启动）：

```
bin/kafka-server-start.sh server_1.properties

bin/kafka-server-start.sh server_2.properties

bin/kafka-server-start.sh server_3.properties
```

### 查看集群情况

我这里本地安装了`CMAK`，查看如下：

![](http://img.yelizi.top/b25df7f3-13e9-4215-b5a8-2439d7830490.jpg$xyz)


### 测试

**创建topic**

```
bin/kafka-topics.sh --zookeeper localhost:2181 --create --topic test32 --partitions 3 -replication-factor 2
```

**分别启动生产者和消费者**

```
bin/kafka-console-producer.sh --bootstrap-server localhost:9003 --topic test32 

bin/kafka-console-consumer.sh --bootstrap-server localhost:9001 --topic test32
```

**测试消息消费**

![](http://img.yelizi.top/8d7b0abe-4388-4cf1-bb14-19d858879df0.jpg$xyz)

![](http://img.yelizi.top/5ab33445-b4df-4b78-8fb3-7dd0412b970a.jpg$xyz)


**测试生产者性能**

```
bin/kafka-producer-perf-test.sh --topic test32 --num-records 100000 --record-size 1000 --throughput 200000 --producer-props bootstrap.servers=localhost:9002
```

![](http://img.yelizi.top/3869238c-5c32-4802-9372-244baa8e43bd.jpg$xyz)

每秒到7w多


**测试消费者性能**

```
bin/kafka-consumer-perf-test.sh --bootstrap-server localhost:9002 --topic test32 -fetch-size 1048576 --messages 100000 --threads 1
```

![](http://img.yelizi.top/f8e62e74-5205-4625-a60f-266661e07aa4.jpg$xyz)

每秒处理15w左右


### Spring Boot 集成 kafka

[项目地址](https://github.com/jlbluluai/xyz-study/tree/master/xyz-study-kafka-spring-demo)

启动后命令行执行如下发送消息：

```
curl localhost:8080/kafka/demo/send?topic=xyz.test\&message=hello
```

日志打印如下：

```
2021-09-15 15:11:49.855  INFO 56678 --- [ad | producer-1] com.xyz.study.service.KafkaProducerDemo  : send msg success, topic=xyz.test, message="hello", result=SendResult [producerRecord=ProducerRecord(topic=xyz.test, partition=null, headers=RecordHeaders(headers = [], isReadOnly = true), key=null, value="hello", timestamp=null), recordMetadata=xyz.test-0@9]
2021-09-15 15:11:49.855  INFO 56678 --- [ntainer#0-0-C-1] com.xyz.study.service.KafkaConsumerDemo  : receive message, topic=xyz.test, value="hello"
```

成功发送消息并接收到消息