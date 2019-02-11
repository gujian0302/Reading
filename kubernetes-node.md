# Kubernetes NodeSelector 定向调度

1. kubectl label nodes <node-name> <label-key>=<label-value>
2. pod.yaml 添加node-selector

```
spec:
  template: 
    metadata: 
      labels:
        name: redis-master
    spec:
      containers:
      - name: master
        image: kubeguide/redis-master
        ports:
          - containerPort: 6379
      nodeSelector:
        zone: north
```

## NodeAffinity: Node亲和性调度

1. RequiredDuringSchedulingIgnoredDuringExecution: 
2. PreferredDuringSchedulingIgnoredExecution:

```

```