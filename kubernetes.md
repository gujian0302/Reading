# Kubernetes 基本概念

### Master 

管理控制kubernetes集群的节点

1. Kubernetes API Server (kube-apiserver): 提供Http Rest接口的关键服务进程,是Kubernetes 里所有的资源的增，删，改，查等操作的唯一入口
2. Kubernetes Controller Manager: 资源对象的自动化控制中心
3. Kubernetes Scheduler 负责资源调度（Pod调度) 的集成

### Node

除了Master节点以外的节点，Kubernetes集群中的其他机器都可以成为Node节点，Node节点是Kubernetes集群中工作负载的节点，每个Node都会被分配一些工作负载


1. kubelet 负责Pod对应的容器的创建、启停的任务，同时与master节点密切协作
2. kube-proxy 实现kubernetes Server通信负载均衡
3. Docker Engine: Docker引擎

### Deployment

更好的处理pod的编排问题,作为RC(Replication Controller)的升级，支持滚动升级，回滚，暂停修改

### Pod

最基本的单元，包含一个活多个紧密相关的用户业务容器，每个Pod都有一个特殊的被称为根容器的Pause容器

### Replication Controller
定义一个期望的场景，期望某个Pod在任意时刻副本的数量都到达期望值（逐渐被Deployment替换）

```
spec: 
  replicas: 2
  selector:
    tier: frontend
```



### Horizontal Pod Autoscaler

自动拓容

```
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: php-apache
  namespace: default
spec:
  maxReplicas: 10
  minReplicas: 1
  scaleTargetRef:
    kind: Deployment
    name: php-apache
  targetCPUUtilizationPercentage: 90
```

### StatefulSet

面向有状态的服务，mysql, mongodb,zookeeper

### Service

通过Label Selector来定义一个服务的访问入口地址，提供一个全局唯一的虚拟ip(clusterIp)，通过负载均衡器，发送到后端的某个Pod上面

```
apiVersion: v1
kind: Service
metadata:
  name: tomcat-service
spec:
  ports:
  - port: 8080
  selector:
    tier: frontend
```


### Ingress
