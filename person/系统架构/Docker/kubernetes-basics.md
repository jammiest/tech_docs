# Kubernetes 基础完全指南

## 核心概念架构

```
+----------------+     +----------------+     +----------------+
|   控制平面     | --> |   工作节点     | --> |   网络存储     |
| - API Server  |     - kubelet      |     - CNI插件     |
| - etcd        |     - kube-proxy   |     - CSI驱动     |
| - Scheduler   |     - 容器运行时   |     - 服务发现    |
| - Controller  |     - Pod          |     - 负载均衡    |
+----------------+     +----------------+     +----------------+
```

## 核心组件详解

### 控制平面组件
```bash
# 1. kube-apiserver: 集群API入口
# 功能：认证、授权、校验、持久化

# 2. etcd: 分布式键值存储
# 功能：存储集群所有状态数据

# 3. kube-scheduler: 调度器
# 功能：将Pod分配到合适节点

# 4. kube-controller-manager: 控制器管理器
# 包含：Node控制器、Replication控制器、Endpoint控制器等

# 5. cloud-controller-manager: 云控制器管理器
# 功能：与云提供商API交互
```

### 工作节点组件
```bash
# 1. kubelet: 节点代理
# 功能：管理Pod生命周期，与容器运行时交互

# 2. kube-proxy: 网络代理
# 功能：维护节点网络规则，实现服务负载均衡

# 3. 容器运行时: Docker/containerd/CRI-O
# 功能：运行容器
```

## 基本对象与资源

### Pod - 最小部署单元
```yaml
# pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    ports:
    - containerPort: 80
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

### Deployment - 部署管理
```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
```

### Service - 服务发现
```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP  # 还有NodePort、LoadBalancer
```

## 基本操作命令

### 集群管理命令
```bash
# 查看集群信息
kubectl cluster-info

# 查看节点状态
kubectl get nodes
kubectl describe node <node-name>

# 查看集群组件状态
kubectl get componentstatuses

# 查看资源使用
kubectl top nodes
kubectl top pods
```

### 资源操作命令
```bash
# 创建资源
kubectl apply -f deployment.yaml
kubectl create deployment nginx --image=nginx:1.21

# 查看资源
kubectl get pods
kubectl get deployments
kubectl get services
kubectl get all

# 详细查看
kubectl describe pod <pod-name>
kubectl describe deployment <deployment-name>

# 删除资源
kubectl delete -f deployment.yaml
kubectl delete pod <pod-name>
```

### 调试与排查命令
```bash
# 查看日志
kubectl logs <pod-name>
kubectl logs -f <pod-name>  # 实时日志
kubectl logs <pod-name> -c <container-name>  # 多容器Pod

# 进入容器
kubectl exec -it <pod-name> -- /bin/bash
kubectl exec <pod-name> -- ls /app

# 端口转发
kubectl port-forward <pod-name> 8080:80

# 调试命令
kubectl debug -it <pod-name> --image=busybox --target=<pod-name>
```

## 配置与存储

### ConfigMap - 配置管理
```yaml
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database.url: "postgresql://localhost:5432"
  log.level: "info"
  app.settings: |
    timeout=30
    retries=3
```

### Secret - 密钥管理
```yaml
# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  username: dXNlcm5hbWU=  # base64编码
  password: cGFzc3dvcmQ=
```

### Volume - 存储卷
```yaml
# pod-with-volume.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-volume
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: app-config
```

## 网络与服务

### Service 类型详解
```yaml
# 1. ClusterIP (默认)
apiVersion: v1
kind: Service
metadata:
  name: clusterip-service
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
  - port: 80

# 2. NodePort
apiVersion: v1
kind: Service
metadata:
  name: nodeport-service
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30007

# 3. LoadBalancer
apiVersion: v1
kind: Service
metadata:
  name: loadbalancer-service
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - port: 80
```

### Ingress - 入口控制器
```yaml
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-service
            port:
              number: 80
```

## 资源管理与调度

### Resource 资源限制
```yaml
# resources.yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-demo
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

### Namespace - 命名空间
```bash
# 创建命名空间
kubectl create namespace development

# 在特定命名空间操作
kubectl get pods -n development
kubectl apply -f deployment.yaml -n development

# 查看所有命名空间资源
kubectl get all --all-namespaces
```

## 健康检查与自愈

### Liveness Probe - 存活检查
```yaml
# liveness.yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-demo
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    livenessProbe:
      httpGet:
        path: /health
        port: 80
      initialDelaySeconds: 30
      periodSeconds: 10
```

### Readiness Probe - 就绪检查
```yaml
# readiness.yaml
apiVersion: v1
kind: Pod
metadata:
  name: readiness-demo
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    readinessProbe:
      httpGet:
        path: /ready
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
```

## 部署策略

### Rolling Update - 滚动更新
```yaml
# rolling-update.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.22  # 更新镜像版本
        ports:
        - containerPort: 80
```

### Blue-Green Deployment
```bash
# 蓝绿部署策略
# 1. 部署新版本（绿色环境）
kubectl apply -f deployment-green.yaml

# 2. 测试新版本
kubectl port-forward green-pod 8080:80

# 3. 切换流量
kubectl patch service my-service -p '{"spec":{"selector":{"version":"green"}}}'

# 4. 清理旧版本
kubectl delete deployment blue-deployment
```

## 监控与日志

### 基础监控
```bash
# 查看资源使用
kubectl top nodes
kubectl top pods

# 查看事件
kubectl get events --sort-by=.metadata.creationTimestamp

# 查看资源详情
kubectl describe pod <pod-name>
```

### 日志收集
```bash
# 多容器日志查看
kubectl logs <pod-name> -c <container-name>

# 多个Pod日志查看
kubectl logs -l app=nginx

# 日志文件导出
kubectl logs <pod-name> > pod.log
```

## 安全基础

### ServiceAccount - 服务账户
```yaml
# serviceaccount.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-serviceaccount
```

### RBAC - 基于角色的访问控制
```yaml
# role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]

# rolebinding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
subjects:
- kind: ServiceAccount
  name: my-serviceaccount
roleRef:
  kind: Role
  name: pod-reader
```

## 故障排查指南

### 常见问题排查
```bash
# Pod启动失败
kubectl describe pod <pod-name>
kubectl logs <pod-name>

# 服务无法访问
kubectl get endpoints <service-name>
kubectl describe service <service-name>

# 节点问题
kubectl describe node <node-name>
kubectl get events --field-selector involvedObject.kind=Node

# 资源不足
kubectl describe pod <pod-name> | grep -A 10 Events
kubectl get nodes -o wide
```

### 调试工具
```bash
# 使用busybox调试网络
kubectl run debug --image=busybox --rm -it --restart=Never -- nslookup my-service

# 检查DNS解析
kubectl run dns-test --image=busybox --rm -it --restart=Never -- nslookup kubernetes.default

# 检查网络连通性
kubectl run network-test --image=busybox --rm -it --restart=Never -- ping <target-ip>
```

## 学习资源与下一步

### 推荐学习路径
1. **基础概念**: Pod、Deployment、Service
2. **核心操作**: kubectl命令、YAML配置
3. **网络存储**: Volume、ConfigMap、Secret
4. **高级特性**: Ingress、RBAC、HPA
5. **生产实践**: 监控、日志、安全

### 实践环境搭建
```bash
# 使用Minikube（本地开发）
minikube start --driver=docker
minikube dashboard

# 使用Kind（本地集群）
kind create cluster
kubectl cluster-info --context kind-kind

# 使用k3d（轻量级K3s）
k3d cluster create my-cluster
```

> 提示：Kubernetes学习曲线较陡，建议从基础概念开始，逐步实践。

!> 重要：生产环境部署前，务必充分测试并了解相关安全最佳实践。