# Security in Kubernetes and Istio

## 目录

- [云原生语境下的安全保证](#云原生语境下的安全保证)
  - [云原生层次模型](#云原生层次模型)
  - [开发环节的安全保证](#开发环节的安全保证)
  - [容器运行时的安全保证](#容器运行时的安全保证)
  - [Kubernetes的安全保证](#kubernetes的安全保证)
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

## Taint

## NetworkPolicy

### 依托于Calico的NetworkPolicy

## 零信任架构（ZTA）

## Istio的安全保证

### 认证

### 鉴权
