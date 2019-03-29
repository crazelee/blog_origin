---
title: Zookeeper 配置中心实现（Golang）
date: 2019-03-02 12:00:00
categories: Zookeeper
tags:
- Zookeeper
- 分布式
copyright: false
---

## 目标
一个乞丐版自更新配置中心，更新配置后，能在各个服务器实现更新

## 架构
![](/images/15514994232526.jpg)

<!--more-->

## 角色
* config-web: 配置后台，主要用于管理配置，增改配置
* config-agent: 监听配置，遇到变动后，自动拉取最新文件到本地
* config-sdk: 业务集成该sdk，用于读取配置

### config-web 配置后台
1. 持久存储为MySQL，也可以加一层缓存Redis，设置一个唯一的业务KEY，对应的ZK里的ZNode
    - 对于配置节点的操作，最终必须落盘，持久化存储于MySQL
2. 持久存储成功后，将配置的内容写入ZK集群中

以下是create节点的代码，set的同，这是简单的操作

```
package main

import (
	"fmt"
	. "go-zk/connect"

	"github.com/samuel/go-zookeeper/zk"
)

func main()  {
	conn := Connect()
	defer conn.Close()

	flags := int32(zk.FlagSequence)
	acl := zk.WorldACL(zk.PermAll)

	// create node
	path := PathConfig.ZNodePath +"/"+ "huodong-"
	data := []byte(`{"num":6.13,"strs":["a","b"]}`)
	createPath, err := conn.Create(path, data, flags, acl)
	if err != nil {
		panic(err)
	}
}

```

### config-agent 监控
1. 由于ZK的特性，能保持集群的一致性，以及提供了监听机制，在节点内容被改变时能提供回调
2. 在config-agent监听对应的业务节点
3. 监听的变动时，会有通知，例如，更改节点内容时，获取节点的内容，然后进行落盘，或者存到内存中都


```
package main

import (
	"fmt"
	"github.com/samuel/go-zookeeper/zk"
	. "go-zk/connect"
	"os"
	"sync"
)
type Watch struct {

}

func (this *Watch)ZkChildrenWatch(c *zk.Conn, path string)  {
	for {
		v, _, get_ch, err := c.ChildrenW(path)
		if err != nil {
			fmt.Println(err)
		}

		fmt.Printf("value of path[%s]=[%s].\n", path, v)

		for {
			select {
			case ch_event := <-get_ch:
				{
					fmt.Printf("%+v\n", ch_event)

					if ch_event.Type == zk.EventNodeCreated {
						fmt.Printf("has new node[%d] create\n", ch_event.Path)
					} else if ch_event.Type == zk.EventNodeDeleted {
						fmt.Printf("has node[%s] detete\n", ch_event.Path)
					} else if ch_event.Type == zk.EventNodeDataChanged {
						this.Callback(c, ch_event.Path)
					} else if ch_event.Type == zk.EventNodeChildrenChanged {
						fmt.Printf("children node change%+v\n", ch_event.Path)
					}
				}
			}
			break
		}
	}
}

func (this *Watch)ZkNodeWatch(c *zk.Conn, path string)  {
	for {
		v, _, get_ch, err := c.GetW(path)
		if err != nil {
			fmt.Println(err)
		}

		fmt.Printf("value of path[%s]=[%s].\n", path, v)

		for {
			select {
			case ch_event := <-get_ch:
				{
					if ch_event.Type == zk.EventNodeCreated {
						fmt.Printf("has new node[%d] create\n", ch_event.Path)
					} else if ch_event.Type == zk.EventNodeDeleted {
						fmt.Printf("has node[%s] detete\n", ch_event.Path)
					} else if ch_event.Type == zk.EventNodeDataChanged {
						this.Callback(c, ch_event.Path)
					}
				}
			}
			break
		}
	}
}

func (this *Watch)Callback(c *zk.Conn, path string) {
	data, _, err := c.Get(path)
	if err != nil {
		fmt.Println(err)
	}

	// create file
	fileName := PathConfig.LocalPath + path + ".json"
	os.Create(fileName)
	f, err := os.OpenFile(fileName, os.O_WRONLY|os.O_TRUNC, 0600)
	defer f.Close()
	if err != nil {
		fmt.Println(err.Error())
	} else {
		_,err=f.Write([]byte(data))
		fmt.Println(err)
		return
	}

	fmt.Print("Write File OK !!!")
}

func main()  {
	conn := Connect()

	// 监听所有子节点变化
	children, _, err := conn.Children(PathConfig.ZNodePath)
	if err != nil {
		panic(err)
	}
	fmt.Printf("%+v\n", children)

	w := Watch{}

	var wg sync.WaitGroup
	wg.Add(1)
	go func(path string) {
		w.ZkChildrenWatch(conn, path)
	}(PathConfig.ZNodePath)
	wg.Wait()



	// 监听节点内容变化
	//var wg sync.WaitGroup
	//wg.Add(len(children))
	//
	//for _, path := range children{
	//	path = PathConfig.ZNodePath + "/" + path
	//	go func(path string) {
	//		defer wg.Done()
	//		log.Print("Zookeeper Watcher Starting, ", path)
	//		w.ZkNodeWatch(conn, path)
	//	}(path)
	//}
	//wg.Wait()

}
```

### config-sdk 客户端加载配置
读取配置的方式很多样，两种思路：

1. 直接读取文件，由业务方直接读取，.json 、 .ini 、 .toml等
2. sdk可以与config-agent结合，如果读取文件加载配置失败，利用agent，重新主动拉一次文件到本地，实现文件的懒加载

## 效果展示


```
# This is Zookeeper config file.

title = "Zookeeper config file"

[zookeeper]
servers = ["10.00.85.70:2181", "10.00.80.191:2181", "10.00.97.239:2181"]
port = 2181
session_timeout = 500
enabled = true

[path]
znode_path = "/huodong/conf"
local_path = "/tmp/zookeeper"
```

1. 由于在本地测试，嫌麻烦就没有部署到服务器了，将locah_path分别改成"/tmp/zookeeper"、"/tmp/zookeeper1"、"/tmp/zookeeper2"，起三个进程

2. 连接到zk服务器，修改节点的内容
`set /huodong/conf/huodong-0000000001 '{"num":6.13,"strs":["a","b"]}'`
3. 看下本地文件就会生成对应的配置文件

![](/images/15515007969424.jpg)




