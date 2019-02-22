## Kubernetes部署流程

#### 编译，打包镜像

1. Dockerfile
2. dockerfile-maven-plugin


#### 上传至私有镜像库

```
docker login
docker push registry.cn-shanghai.aliyuncs.com/{namespace}/{project-name}:{version}
```

#### 生成拉取镜像库密钥并保存至kubernetes Secrets

```
kubectl create secret docker-registry regcred --docker-server=<your-registry-server> --docker-username=<your-name> --docker-password=<your-password> --docker-email=<your-email>

```

##### 如何使用

> *imagePullSecrets*

```
apiVersion:v1
kind: Pod
metadata:
  name: private-reg
spec:
  containers:
  - name: private-reg-container
    image: <your-private-image>
  imagePullSecrets:
  - name: regcred

```


##### 备注: 
[https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/)

#### 编写无状态应用部署脚本(Deployment)


#### 编写服务脚本(Service)

> 作为服务的负载均衡


#### 编写路由Ingress文件

> 作为外部访问的入口，并且将接口转发到不同的服务上面


#### HTTPS证书打包进Kubernetes Secrets中, 并让Ingress使用密钥

```
kubectl create secret tls --help 
kubectl create secret tls tls-secret --cert=path/to/tls.pem --key=path/to/tls.key


apiVersion: extension/v1beta1
tls:
- secretName: tls-secret
  hosts:
  - {DOMAIN}
```


