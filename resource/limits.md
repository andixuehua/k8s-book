https://kubernetes.io/zh-cn/docs/concepts/policy/limit-range/

# 限制范围

默认情况下， Kubernetes 集群上的容器运行使用的[计算资源](https://kubernetes.io/zh-cn/docs/concepts/configuration/manage-resources-containers/)没有限制。 使用资源配额，集群管理员可以以[名字空间](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/namespaces/)为单位，限制其资源的使用与创建。 在命名空间中，一个 Pod 或 Container 最多能够使用命名空间的资源配额所定义的 CPU 和内存用量。 有人担心，一个 Pod 或 Container 会垄断所有可用的资源。 LimitRange 是在命名空间内限制资源分配（给多个 Pod 或 Container）的策略对象。

一个 **LimitRange（限制范围）** 对象提供的限制能够做到：

- 在一个命名空间中实施对每个 Pod 或 Container 最小和最大的资源使用量的限制。

- 在一个命名空间中实施对每个 PersistentVolumeClaim 能申请的最小和最大的存储空间大小的限制。

- 在一个命名空间中实施对一种资源的申请值和限制值的比值的控制。

- 设置一个命名空间中对计算资源的默认申请/限制值，并且自动的在运行时注入到多个 Container 中。

  

- #### 设置项目的资源限定

  如果容器没有声明自己的 CPU 请求和限制，将为容器指定默认 CPU 请求和限制。

```shell
oc new-project test-node

oc apply -f deploy-test.yaml

＃先检查pod中是否有设置resource的限定


```

- deploy-test.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: log-collection
spec:
  selector:
    matchLabels:
      app: log-collection
  replicas: 
  template:
    metadata:
      labels:
        app:  log-collection
    spec:
      containers:
        - name:  log-collection
          image: quay.io/qxu/log-collection
          ports:
            - containerPort: 8080
```

- 实现限定

```shell

kubectl apply -f limit-range.yaml   -n test1

#删除原有部署，重新部署后，再检查pod中的资源限定定义
kubectl -f deploy-test.yaml
kubectl apply -f deploy-test.yaml


发现pod中会自动加入：
  containers:
    - resources:
        limits:
          cpu: 500m
          memory: 1536Mi
        requests:
          cpu: 150m
          memory: 256Mi
```

- limit-range.yaml  

```yaml
kind: LimitRange
apiVersion: v1
metadata:
  name: test-debug-core-resource-limits
spec:
  limits:
    - type: Container
      max:
        cpu: '2'
        memory: 2Gi
      default:
        cpu: 500m
        memory: 1536Mi
      defaultRequest:
        cpu: 150m
        memory: 256Mi
    - type: Pod
      max:
        cpu: '2'
        memory: 2Gi
        
        
 kubectl get limitrange -n test1
        
```



参考： https://kubernetes.io/zh/docs/concepts/policy/limit-range/



新建一个超过max memory 的应用，这时不会成功建立 pod, 

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: log-collection
spec:
  selector:
    matchLabels:
      app: log-collection
  replicas: 1
  template:
    metadata:
      labels:
        app: log-collection
    spec:
      containers:
        - name: log-collection
          image: quay.io/qxu/log-collection
          ports:
            - containerPort: 8080
          resources:
            requests:
              cpu: 150m
              memory: 2.5Gi  
```



可使用以下命令：修改为1.5 Gi可以成功

```
 kubectl get event -n test
46s         Warning   FailedCreate        replicaset/log-collection-6c454dc7f8   Error creating: Pod "log-collection-6c454dc7f8-nmqbd" is invalid: spec.containers[0].resources.requests: Invalid value: "2560Mi": 
must be less than or equal to memory limit

kubectl get deployment log-collection -n test
    message: 'Pod "log-collection-6c454dc7f8-g5ww2" is invalid: spec.containers[0].resources.requests:
      Invalid value: "2560Mi": must be less than or equal to memory limit'

```

