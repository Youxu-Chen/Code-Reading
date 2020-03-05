# Kubernetes架构

[TOC]

## Kubernetes整体架构

![整体架构](https://img-blog.csdnimg.cn/20190324224238388.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzUzODQxOQ==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190324224256359.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzUzODQxOQ==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190324224318400.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzUzODQxOQ==,size_16,color_FFFFFF,t_70)



上面三幅图分别介绍了K8s的整体组织架构、Master节点架构和Node节点架构示意图。K8s集群采用Master/Slave架构，Master节点主要负责处理用户请求、集群扩缩容、任务调度和状态维护等功能；Slave节点（即Node节点）负责运行用户任务，并定期与Master节点交互，保持集群始终维护在正常状态。Node节点上运行的每个任务均封装在pod里，pod可以理解成为运行在容器上的job封装的环境。pod是K8s里任务调度的最小单元。



![在这里插入图片描述](https://img-blog.csdnimg.cn/20190324224152631.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzUzODQxOQ==,size_16,color_FFFFFF,t_70)

接下来主要介绍K8s中Master节点和Node节点中各组件及其功能。

## Master节点组件

### kube-apiserver

[kube-apiserver](https://kubernetes.io/docs/admin/kube-apiserver)用于暴露Kubernetes API。任何的资源请求/调用操作都是通过kube-apiserver提供的接口进行。请参阅[构建高可用群集](https://kubernetes.io/docs/admin/high-availability)。

### etcd

[kube-apiserver](https://kubernetes.io/docs/admin/kube-apiserver)用于暴露Kubernetes API。任何的资源请求/调用操作都是通过kube-apiserver提供的接口进行。请参阅[构建高可用群集](https://kubernetes.io/docs/admin/high-availability)。

### kube-controller-manager

[kube-controller-manager](https://kubernetes.io/docs/admin/kube-controller-manager)运行管理控制器，它们是集群中处理常规任务的后台线程。逻辑上，每个控制器是一个单独的进程，但为了降低复杂性，它们都被编译成单个二进制文件，并在单个进程中运行。

这些控制器包括：

- [节点（Node）控制器](http://docs.kubernetes.org.cn/304.html)。
- 副本（Replication）控制器：负责维护系统中每个副本中的pod。
- 端点（Endpoints）控制器：填充Endpoints对象（即连接Services＆Pods）。
- [Service Account](http://docs.kubernetes.org.cn/84.html)和Token控制器：为新的[Namespace](http://docs.kubernetes.org.cn/242.html) 创建默认帐户访问API Token。

### cloud-controller-manager

云控制器管理器负责与底层云提供商的平台交互。云控制器管理器是Kubernetes版本1.6中引入的，目前还是Alpha的功能。

云控制器管理器仅运行云提供商特定的（controller loops）控制器循环。可以通过将`--cloud-provider` flag设置为external启动kube-controller-manager ，来禁用控制器循环。

cloud-controller-manager 具体功能：

- 节点（Node）控制器
- 路由（Route）控制器
- Service控制器
- 卷（Volume）控制器

### kube-scheduler

kube-scheduler 监视新创建没有分配到[Node](http://docs.kubernetes.org.cn/304.html)的[Pod](http://docs.kubernetes.org.cn/312.html)，为Pod选择一个Node。

### 插件 addons

插件（addon）是实现集群pod和Services功能的 。Pod由[Deployments](http://docs.kubernetes.org.cn/317.html)，ReplicationController等进行管理。Namespace 插件对象是在kube-system Namespace中创建。

#### DNS

虽然不严格要求使用插件，但Kubernetes集群都应该具有集群 DNS。

群集 DNS是一个DNS服务器，能够为 Kubernetes services提供 DNS记录。

由Kubernetes启动的容器自动将这个DNS服务器包含在他们的DNS searches中。

了解[更多详情](https://www.kubernetes.org.cn/542.html)

#### 用户界面

kube-ui提供集群状态基础信息查看。更多详细信息，请参阅[使用HTTP代理访问Kubernetes API](https://kubernetes.io/docs/tasks/access-kubernetes-api/http-proxy-access-api/)

#### 容器资源监测

[容器资源监控](https://kubernetes.io/docs/user-guide/monitoring)提供一个UI浏览监控数据，类似dashboard。

#### Cluster-level Logging

[Cluster-level logging](https://kubernetes.io/docs/user-guide/logging/overview)，负责保存容器日志，搜索/查看日志。

## Node节点组件



### kubelet

[kubelet](https://kubernetes.io/docs/admin/kubelet)是主要的节点代理，它会监视已分配给节点的pod，具体功能：

- 安装Pod所需的volume。
- 下载Pod的Secrets。
- Pod中运行的 docker（或experimentally，rkt）容器。
- 定期执行容器健康检查。
- 报告pod状态
- 报告node状态

### kube-proxy

[kube-proxy](https://kubernetes.io/docs/admin/kube-proxy)通过在主机上维护网络规则并执行连接转发来实现Kubernetes服务抽象

### docker

docker用于运行容器。

### RKT

rkt运行容器，作为docker工具的替代方案。

### supervisord

supervisord是一个轻量级的监控系统，用于保障kubelet和docker运行。

### fluentd

fluentd是一个守护进程，可提供[cluster-level logging](https://kubernetes.io/docs/concepts/overview/components/#cluster-level-logging).。

## Kubernetes工作流程

结合K8s的整体架构来看，用户通过与Master节点交互，提交任务；Master节点接收到用户任务后进行编排，并将任务分发到Node集群中；任务在Node节点上封装成Pod运行，同时Node节点会监测自身状态信息反馈给Master节点，例如Pod Running信息等；Master节点根据采集到的集群信息维护整个任务的运行状态，例如当某个Pod运行异常时，再启动一个新的Pod从而保证集群整体Pod数量稳定。

注：Pod的定义与表现形式

> Pod是Kubernetes创建或部署的最小/最简单的基本单位，一个Pod代表集群上正在运行的一个进程。
>
> 一个Pod封装一个应用容器（也可以有多个容器），存储资源、一个独立的网络IP以及管理控制容器运行方式的策略选项。Pod代表部署的一个单位：Kubernetes中单个应用的实例，它可能由单个容器或多个容器共享组成的资源。
>
> Docker是Kubernetes Pod中最常见的runtime ，Pods也支持其他容器runtimes。
>
> Kubernetes中的Pod使用可分两种主要方式：
>
> - Pod中运行一个容器。“one-container-per-Pod”模式是Kubernetes最常见的用法; 在这种情况下，你可以将Pod视为单个封装的容器，但是Kubernetes是直接管理Pod而不是容器。
> - Pods中运行多个需要一起工作的容器。Pod可以封装紧密耦合的应用，它们需要由多个容器组成，它们之间能够共享资源，这些容器可以形成一个单一的内部service单位 - 一个容器共享文件，另一个“sidecar”容器来更新这些文件。Pod将这些容器的存储资源作为一个实体来管理。

## Kubernetes更多概念

参见[Kubernetes文档](http://docs.kubernetes.org.cn/232.html)

