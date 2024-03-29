# etcd
Etcd是CoreOS基于Raft开发的分布式key-value存储，可用于服务发现、共享配置以及一致性 保障(如数据库选主、分布式锁等)。

在分布式系统中，如何管理节点间的状态一直是一个难题，etcd像是专门为集群环境的服务发现 和注册而设计，它提供了数据TTL失效、数据改变监视、多值、目录监听、分布式锁原子操作等 功能，可以方便的跟踪并管理集群节点的状态。

* 键值对存储:将数据存储在分层组织的目录中，如同在标准文件系统中 
* 监测变更:监测特定的键或目录以进行更改，并对值的更改做出反应
* 简单: curl可访问的用户的API(HTTP+JSON)
* 安全: 可选的SSL客户端证书认证
* 快速: 单实例每秒1000次写操作，2000+次读操作 • 可靠: 使用Raft算法保证一致性

## 主要功能
* 基本的key-value存储
* 监听机制
* key的过期及续约机制，用于监控和服务发现
* 原子Compare And Swap和Compare And Delete，用于分布式锁和leader选举

## 使用场景
* 也可以用于键值对存储，应用程序可以读取和写入 etcd 中的数据 \
* etcd 比较多的应用场景是用于服务注册与发现
* 基于监听机制的分布式异步系统

## 键值对存储
etcd 是一个键值存储的组件，其他的应用都是基于其键值存储的功能展开。
* 采用kv型数据存储，一般情况下比关系型数据库快。
* 支持动态存储(内存)以及静态存储(磁盘)。
* 分布式存储，可集成为多节点集群。
* 存储方式，采用类似目录结构。(B+tree)
  * 只有叶子节点才能真正存储数据，相当于文件。
  * 叶子节点的父节点一定是目录，目录不能存储数据。

## 服务注册与发现

* 强一致性、高可用的服务存储目录。
  * 基于 Raft 算法的 etcd 天生就是这样一个强一致性、高可用的服务存储目录。
* 一种注册服务和服务健康状况的机制。
  * 用户可以在 etcd 中注册服务，并且对注册的服务配置 key TTL，定时保持服务的心跳以达 到监控健康状态的效果。

## 消息发布与订阅
* 在分布式系统中，最适用的一种组件间通信方式就是消息发布与订阅。
* 即构建一个配置共享中心，数据提供者在这个配置中心发布消息，而消息使用者则订阅他们关心的主题，一旦主题有消息发布，就会实时通知订阅者。
* 通过这种方式可以做到分布式系统配置的集中式管理与动态更新。
* 应用中用到的一些配置信息放到etcd上进行集中管理。
* 应用在启动的时候主动从etcd获取一次配置信息，同时，在etcd节点上注册一个Watcher并等待，以后每次配置有更新的时候，etcd都会实时通知订阅者，以此达到获取最新配置信息的目的。

## 核心:TTL & CAS
TTL(time to live)指的是给一个key设置一个有效期，到期后这个key就会被自动删掉，这在 很多分布式锁的实现上都会用到，可以保证锁的实时有效性。

Atomic Compare-and-Swap(CAS)指的是在对key进行赋值的时候，客户端需要提供一些条 件，当这些条件满足后，才能赋值成功。这些条件包括:

* prevExist:key当前赋值前是否存在
* prevValue:key当前赋值前的值
* prevIndex:key当前赋值前的Index

key的设置是有前提的，需要知道这个key当前的具体情况才可以对其设置。

## Raft协议
Raft协议基于quorum机制，即大多数同意原则，任何的变更都需超过半数的成员确认.

![](resources/raft.png)

[理解Raft协议](http://thesecretlivesofdata.com/raft/)
