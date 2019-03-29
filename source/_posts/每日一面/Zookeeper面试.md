---
title: Zookeeper 面试题
date: 2019-02-25 20:00:08
categories: Zookeeper
tags:
- Zookeeper
- 分布式
copyright: false
---

## 1. Zookeeper的用途，选举的原理是什么？

**用途**

1. 分布式锁
2. 服务注册和发现
    - 利用Znode和Watcher，可以实现分布式服务的注册和发现。最著名的应用就是阿里的分布式RPC框架Dubbo。
3. 共享配置和状态信息
    - Redis的分布式解决方案Codis（豌豆荚），就利用了Zookeeper来存放数据路由表和 codis-proxy 节点的元信息。同时 codis-config 发起的命令都会通过 ZooKeeper 同步到各个存活的 codis-proxy。 
4. 软负载均衡 

<!--more-->

**选举原理**

1. 每个 server 发出一个投票： 投票的最基本元素是（SID-服务器id,ZXID-事物id）
1. 接受来自各个服务器的投票
1. 处理投票：优先检查 ZXID(数据越新ZXID越大),ZXID比较大的作为leader，ZXID一样的情况下比较SID
1. 统计投票：这里有个过半的概念，大于集群机器数量的一半，即大于或等于（n/2+1）,我们这里的由三台，大于等于2即为达到“过半”的要求。这里也有引申到为什么 Zookeeper 集群推荐是单数。

## 2. zookeeper watch机制

| 客户端 | 服务端 |
| --- | --- |
| Main进程 |  |
| 创建ZK客户端，会创建connet网络连接通信线程，listener监听线程 |  |
| 通过connect线程将注册的监听事件发送给Zookeeper服务端 |  |
|  | 将监听事件添加到注册监听器列表 |
|  | 监听到有数据或路径变化，将消息发送给listener |
| listener线程内部调用process方法 |  |


![](/images/15510827433474.jpg)

## 3. Zookeeper 分布式锁
![](/images/15511531924836.jpg)


