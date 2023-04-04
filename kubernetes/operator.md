### Kubernetes 对象是可扩展的，扩展的方式:
* **基于原生对象**
  * 生成types对象，并通过client-go 生成相应的clientset, lister, informer.
  * 实现对象的registery backend，即定义对象任何存储进etcd.
  * 注册对象的scheme至apiserver.
  * 创建该对象的apiservice生命，注册该对象所对应的api handler。
  * 基于原生对象往往需要通过aggregation apiserver把不同对象组合起来。
* **基于 CRD**
    * 在不同应用业务环境下，对于平台可能有一些特殊的需求，这些需求可以抽象为Kubernetes 的扩展资源，而Kubernetes CRD(CustomResourceDefinition）为这样的需求提供了轻量级的机制，保证新的资源的快速注册和使用。

### 如何使用CRD
用户向Kubernetes API 服务注册一个带特定schema 的资源，并定义相关API

* 注册一系列该资源的实例
* 在Kubernetes 的其他资源对象中引用这个新注册资源的对象实例
* 用户自定义的controller例程需要对这个引1用进行释义和实施，让新的资源对象达到预期的状态

### 基于CRD的开发过程
借助 Kubernetes RBAC 和authentication 机制来保证该扩展资源的 security、access control, authentication 和 multitenancy.

将扩展资源的数据存储到 Kubernetes 的etcd 集群。

借助 Kubernetes 提供的 controller 模式开发框架，实现新的 controller，并借助 APIServer监听 etcd 集群关于该资源的状态并定义状态变化的处理逻辑。

该功能可以让开发人员扩展添加新功能，更新现有的功能，并旦可以自动执行一些管理任务，这些自定义的控制器就像 Kubernetes 原生的组件一样，Operator 直接使用 Kubernetes API进行开发，也就是说他们可以根据这些控制器内部编写的白定义规则来监控集群、更改
Pods/Services、对正在运行的应用进行扩缩容。

### 控制器模式
![](resources/controller_manager_informer.png)
