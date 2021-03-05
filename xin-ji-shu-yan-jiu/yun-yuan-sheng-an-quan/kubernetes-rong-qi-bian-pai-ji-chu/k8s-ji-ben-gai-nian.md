# k8s基本概念

本文来源：[https://yeasy.gitbook.io/docker\_practice/kubernetes/concepts](https://yeasy.gitbook.io/docker_practice/kubernetes/concepts)

如有侵权，请联系我来删除！

## 主要组件



![k8s&#x4E3B;&#x8981;&#x7EC4;&#x4EF6;&#x5BFC;&#x56FE;](../../../.gitbook/assets/image%20%28131%29.png)

* 节点（`Node`）：一个节点是一个运行 Kubernetes 中的主机。
* 容器组（`Pod`）：一个 Pod 对应于由若干容器组成的一个容器组，同个组内的容器共享一个存储卷\(volume\)。
* 容器组生命周期（`pod-status`）：包含所有容器状态集合，包括容器组状态类型，容器组生命周期，事件，重启策略，以及 replication controllers。
* Replication Controllers：主要负责指定数量的 pod 在同一时间一起运行。
* 服务（`services`）：一个 Kubernetes 服务是容器组逻辑的高级抽象，同时也对外提供访问容器组的策略。
* 卷（`volumes`）：一个卷就是一个目录，容器对其有访问权限。
* 标签（`labels`）：标签是用来连接一组对象的，比如容器组。标签可以被用来组织和选择子对象。
* 接口权限（`accessing_the_api`）：端口，IP 地址和代理的防火墙规则。
* web 界面（`ux`）：用户可以通过 web 界面操作 Kubernetes。
* 命令行操作（`cli`）：`kubectl`命令。

## 运行原理 <a id="yun-hang-yuan-li"></a>

下面这张图完整展示了 Kubernetes 的运行原理。

![Kubernetes &#x67B6;&#x6784;](https://gblobscdn.gitbook.com/assets%2F-M5xTVjmK7ax94c8ZQcm%2F-M5xT_hHX2g5ldlyp9nm%2F-M5xTwBMQKj6UjWGe039%2Fk8s_architecture.png?alt=media)

可见，Kubernetes 首先是一套分布式系统，由多个节点组成，节点分为两类：一类是属于管理平面的主节点/控制节点（Master Node）；一类是属于运行平面的工作节点（Worker Node）。

显然，复杂的工作肯定都交给控制节点去做了，工作节点负责提供稳定的操作接口和能力抽象即可。

从这张图上，我们没有能发现 Kubernetes 中对于控制平面的分布式实现，但是由于数据后端自身就是一套分布式的数据库 Etcd，因此可以很容易扩展到分布式实现。

## 控制平面 <a id="kong-zhi-ping-mian"></a>

### 主节点服务 <a id="zhu-jie-dian-fu-wu"></a>

主节点上需要提供如下的管理服务：

* `apiserver` 是整个系统的对外接口，提供一套 RESTful 的 [Kubernetes API](https://kubernetes.io/zh/docs/concepts/overview/kubernetes-api/)，供客户端和其它组件调用；
* `scheduler` 负责对资源进行调度，分配某个 pod 到某个节点上。是 pluggable 的，意味着很容易选择其它实现方式；
* `controller-manager` 负责管理控制器，包括 endpoint-controller（刷新服务和 pod 的关联信息）和 replication-controller（维护某个 pod 的复制为配置的数值）。

### Etcd <a id="etcd"></a>

这里 Etcd 即作为数据后端，又作为消息中间件。

通过 Etcd 来存储所有的主节点上的状态信息，很容易实现主节点的分布式扩展。

组件可以自动的去侦测 Etcd 中的数值变化来获得通知，并且获得更新后的数据来执行相应的操作。

## 工作节点 <a id="gong-zuo-jie-dian"></a>

* kubelet 是工作节点执行操作的 agent，负责具体的容器生命周期管理，根据从数据库中获取的信息来管理容器，并上报 pod 运行状态等；
* kube-proxy 是一个简单的网络访问代理，同时也是一个 Load Balancer。它负责将访问到某个服务的请求具体分配给工作节点上的 Pod（同一类标签）。

![Proxy &#x4EE3;&#x7406;&#x5BF9;&#x670D;&#x52A1;&#x7684;&#x8BF7;&#x6C42;](https://gblobscdn.gitbook.com/assets%2F-M5xTVjmK7ax94c8ZQcm%2F-M5xT_hHX2g5ldlyp9nm%2F-M5xTwBOGp7VGvSMxrqj%2Fkube-proxy.png?alt=media)



