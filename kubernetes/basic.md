## Kubernetes Components
There are five main components in kubernetes cluster, included "API Server", "Controller Manager", "Scheduler", "Kubelet" and "Kube-Proxy", and storage is "ETCD"
### API Server
API Server 提供集群管理的 REST API 接口, 包括：认证(Authentication)，授权(Authorization)，准入Admission(Mutating & Valiating)。其他模块通过API Server查询或修改数据，只有APIServer才直接操作etcd。APIServer 提供 etcd 数据缓存以减少集群对 etcd 的访问。
API Server展开
![](resources/APIServer.png)
### Controller Manager
Controller Manager 是一个控制器的集合，负责管理集群中的各种资源，确保Kubernetes遵循声明式系统规范，确保系统的真实状态（Actual State）与用户定义的期望状态（Desired State）一致。
控制器的工作流程：
![](resources/controller_manager_informer.png)
Informer工作机制：
![](resources/informer_mechanism.png)
### Scheduler
Scheduler 负责将 Pod 调度到合适的节点上，调度的策略是通过 Scheduler Policy 配置的，可以通过 Scheduler Policy 配置 Pod 的优先级，以及 Pod 调度的策略。
调度阶段：
* Predict
* Prioritize
* Bind
### Kubelet
Kubelet 是 Kubernetes 中的 Agent，负责管理 Pod 的生命周期，包括创建、启动、停止、删除 Pod，以及 Pod 的健康检查。
### Kube-Proxy
Kube-Proxy 是 Kubernetes 中的网络代理，负责实现 Kubernetes Service 的网络代理功能，包括负载均衡、服务发现等。
## Kubernetes HA Levels
![](resources/kubernetes_ha_levels.png)
