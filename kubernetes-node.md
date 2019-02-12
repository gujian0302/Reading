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
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: beta.kubernetes.io/arch
            operator: In
            values:
            - amd64
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: disk-type
            operator: In
            values:
            - ssd

```

## PodAffinity: Pod亲和与互斥调度策略

根据节点上面运行的Pod的标签而不是节点的标签进行判断和调度

```
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: security
            operator: in
            values:
            - s1
      topologyKey: kubernetes.io/hostname

spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: security
            operator: In
            values:
            - S1
        topologyKey: failure-domain.beta.kuberntes.io/zone
    podAntiAffinity:
      requiredDuringScheduingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - nginx
        topologyKey: kubernetes.io/hostname
```

*topologyKey*: which is the key for the node label that the system uses to denote such a topology domain.

# Taints && Tolerations

让Node拒绝Pod运行，在Node上设置一个或多个Taint之后，除非Pod明确声明=能够容忍这些污点，否则无法再这些Node上运行

```
kubectl taint node node1 key=value:NoSchedule

tolerations:
- key: "key"
  operator: "Equal"
  value: "value"
  effect: "NoSchedule"

or

tolerations:
- key: "key"
  operator: "Exists"
  effect: "NoSchedule"
```

# DaemonSet

在每一个Node上仅运行一份Pod的副本实例