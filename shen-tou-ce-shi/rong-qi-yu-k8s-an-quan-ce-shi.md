# 容器与K8S安全测试

## 1.Docker安全测试命令

## 2.K8S安全测试命令

### 2.1 API Server安全测试

Kubernetes v1.10版本前API Server监听端口的默认配置为8080，即对此端口的任何请求都将绕过身份验证和授权检查，若将此端口保持开启状态，则访问Kubernetes主机的任何人都可完全控制整个集群。对于不安全的端口开放，Kubernetes提供了安全端口6443，

```text
curl -i https://<ip_address>:8080 -k
```

IP address为部署Kubernetes的Master节点IP，如果输出为一系列的API，那么不安全端口为打开状态；如果链接被拒绝，则证明安全端口为打开状态。

```text
curl -i https://<ip_address>:8443 -k
```

### 2.2  **Kubelet**安全测试

