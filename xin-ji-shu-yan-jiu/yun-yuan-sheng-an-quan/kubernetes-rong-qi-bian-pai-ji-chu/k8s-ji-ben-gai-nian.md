# k8s基本概念

本文来源：[https://yeasy.gitbook.io/docker\_practice/kubernetes/concepts](https://yeasy.gitbook.io/docker_practice/kubernetes/concepts)

如有侵权，请联系我来删除！



![k8s&#x4E3B;&#x8981;&#x7EC4;&#x4EF6;&#x5BFC;&#x56FE;](../../../.gitbook/assets/image%20%28131%29.png)

* 节点（`Node`）：一个节点是一个运行 Kubernetes 中的主机。
* 容器组（`Pod`）：一个 Pod 对应于由若干容器组成的一个容器组，同个组内的容器共享一个存储卷\(volume\)。
* 容器组生命周期（`pos-states`）：包含所有容器状态集合，包括容器组状态类型，容器组生命周期，事件，重启策略，以及 replication controllers。
* Replication Controllers：主要负责指定数量的 pod 在同一时间一起运行。
* 服务（`services`）：一个 Kubernetes 服务是容器组逻辑的高级抽象，同时也对外提供访问容器组的策略。
* 卷（`volumes`）：一个卷就是一个目录，容器对其有访问权限。
* 标签（`labels`）：标签是用来连接一组对象的，比如容器组。标签可以被用来组织和选择子对象。
* 接口权限（`accessing_the_api`）：端口，IP 地址和代理的防火墙规则。
* web 界面（`ux`）：用户可以通过 web 界面操作 Kubernetes。
* 命令行操作（`cli`）：`kubectl`命令。

