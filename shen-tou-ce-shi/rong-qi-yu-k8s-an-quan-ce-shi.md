# 容器与K8S安全测试

## 1.Docker安全测试命令

## 2.K8S安全测试命令

![](../.gitbook/assets/image%20%28128%29.png)

1. 用户与 kubectl 或者 Kubernetes Dashboard 进行交互，提交需求。（例: kubectl create -f pod.yaml）;

2. kubectl 会读取 ~/.kube/config 配置，并与 apiserver 进行交互，协议：http/https;

3. apiserver 会协同 ETCD 等组件准备下发新建容器的配置给到节点，协议：http/https（除 ETCD 外还有例如 kube-controller-manager, scheduler等组件用于规划容器资源和容器编排方向，此处简化省略）;

4. apiserver 与 kubelet 进行交互，告知其容器创建的需求，协议：http/https；

5. kubelet 与Docker等容器引擎进行交互，创建容器，协议：http/unix socket.

### 2.1 API Server安全测试

Kubernetes的API Server组件为控制Kubernetes的REST API，命令行工具Kubectl为API Server的客户端。Kubernetes v1.10版本前API Server监听端口的默认配置为8080，即对此端口的任何请求都将绕过身份验证和授权检查，若将此端口保持开启状态，则访问Kubernetes主机的任何人都可完全控制整个集群。对于不安全的端口开放，Kubernetes提供了安全端口6443，

```text
curl -i https://<ip_address>:8080 -k
```

IP address为部署Kubernetes的Master节点IP，如果输出为一系列的API，那么不安全端口为打开状态；如果链接被拒绝，则证明安全端口为打开状态。

```text
curl -i https://<ip_address>:8443 -k
```

### 2.2  **Kubelet**安全测试

Kubelet为Kubernetes每个节点上的代理，主要负责Pod的生命周期管理同时也会向Controller Manager组件汇报当前节点以及其上运行Pod的状态和指标，Kubelet组件自身还运行一个API，其它组件通过该API来执行Pod的启停操作。想象如果未经授权的用户可在任何节点上访问该API，便也同样可对集群中的任意Pod进行操纵从而控制整个集群。

每一个Node节点都有一个kubelet服务，kubelet监听了10250，10248，10255等端口。

其中10250端口是kubelet与apiserver进行通信的主要端口，通过该端口kubelet可以知道自己当前应该处理的任务，该端口在最新版Kubernetes是有鉴权的，但在开启了接受匿名请求的情况下，不带鉴权信息的请求也可以使用10250提供的能力；因为Kubernetes流行早期，很多挖矿木马基于该端口进行传播和利用，所以该组件在安全领域部分群体内部的知名度反而会高于 APIServer。

![](../.gitbook/assets/image%20%28129%29.png)

作者在此处提供了一个简单的检查方法，如下所示：

```text
curl -sk https://<ip_address>:10250/pods
```

IP address为部署Kubernetes的Master节点IP，如果输出为Unauthorized，则说明匿名认证为关闭状态，进而说明配置是相对安全的。





