## Istio架构
![](resources/istio_architecture.png)
## 数据平面Envoy

## 控制平面

## 流量管理

## 网络
Service Mesh 涉及的网络栈
![](resources/service_mesh_network.png)

## Istio多集群
### 跨地域流量管理的挑战
* 采用多活数据中心的网络拓扑，任何生产应用都需要完成跨三个数据中心的部署。
* 为满足单集群的高可用，针对每个数据中心，任何应用都需进行多副本部署，并配置负载均衡。
* 以实现全站微服务化，但为保证高可用，服务之间的调用仍以南北流量为主。
* 针对核心应用，除集群本地负载均衡配置以外，还需配置跨数据中心负载均衡并通过权重控制将 99%的请求转入本地数据中心，将1% 的流量转向跨地域的数据中心。

![](resources/traffic_management_multiple_region.png)

### 规模化带来的挑战(eBay)
* 3主数据中心，20 边缘数据中心 ，100+ Kubernetes 集群
* 规模化运营 Kubernetes 集群
  * 总计100,000物理节点
  * 单集群物理机节点规模高达5,000
* 业务服务全面容器化，单集群
  * Pod 实例可达 100,000
  * 发布服务 5,000-10000
* 单集群多环境支持
  * 功能测试、集成测试、压力测试共用单集群
  * 不同环境需要彼此隔离
* 异构应用
  * 云业务，大数据，搜索服务
  * 多种应用协议
  * 灰度发布
* 日益增长的安全需求
  * 全链路TLS
* 可见性需求
  * 访问日志
  * Tracing

### 多集群部署
* Kubernetes 集群联邦
  * 集群联邦 APIServer 作为用户访问 Kubernetes 集群入
  * 所有 Kubernetes 集群注册至集群联邦
* 可用区
  * 数据中心中具有独立供电制冷设备的故障域
  * 同一可用区有较小网络延迟
  * 同一可用区部署了多个 Kubernetes 集群
* 多集群部署
  * 同一可用区设定一个网关集群
  * 网关集群中部署 Istio Primary
  * 同一可用区的其他集群中部署 Istio Remote
  * 所有集群采用相同 RootCA
  * 相同环境 TrustDomain 相同
* 东西南北流量统一管控
  * 同一可用区的服务调用基于 Sidecar
  * 跨可用区的服务调用基于 Istio Gateway

![](resources/istio_fedreation_management.png)

### 入站流量架构 L4 + L7
为不同应用配置独立的网关服务以方便网络隔离。

基于 IPVS/XDP 的 Service Controller :

* 四层网关调度；
* 虚拟IP地址分配；
* 基于 IPIP 协议的转发规则配置；
* 基于 BGP 的IP路由宣告；
* 在Ingress Pod 中配置 Tunnel 设备，并绑定虚拟 IP 地址以卸载IPIP包。

![](resources/input_traffic_arch.png)

### 单网关集群多环境支持
![](resources/single_gateway_mutiplue_environment.png)

### 应用高可用接入方案
![](resources/istio_ha.png)


#### 创建WorkloadEntry
#### 定义ServiceEntry
#### 定义基于Locality的流量转发规则

### 应对规模化集群的挑战
Istio xDS 默认发现集群中所有的配置和服务状态，在超大规模集群中，Istiod 或者 Envoy 都承受比较大的压力。

* 集群中的有 10000Service，每个 Service 开放 80 和443 两个端口，Istio 的 CDS 会discover 出20000个Envoy Cluster.
* 如果开启多集群，Istio 还会为每个cluster 创建符合域名规范的集群。
* Istio 还需要发现 remote cluster 中的 Service, Endpoint 和Pod 信息，而这些信息的频繁变更，会导致网络带宽占用和控制面板的压力都很大。

meshConfig 中控制可见性：

* defaultServiceExportTo:
* defaultVirtualServiceExportTo:
* defaultDestinationRuleExportTo:

通过 Istio 对象中的exportT。属性覆盖默认配置。

#### Istiod自身的规模控制

社区新增加了discoverySelector的支持，允许Istiod 只发现添加了特定 label的namespaces 下的Istio 以及 Kubernetes 对象。

但因为 Kubernetes框架的限制，改功能依然要让 Istiod 接收所有配置和状态变更新细，并旦在Istiod 中进行对象过滤。在超大集群规模中，并木降低网络带克占用和 1stiod 的处理压力。

需要继续寻求从 Kubernetes Server 端过滤的解决方案。

### 基于联邦的统一流量模型
![](resources/traffic_model_management.png)

### 统一流量模型 - NameService
![](resources/traffic_model_nameservice.png)

### AccessPoint 控制器
PlacementPolicy 控制，用户可以选择目标集群来完成流量配置，甚至可以选择关联的FederatedDeployment 对象，使得 Access Point 自动发现目标集群并完成配置。

完成了状态上报，包括网关虛拟 IP 地址，网关 FQDN，证书安装状态以及版本信息，路由策略是否配置完成等。这补齐了Istio 自身的短板，使得任何部署在 Istio 的应用的网络配置
状态一目了然。

发布策路控制，针对多集群的配置，可实现单集群的灰度发布，并旦能够自动質停发布，管理员验证单个集群的变更正确以后，再继续发布。通过此机制，避免因为全局流量变更产生
的故障。

不同域名的 AccessPoint 可拥有不同的四层网关虛拟IP 地址，以实现基于 IP 地址的四层网
络隔离。

控制器可以基于 AccessPoint 自动创建 WorkloadEntry，并设置 Locality 信息。

### 展望
* 全网构建基于Mesh的流量管理
* 在用户无感知的前提下将南北流量转成东西流量
* 数据平台加速Cilium