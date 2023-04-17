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
  * 总计 100,000 物理节点
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


