# Kubernetes 组件

当你部署了 Kubernetes, 你就获得一个集群。

一个 K8s 集群由一些列被称为 Node 的工人机器组成，这些节点运行容器化的应用。每一个集群至少拥有一个工人节点。

工作 Node 托管 Pods, 这些 Pod 是应用程序工作负载的组成部分。**control plane** 管理在集群中的 worker nodes 和 Pods。 在生产环境中，control plane 通常运行在多台计算机上，同上通常运行在多个节点上，提高容错性和高可用性。

本篇文档概览描述了各种各样的你需要构建一个完整和可工作的K8s集群所需要的组件。

这是一张 K8s 集群的图片，描述了各个组件如何关联在一起。
![](../../images/components-of-kubernetes.svg)


## Control Plane 组件
Control Plane 的组件对集群进行全局决策(例如，调度), 同样也会探测和相应集群时间(例如，启动一个新的Pod当部署的副本不满足的时候)

Control plane 组件可以在集群的任意一台机器上。为了简单起见，典型地设置一个脚本在同一台机器上启动所有的 control plane 组件，并且在该机器上不再运行任何的用户容器。有关跨多个虚拟机运行 control plane 设置的事例，清参考阅读使用kubeadm 创建高可用性集群。

### kube-apiserver
API server 是 K8s control plane 的一个组件，用于暴露 K8s API。API server 是 K8s control plane 的前端。

kube-apiserver是一个主要实现的 Kubernetes API 服务器。kube-apiserver 被设计来水平的扩展 —— 也就是说，它通过部署更多的实例来扩展。你可以运行一系列的 kube-apiserver 实例，同时在多个实例之间保持流量均衡。

### etcd
etcd 是一个一致性，高性能的 key-value 存储，被用于K8s所有集群数据的后端存储。

如果你使用 etcd 作为 K8s 集群的后端存储，确保你有为这些数据备份的计划。

你可以更加深入的了解 etcd, 通过阅读它的[官方文档](https://etcd.io/docs/)

### kube-schedular
Control plane 组件监视那些没有分配节点的新创建的Pod, 并选择一个节点以在其上运行该Pod。

调度决策考虑的因素包括：个体和集体的资源需求，硬件/软件/策略的限制，亲和力和反亲和力规范, 数据局部性，内部工作负载接口，以及截止时间。

### kube-controller-manager
Control Plane 组件运行 controller 进程

逻辑上，每一个 controller 是一个独立的进程，但是为了减少复杂性，所有的controller 被编译到单独的二进制，并且运行在一个单一的进程中。

一些 controller 的类型如下：

1. Node controller: 负责发现和相应当 Node 挂掉了

2. Job controller: 观察 job 对象，这些 job 对象代表一次性任务，然后创建一个 Pod 来运行这些任务直到完成。

3. Endpoints controller: 填充端点对象(这就是，加入 Services & Pods)

4. Service Account & Token controllers: 为新的命名空间创建默认账号和 API 获取 tokens。

### cloud-controller-manager
一个 Kubernetes controller plane 组件嵌入云特定的控制逻辑。cloud controller manager 让你链接你的集群到你云提供的 API，并将与云平台交互的组件与集群交互的组件分离开来。

could-controller-manager 仅仅运行在特定于您的云提供商的 controller。如果您是在自己的场所或PC的学习环境中运行 Kubernetes, 则该集群没有 could controller manager。

通过 kube-controller-manager, cloud-controller-manager 将一些列逻辑独立的控制回路组合成一个独立的二进制，让你能够运行作为一个单独的进程。你可以水平的扩展来提升性能来帮助容忍错误。

以下的 controllers 可以有云提供商依赖：

1. Node Controller: 检查云提供商来决定一个 在云上的node 是否已经被删除在 node 停止响应之后。

2. Roue controller: 为了设置在底层云基础设施的路由。

3. Service controller: 为了创建，更新和删除提供商负载均衡

## Node 组件

Node 组件运行在每一个 node 上，维护正在运行的 pod, 并且题哦功能 kubernetes 的运行环境。

### kubelet
kubelet 是一个代理运行在集群的每一个节点上。它确保每一个容器正在一个 Pod 上运行。

Kubelet需要一组PodSpec来提供各种各样的机制来确保被PodSpec定义P的容器是正在健康的运行。kubelet 不会管理那些不是由 Kubernetes 创建的容器。

### kube-proxy
kube-proxy 是一个网络代理运行在你集群的每一个节点上，实现了一部分 Kubernetes Service 概念。

kube-proxy 维护节点上的网络规则。这些网络规则允许网络和你来自你集群内部或外部的网络会话的Pods进行通信

kube-proxy 使用操作系统包过滤层，如果包过滤层有且可用的话。否则，kube-proxy 就本身转发流量。

### Container runtime
container runtime 是一个软件，负责运行容器。

K8s 机制一系列的容器运行时：Docker, containerd, CRI-O, 以及任何实现了 [Kubernetes CRI(Container Runtime Interface)](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-node/container-runtime-interface.md)


## Addon 附件
Addons 使用 K8s 资源(DaemonSet, Deployment等)来实现集群特性。因为提供集群级别功能。所以插件的命名空间资源属于 kube-system 命名空间。

下面将描述选择的插件；可用插件的扩展列表，请看[Addons](https://kubernetes.io/docs/concepts/cluster-administration/addons/)

### DNS
相比较与其他插件并不是严格的需要，但是所有的 K8s 应该需要 cluster DNS, 因为很多地方需要它。

Cluster DNS 是一个 DNS 服务，除了在你环境中其他 DNS 服务，Cluster DNS 为 K8s Service 提供 DNS 记录。

由 K8s 创建的容器自动在它们的搜索中包含该 DNS server。

### Web UI(面板)
面板是一个用于 K8s 集群的基于Web 的通用UI。它运行用户来管理和问题排查在集群中的应用，同样包括集群本身。

### Container Resource Monitoring
Container Resource Monitoring 记录关于数据中心容器的一般时间序列指标，并且为该数据提供UI用于浏览。

### Cluster-level Logging
一个集群级别日志机制是为了负责保存容器日志到一个中心日志存储，并且提供对应的搜索和浏览接口。