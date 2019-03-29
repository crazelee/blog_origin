---
title: Zookeeper 集群搭建
date: 2019-02-28 11:00:00
categories: Zookeeper
tags:
- Zookeeper
- 分布式
copyright: false
---

## 目标
用三台机器搭建一个Zookeeper集群

## 新增myid文件
用于配置当前IP对应的编号。
可以查看/conf/zoo.cfg的dataDir路径，在该路径新增myid，内容就只协商对应的编号就行
![](/images/15514264985081.jpg)


```
$ cat /tmp/zookeeper/myid
1
```

<!--more-->

## 编辑配置文件
`vi /conf/zoo.cfg` 在该配置文件下新增以下内容，每个服务器都需要增加

```
server.1=10.99.85.70:2888:3888
server.2=10.99.80.191:2888:3888
server.3=10.99.97.239:2888:3888
```

**说明**：server.A=B:C:D 

1. A 是一个数字，就是myid里的那个数字，表示这个是第几号服务器
2. B 是这个服务器的 ip 地址
3. C 用来集群成员的信息交换以及与集群中的 Leader 服务器交换信息
4. D 端口是在leader挂掉时专门用来进行选举Leader所用。

## 逐一启动服务
```
$ bin/zkServer.sh start

#ip 1
$ bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /zookeeper-3.4.8/bin/../conf/zoo.cfg
Mode: follower

#ip 2
$ bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /zookeeper-3.4.8/bin/../conf/zoo.cfg
Mode: leader

# ip 3
$ bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /zookeeper-3.4.8/bin/../conf/zoo.cfg
Mode: follower
```

## Leader选举情况
#### 步骤一：
启动服务器1时，投票给自己，服务器1获得了1票，没达到半数

#### 步骤二：
1. 启动服务器2时，投票给自己，服务器2获得1票，没达到半数；
2. 服务器1与2都是1票，重新发起投票；
3. 2的服务器ID大于1，1会重新投票给2，2还是仍然投票给自己
4. 服务器2获得了2票，达到半数，当选Leader



