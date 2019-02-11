# How to Deploy Spring Cloud To Kubernetes

| module | Spring Cloud | Kubernetes |
| -------| -------------| ---------- |
| 中央配置服务器| Spring Cloud *Config* | Kubernetes *ConfigMaps and Secrets*|
| API网关| Spring Cloud *Zuul* | Kubernetes *ingresses* |
| 服务发现| Spring Cloud *Eureka* | Kubernetes *etcd* |

---
###Spring Cloud Kubernetes

* spring cloud kubernetes 并没有发布在spring-cloud-dependencies bom 中


####Build Local Requirements
1. kubectl create clusterrolebinds
2. spring application name must be  the same as the Kubernetes servicee name
3. in order to solve interservice communication. we should additionally add *spring-cloud-starter-kubernetes-ribbon*, spring cloud kubernetes Ribbon provides some beans that force Ribbon to communicate with the kubernetes API through Fabric8 kubernetesClient
4. Ingress is a substitute of zuul. It plays the role of an API Gateway.
5. use zuul to create a swagger API gateway.


```
#deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongodb
  labels: 
    app: mongodb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongodb
  
  template:
    metadata:
      labels:
        app: mongodb
      spec:
        containers:
        - name: mongodb
          image: mongo:latest
          ports: 
          - containerPort: 27017
          env:
          - name: MONGO_INITDB_DATABASE
            valueFrom: 
              configFrom:
                configMapKeyRef:
                  name: mongodb
                  key: database-name
         - name: MONGO_INITDB_ROOT_USERNAME
           valueFrom:
             secretKeyRef：
               name: mongodb
               key: database-user
         - name: MONGO_INITDB_ROOT_PASSWORD
           valueFrom:
             secretKeyRef:
               name: mongodb
               key: database-password
                 
```

```
#ingress.yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: gateway-ingress
  annotations: nginx.ingress.kubernetes.io/rewrite-target: /
  
spec:
  backend: 
    serviceName: default-http-backend
    servicePort: 80
  rules:
  - host: microservices.info
    http:
      paths:
      - path: /employee
        backend: 
          serviceName: employee
          servicePort: 8080
      - path: /department
        backend: 
          serviceName: department
          servicePort: 8080
      - path: /organization
        backend:
          serviceName: organization
          servicePort: 8080
      
```

```java
@Configuration
public class GatewayApi {
 @Autowired
 ZuulProperties properties;
 @Primary
 @Bean
 public SwaggerResourcesProvider swaggerResourcesProvider() {
  return () -> {
   List resources = new ArrayList();
   properties.getRoutes().values().stream()
   .forEach(route -> resources.add(createResource(route.getId(), "2.0")));
   return resources;
  };
 }
 private SwaggerResource createResource(String location, String version) {
  SwaggerResource swaggerResource = new SwaggerResource();
  swaggerResource.setName(location);
  swaggerResource.setLocation("/" + location + "/v2/api-docs");
  swaggerResource.setSwaggerVersion(version);
  return swaggerResource;
 }
}
```