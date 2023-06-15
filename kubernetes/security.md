# Security in Kubernetes and Istio

## 目录

- [Security in Kubernetes and Istio](#security-in-kubernetes-and-istio)
  - [目录](#目录)
  - [云原生语境下的安全保证](#云原生语境下的安全保证)
    - [云原生层次模型](#云原生层次模型)
    - [开发环节的安全保证](#开发环节的安全保证)
    - [容器运行时的安全保证](#容器运行时的安全保证)
    - [Kubernetes的安全保证](#kubernetes的安全保证)
    - [集群的安全通信](#集群的安全通信)
    - [控制平面的安全保证](#控制平面的安全保证)
      - [NodeRestriction](#noderestriction)
    - [存储加密](#存储加密)
    - [Security Context](#security-context)
      - [Kubernetes提供了三种配置Security Context的方式](#kubernetes提供了三种配置security-context的方式)
  - [Taint](#taint)
  - [NetworkPolicy](#networkpolicy)
    - [依托于Calico的NetworkPolicy](#依托于calico的networkpolicy)
  - [零信任架构（ZTA）](#零信任架构zta)
  - [Istio的安全保证](#istio的安全保证)
    - [认证](#认证)
    - [鉴权](#鉴权)

## 云原生语境下的安全保证

- 安全保证时贯穿软件整个生命周期的各个部分
- 安全和效率有时候是相违背的
- 如何将两者统一起来，提升安全性的同时，保证效率
- 这需要我们将安全思想贯穿到软件开发运维的所有环节

### 云原生层次模型

软件开发的生命周期：开发 -> 分发 -> 部署 -> 运行

![dev life cycle](resources/dev-lifecycle.png)

### 开发环节的安全保证

- SaaS应用的12-factor原则的一些理念与云原生安全不谋而合
- 传统的安全三元素**CIA (Confidentiality, Integrity, Availability)**，在云原生安全中被充分应用。
  - Confidentiality: 保证数据的机密性
  - Integrity: 保证数据的完整性
  - Availability: 保证数据的可用性
  - Auditability: 保证数据的可审计性（**云原生安全**）
- 基础设施即代码（Infrastructure as Code, IaC）也与云原生的实践紧密相关。
- 这些方法和原则，都强调通过早期集成安全检测，以确保对过程的控制，使其按预期运行。
- 通过早期检测的预防性成本，降低后续的修复出成本，提升了安全的价值。

### 容器运行时的安全保证

### Kubernetes的安全保证

### 集群的安全通信

Kubernetes期望集群中所有的API通信在默认情况下使用TLS加密，大多数安装方法也允许创建所需的证书并且分发到集群组件中。

![enable tls in k8s communication](resources/k8s-tls-communication.png)

### 控制平面的安全保证

- 认证
- 授权
- 配额

#### NodeRestriction

### 存储加密

```yaml
apiVersion: v1
kind: EncryptionConfiguration
resources:
  - resources:
    - secrets
    providers:
    - aesgcm:
        keys:
        - name: key1
          secret: Zm9vYmFyYmF6
    - aescbc:
        keys:
        - name: key1
          secret: Zm9vYmFyYmF6
    - kms:
        name: my-kms-provider
        endpoint: https://my-kms-provider.example.com
        cachesize: 100
        timeout: 10s
    - identity: {}
```

### Security Context

Pod定义包含了一个安全上下文，用于描述允许它请求访问某个节点的特定Linux用户、获得特权或访问主机网络，以及允许它在主机节点上不受约束地运行的其他控件。

Pod安全策略可以限制哪些用户或服务账户可以提供的安全上下文设置。例如：Pod的安全策略可以限制卷挂载、特权、主机网络访问、特定的Linux用户或组、以及Linux能力。尤其是hostpath, 这些都是Pod应该控制的一些方面。

一般来说，大多数应用程序需要限制对主机资源的访问，他们可以在不访问主机信息的情况下成功以根进程（UID 0）运行。但是，考虑到与root用户相关的特权，在编写应用程序容器时，应该避免使用root用户。

类似地，希望阻止客户端应用程序逃避其容器的管理员，应该使用限制性的Pod安全策略。

#### Kubernetes提供了三种配置Security Context的方式

- Container-level Security Context: 仅应用到指定的容器
- Pod-level Security Context: 应用到Pod中的所有容器以及Pod级别的Volumes
- Pod Security Policy (PSP): 应用到集群内部所有的Pod以及Volumes

## Taint

## NetworkPolicy

### 依托于Calico的NetworkPolicy

## 零信任架构（ZTA）

## Istio的安全保证

### 认证

### 鉴权
