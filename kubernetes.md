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

### Deployment(全自动调度)

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


### DaemonSet

在每一个Node上仅运行一份Pod的副本实例

### StatefulSet

面向有状态的服务，mysql, mongodb,zookeeper

### Service

通过Label Selector来定义一个服务的访问入口地址，提供一个全局唯一的虚拟ip(clusterIp)，通过负载均衡器，发送到后端的某个Pod上面
通过创建Service,可以为一组具有相同功能的容器提供一个统一个入口地址，并且将请求负载均衡分发到后端的各个容器应用上

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

将不同的URL地址访问请求转发到护短不同的Service,以实现Http层业务路由机制

```
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: nginx-ingress-lb
  labels:
    name: nginx-ingress-lb
  namespace: kube-system
spec:
  template:
    metadata:
      labels:
        name: nginx-ingress-lb
    spec:
      terminationGracePeriodSeconds: 60
      containers:
        - image: gcr.io/google_containers/nginx-ingress-controller: 0.9.0-beta.2
          name: nginx-ingress-lb
          readinessProbe:
            httpGet:
              path: /healthz
              port: 10254
              scheme: HTTP
      ports:
        - containerPort: 80
          hostPort:80
        - containerPort: 443
          hostPort: 443

      args:
      - /nginx-ingress-controller
      - --default-backend-service=${POD_NAMESPACE}/default-http-backend


apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: default-http-backend
  labels:
    k8s-app: default-http-backend
  namespace: kube-system
spec:
  replicas: 1
  template:
    metadata:
      labels:
        k8s-app: default-http-backend
    spec:
      image: gcr.io/googleCotnainers/defaultbackend:1.0

apiVersion: v1
kind: Service
metadata: 
  name: default-http-backend
  namespace: kube-system
  labels:
    k8s-app: default-http-backend
spec:
  ports:
  - port: 80
    targetPort: 8080
  selector:
    k8s-app: default-http-backend

apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: mywebsite-ingress
spec:
  rules:
  - host: mywebsite.com
    http:
      paths: 
      - path: /demo
        backend:
          serviceName: webapp
          servicePort: 8080

```

### apiVersion

| Kind | apiVersion|
| ---- | --------- |
| CertificateSigningRequest| certificate.k8s.io/v1beta1|
| ClusterRoleBinding | rbac.authorization.k8s.io/v1 |
| ClusterRole | rbac.authorization.k8s.io/v1 |
| ComponentStatus | v1|
| ConfigMap | v1 |
| ControllerRevision | apps/v1| 
| CronJob | batch/v1beta1 |
| DaemonSet | extensions/v1beta1|
| Deployment | extensions/v1beta1|
| Endpoints | v1|
| Event | v1 |
| HorizontalPodAutoscaler | autoscaling/v1|
| Ingress | extension/v1beta1 |
| Job | batch/v1 |
| LimitRange | v1|
| Namespace | v1 |
| NetworkPolicy | extensions/v1beta1 |
| Node | v1 |
| PersistenceVolumeClaim | v1|
| PersistenceVolume | v1 |
| PodDisruptionBudget| policy/v1beta1|
| Pod                | v1 |
| PodSecurityPolicy  | extensions/v1beta1|
| PodTemplate        | v1 |
| ReplicaSet |        extensions/v1beta1|
| ReplicationController | v1 |
| ResourceQuota| v1|
| RoleBinding | rbac.authorization.k8s.io/v1|
| Role | rbac.authorization.k8s.io/v1|
| Secret | v1|
| ServiceAccount | v1|
| Service | v1|
| StatefulSet | apps/v1| 

### Job 批处理调度

1. Job Template Expansion模式: 一个Job对象对应一个待处理的WorkItem
2. Queue With Pod Per Work Item: Job会启动N个Pod,每个Pod对应一个WorkItem
3. Queue With Variable Pod Count 模式： 也是采用一个任务队列存放WorkItem,Job启动的Pod的数量是可变的

### CronJob 定时任务

--runtime-config=batch/v2alpha1=true
Minutes Hours DayOfMonth Month DayOfWeek Year

```
#cron.yaml
apiVersion: batch/v2alpha1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec: containers:
        - name: hello
          image: busybox
          args:
          - /bin/sh
          - -c
          - date;/ echo Hello from the Kubernetes cluster
        restartPolicy: OnFailure

kubectl create -f cron.yaml
```

### Deployment 升级和回滚

RollingUpdate策略

1. Recreate 重建 先全部杀掉正在运行的Pod spec.strategy.type=Recreate
2. RollingUpdate 滚动更新 spec.strategy.type=RollingUpdate
     spec.strategy.rollingUpdate.maxUnavaiable
     spec.strategy.rollingUpdate.maxSurge

```
##设置新的镜像
kubectl set image deploy/nginx-deployment nginx=nginx:1.9.1

##查看升级状态
kubectl rollout status deploy/nginx-deployment

##回滚
kubectl rollout history deployment/nginx-deployment
kubectl rollout history deployment/nginx-deployment --revision=3
kubectl rollout undo deployment/nginx-deployment
kubectl rollout undo deployment/nginx-deployment --to-revision=2


##暂停更新,以更改资源设置等
kubectl rollout pause deployment/nginx-deployment
kubectl rollout resume deployment/nginx-deployment

```

##### compare with rc

rc滚动升级不含有deployment应用版本升级过程中的历史记录，新旧版本数量的精细控制等功能


### HPA(Horizontal Pod Autoscaler)

```
#hpa-php-apache.yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: php-apache
spec:
  scaleTargetRef:
    apiVersion: apps/v1/beta1
    kind: Deployment
    name: php-apache
  minReplicas: 1
  maxReplicas: 10
  targetCPUUtilizationPercentage: 50
```

### HeadLess Service
Sometimes you don't need or want load-balancing and a single service IP. In this case, you can create "headless" service by specifying "None" for the clusterIp 

### StatefulSet(有状态的集群)
a StatefulSet maintains a sticky indentity for each of their pods. These pods are created from the same spec, but are not interchangeable.each has a persistent identifier that it maintains across any rescheduling.

### 在集群外部访问Pod或Service

1. pod:容器的端口映射到物理机上
2. pod:hostNetwork=true，将该Pod中所有的容器的端口号都将直接映射到物理机上
3. 将Server端口映射到物理机

```
apiVersion: v1
kind: Service
metadata:
  name: webapp
spec: 
  type: NodePort
  ports:
  - port: 8080
    targetPort: 8080
    nodePort: 8081
  selector:
    app: webapp
```

1. 设置Service类型为NodePort
2. 设置LoadBalancer映射到云服务器LoadBalancer


### DNS 服务

1. etcd: DNS 存储
2. kube2sky: 将Kubernetes Master中的Service注册到etcd
3. skuDNS: 提供DNS域名解析服务
4. healthz: 提供对skydns服务的健康检查功能

### 阿里云Kubernetes免密
[https://help.aliyun.com/document_detail/103178.html?spm=5176.10695662.1996646101.searchclickresult.31544983xtHFm5](https://help.aliyun.com/document_detail/103178.html?spm=5176.10695662.1996646101.searchclickresult.31544983xtHFm5)

### Kubernetes Rabbitmq Cluster
[https://www.kubernetes.org.cn/4679.html](https://www.kubernetes.org.cn/4679.html)
