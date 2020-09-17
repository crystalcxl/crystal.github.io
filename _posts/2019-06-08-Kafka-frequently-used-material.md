---
layout: post
title:  "Kafka资料整理"
date:   2019-06-12 11:40:18 +0800
categories: BigData
tags: kafka
author: crystalcxl
---

[TOC]

记录下kafka常用命令







# 1.安装部署

## 1.1 环境依赖

Jdk 1.8以上

## 1.2 安装步骤

下载源码

1、tar -xzf kafka_2.11-2.0.0.tgz

2、cd kafka_2.11-2.0.0

 

启动自带zookeeper

bin/zookeeper-server-start.sh config/zookeeper.properties

 

启动kafka

bin/kafka-server-start.sh config/server.properties

## 1.3 参数调优

### ***\*1.3.1 部署数目的调优\****

l Broker数

Broker数，意味着集群的节点数，broker越多，producer和consumer的吞吐量越大，系统的最大延时越小。

 

l 分区数

1.越多的分区可以提供更高的吞吐量

2.越多的分区需要打开更多的文件句柄

3.更多地分区会导致更高的不可用性

4.越多的分区可能增加端对端的延时

5.越多的分区意味着服务端/客户端需要更多的内存

   当partition小于broker数目时：partition数量越多，producer和consumer的吞吐量越高，最大延时越低;当partition大于broker数目时，producer和consumer的吞吐量趋于平稳，甚至有所下降，最大延时，总体也保持平稳，最后甚至有所上升。

 

l 备份数

备份数越大，produder吞吐量越低，consumer吞吐量则与备份数无关

 

l consumer数

 在partition一样的时候，当consumer数小于或等于partition数时，consumer-group的吞吐量越大;当consumer数大于partition数时，consumer-group的吞吐量不会增加。因为，topic下的一个分区只能被同一个consumer group下的一个consumer 线程或进程来消费。

 

 

l Producer数

分区一定的情况下，producer数小于分区数基础上，增加producer数量可以提高吞吐量。

### ***\*1.3.2 参数配置的调优\****

l 吞吐量(Throughput)

producer端

​         batch.size:增加到100000 - 200000（默认为16384）

​         linger.ms: 增加到10 - 100 (默认为0)

​         compression.type=lz4(默认为None)

​         acks=1(默认为1)

​         retries=0(默认为0)

​         buffer.memory：如果分区数很多则适当增加 (默认为32MB)

Consumer端

​         fetch.min.bytes:增加到10 ~ 100000 (默认为1)

 

l 延时(Latency)

Producer端

​         linger.ms=0(默认为0)

​         compression.type=none(默认为None)

​         acks=1(默认为1)

Broker端

​         num.replica.fetchers：如果ISR频繁进出或follower无法追上leader的情况则适当增加该值，但通常不要超过CPU核数+1(默认为1)

Consumer端

​         fetch.min.bytes=1(默认为1)

 

l 持久性(Durability)

 

Producer端

​         replication.factor=3 每个topic的配置

​         acks=all(默认为1)

​         retries:1或更大的值(默认为0)

​         max.in.flight.requests.per.connection= 1 (默认为5)防止消息乱序

Broker端

​         default.replication.factor=3(默认为1)

​         auto.create.topics.enable=false(默认为true)

​         min.insync.replicas=2(默认为1) ，即设置为replication factor - 1，topic的配置可以覆盖

​         unclean.leader.election.enable=false(默认为true)，topic的配置可以覆盖

​         broker.rack: 如果有机架信息，则最好设置该值，保证数据在多个rack间的分布性以达到高持久化(默认为null)

​         log.flush.interval.messages和log.flush.interval.ms: 对TPS比较低的topic，则推荐设置成比较低的值，比如1 (默认允许操作系统控制刷新)， topic的配置可以覆盖

Consumer端

​         auto.commit.enable=false (默认为true)自己控制位移

 

l 可用性(Availability)

Broker端

​        unclean.leader.election.enable=true(默认为true)，topic的配置可以覆盖

​       min.insync.replicas=1(默认为1) ，topic的配置可以覆盖

​       num.recovery.threads.per.data.dir=log.dirs(默认为1)中配置的目录数

Consumer端

​       session.timeout.ms：尽可能的低(默认为1000)

 

 

### ***\*1.3.3 操作系统的调优\****

禁掉swap

JVM推荐使用最新的G1来代替CMS作为垃圾回收

JDK 版本为1.8

磁盘

网卡

 

 

 

## 1.4 自动化部署

# 2.基础性能

## 2.1 容量评估

Kafka的容量和硬盘相关 

## 2.2 性能

消息大小

发送频率

备份数量

分区数量

 

1)  kafka轻松支持每秒发送50万条消息以上，消息体越大支持的分送频率越低

2)  kafka吞吐率比较大，瓶颈一般是网络而不是磁盘。

3)  分区多少直接影响吞吐率，大致正比关系。

4)  备份数量不直接影响吞吐率。

 

 

## 2.3 kafka使用常用命令

 

创建topic

kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test

 

展示topic

kafka-topics.sh --list --zookeeper localhost:2181

 

描述topic

kafka-topics.sh --describe --zookeeper localhost:2181 --topic my-replicated-topic

 

修改partition

kafka-topics.sh --zookeeper localhost:2181 --alter --topicmytopic --partition 3

 

生产者:

kafka-console-producer.sh --broker-list localhost:9092 --topic test

 

消费者:

kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning

 

删除topic

 1.kafka-topics.sh –zookeeper localhost:2181 –delete –topic mytopic 

 2.client删除

1）打开zookeeper client 执行：./bin/zkCli.sh -server 192.168.1.11:2181 

2）在Zookeeper客户端下执行命令 ls /brokers/topics 

3)在Zookeeper客户端下执行命令 rmr /brokers/topics/mytopic 其中，mytopic为要删除的topic。 

4)验证是否删除： ls /config/topics ls /brokers/topics ls /admin/delete_topics Admin/delete_topics

 

 

# 3.监控

## 3.1进程监控

使用 Systemd监控kafka进程是否存在，并自动拉起

 

## 3.2业务监控

通过研究，发现主流的三种kafka监控程序分别为：

l Kafka Web Conslole

l Kafka Manager

l KafkaOffsetMonitor

现在依次介绍以上三种工具：

### ***\*Kafka Web Conslole\****

使用Kafka Web Console，可以监控：

 

l Brokers列表

l Kafka 集群中 Topic列表，及对应的Partition、LogSize等信息

l 点击Topic，可以浏览对应的Consumer Groups、Offset、Lag等信息

l 生产和消费流量图、消息预览…

![img](file:///C:\Users\cxl\AppData\Local\Temp\ksohtml18448\wps1.jpg) 

 

程序运行后，会定时去读取kafka集群分区的日志长度，读取完毕后，连接没有正常释放，一段时间后产生大量的socket连接，导致网络堵塞。

### ***\*Kafka Manager\****

雅虎开源的Kafka集群管理工具:

l 管理几个不同的集群

l 监控集群的状态(topics, brokers, 副本分布, 分区分布)

l 产生分区分配(Generate partition assignments)基于集群的当前状态

l 重新分配分区

![img](file:///C:\Users\cxl\AppData\Local\Temp\ksohtml18448\wps2.jpg) 

### ***\*KafkaOffsetMonitor\****

l KafkaOffsetMonitor可以实时监控：

l Kafka集群状态

l Topic、Consumer Group列表

l 图形化展示topic和consumer之间的关系

l 图形化展示consumer的Offset、Lag等信息