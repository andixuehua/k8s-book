https://kubernetes.io/zh-cn/docs/concepts/configuration/configmap/

# ConfigMap

ConfigMap 是一种 API 对象，用来将非机密性的数据保存到键值对中。使用时， [Pods](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/) 可以将其用作环境变量、命令行参数或者存储卷中的配置文件。

ConfigMap 将你的环境配置信息和 [容器镜像](https://kubernetes.io/zh-cn/docs/reference/glossary/?all=true#term-image) 解耦，便于应用配置的修改。

**注意：**

ConfigMap 并不提供保密或者加密功能。 如果你想存储的数据是机密的，请使用 [Secret](https://kubernetes.io/zh-cn/docs/concepts/configuration/secret/)， 或者使用其他第三方工具来保证你的数据的私密性，而不是用 ConfigMap。

## 动机

使用 ConfigMap 来将你的配置数据和应用程序代码分开。

比如，假设你正在开发一个应用，它可以在你自己的电脑上（用于开发）和在云上 （用于实际流量）运行。 你的代码里有一段是用于查看环境变量 `DATABASE_HOST`，在本地运行时， 你将这个变量设置为 `localhost`，在云上，你将其设置为引用 Kubernetes 集群中的 公开数据库组件的 [服务](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/)。

这让你可以获取在云中运行的容器镜像，并且如果有需要的话，在本地调试完全相同的代码。

ConfigMap 在设计上不是用来保存大量数据的。在 ConfigMap 中保存的数据不可超过 1 MiB。如果你需要保存超出此尺寸限制的数据，你可能希望考虑挂载存储卷 或者使用独立的数据库或者文件服务。



#### 被挂载的 ConfigMap 内容会被自动更新

当卷中使用的 ConfigMap 被更新时，所投射的键最终也会被更新。 kubelet 组件会在每次周期性同步时检查所挂载的 ConfigMap 是否为最新。 不过，kubelet 使用的是其本地的高速缓存来获得 ConfigMap 的当前值。 高速缓存的类型可以通过 [KubeletConfiguration 结构](https://kubernetes.io/zh-cn/docs/reference/config-api/kubelet-config.v1beta1/). 的 `ConfigMapAndSecretChangeDetectionStrategy` 字段来配置。

ConfigMap 既可以通过 watch 操作实现内容传播（默认形式），也可实现基于 TTL 的缓存，还可以直接经过所有请求重定向到 API 服务器。 因此，从 ConfigMap 被更新的那一刻算起，到新的主键被投射到 Pod 中去， 这一时间跨度可能与 kubelet 的同步周期加上高速缓存的传播延迟相等。 这里的传播延迟取决于所选的高速缓存类型 （分别对应 watch 操作的传播延迟、高速缓存的 TTL 时长或者 0）。

以环境变量方式使用的 ConfigMap 数据不会被自动更新。 更新这些数据需要重新启动 Pod。

**说明：** 使用 ConfigMap 作为 [subPath](https://kubernetes.io/zh-cn/docs/concepts/storage/volumes#using-subpath) 卷挂载的容器将不会收到 ConfigMap 的更新。





```

kubectl create configmap log-collection-app-config  --from-file config.ini

kubectl delete deployments.apps log-collection

kubectl apply -f deployment.yaml

kubectloc apply -f deployment-bad.yaml
```



- config.ini

```
name=log-colleciton
db=127.0.0.1:3303/db
password=123456
```

- deployment.yaml

```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: log-collection
  labels:
    app: log-collection
spec:
  replicas: 1
  selector:
    matchLabels:
      deployment: log-collection
  template:
    metadata:
      labels:
        deployment: log-collection
    spec:
      volumes:
        - name: cm-volume
          configMap:
            name: log-collection-app-config

      containers:
        - name: log-collection
          image: >-
            quay.io/qxu/log-collection:v3
          ports:
            - containerPort: 8080
              protocol: TCP
          resources: {}
          volumeMounts:
            - name: cm-volume
              mountPath: /data/config.ini
              subPath: config.ini
              #readOnly: true
          imagePullPolicy: IfNotPresent
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      dnsPolicy: ClusterFirst
      securityContext: {}
```

- deployment-bad.yaml

```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: log-collection-bad
spec:
  replicas: 1
  selector:
    matchLabels:
      deployment: log-collection-bad
  template:
    metadata:
      labels:
        deployment: log-collection-bad
    spec:
      volumes:
        - name: cm-volume
          configMap:
            name: log-collection-app-config

      containers:
        - name: log-collection-bad
          image: >-
            quay.io/qxu/log-collection:v3
          ports:
            - containerPort: 8080
              protocol: TCP
          resources: {}
          volumeMounts:
            - name: cm-volume
              mountPath: /data/config.ini
              readOnly: true
          imagePullPolicy: IfNotPresent
 
```

挂载文件到指定文件名时，要使用subpath,



示例2：redis的配置，参考：https://kubernetes.io/zh/docs/tutorials/configuration/configure-redis-using-configmap/

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: example-redis-config
data:
  redis-config: |
    maxmemory 2mb
    maxmemory-policy allkeys-lru    

```



```
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
  - name: redis
    image: redis:5.0.4
    command:
      - redis-server
      - "/redis-master/redis.conf"
    env:
    - name: MASTER
      value: "true"
    ports:
    - containerPort: 6379
    resources:
      limits:
        cpu: "0.1"
    volumeMounts:
    - mountPath: /redis-master-data
      name: data
    - mountPath: /redis-master
      name: config
  volumes:
    - name: data
      emptyDir: {}
    - name: config
      configMap:
        name: example-redis-config
        items:
        - key: redis-config
          path: redis.conf

```

测试

```
# kubectl exec -it redis -n test /bin/sh
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
# pwd
/data
# cat /redis-master/redis.conf
maxmemory 2mb
maxmemory-policy allkeys-lru

```



示例3，将第一种方法，改造成的key的方式mount configmap，这里的的key可从kubectl get cm -o yaml中查看 

```
kind: Deployment
apiVersion: apps/v1
metadata:
  name: log-collection2
  labels:
    app: log-collection2
spec:
  replicas: 1
  selector:
    matchLabels:
      deployment: log-collection2
  template:
    metadata:
      labels:
        deployment: log-collection2
    spec:
      volumes:
        - name: cm-volume
          configMap:
            name: log-collection-app-config

      containers:
        - name: log-collection
          image: >-
            quay.io/qxu/log-collection:v3
          ports:
            - containerPort: 8080
              protocol: TCP
          resources: {}
          volumeMounts:
            - name: cm-volume
              mountPath: /data
              #subPath: config.ini
              #readOnly: true
          imagePullPolicy: IfNotPresent
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      dnsPolicy: ClusterFirst
      securityContext: {}

```



测试subpath 挂载和mountPath,在修改configmap后,数据能否在pod内显示更新后的内容



```
vi 
kubectl create configmap log-collection-app-config  --from-file config.ini -n test --dry-run -o yaml|kubectl apply -f - -n test

kubectl get cm log-collection-app-config -o yaml -n test

kubectl exec -it log-collection-fb9b5967c-slb49 -n test cat /data/config.ini
```



***可以看到,使用 mountPath: /data的pod可以直接获得configmap的更新,　使用subPath是不会更新***
