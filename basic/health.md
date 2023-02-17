# 配置存活、就绪和启动探针

https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/

如何给容器配置活跃（Liveness）、就绪（Readiness）和启动（Startup）探针

[kubelet](https://kubernetes.io/zh-cn/docs/reference/command-line-tools-reference/kubelet/) 使用存活探针来确定什么时候要重启容器。 例如，存活探针可以探测到应用死锁（应用程序在运行，但是无法继续执行后面的步骤）情况。 重启这种状态下的容器有助于提高应用的可用性，即使其中存在缺陷。

kubelet 使用就绪探针可以知道容器何时准备好接受请求流量，当一个 Pod 内的所有容器都就绪时，才能认为该 Pod 就绪。 这种信号的一个用途就是控制哪个 Pod 作为 Service 的后端。 若 Pod 尚未就绪，会被从 Service 的负载均衡器中剔除。

kubelet 使用启动探针来了解应用容器何时启动。 如果配置了这类探针，你就可以控制容器在启动成功后再进行存活性和就绪态检查， 确保这些存活、就绪探针不会影响应用的启动。 启动探针可以用于对慢启动容器进行存活性检测，避免它们在启动运行之前就被杀掉。





分别在三个同的ns下创建以下三个部署，服务，以及ingress进行测试

不带健康检查的配置

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: log-collection
spec:
  replicas: 1
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
          image: quay.io/qxu/log-collection:health-demo
          ports:
            - containerPort: 8080
```



带健康检查的配置,可将livenessProbe的port修改为9090,，

#### liveness

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
          image: quay.io/qxu/log-collection:health-demo
          ports:
            - containerPort: 8080
          livenessProbe:
            httpGet:
              port: 9090
              path: /
            initialDelaySeconds: 3
            periodSeconds: 10

```



查看 pod会一起重启，可使用kubectl describe pod查看 重启原因 

```
watch -n 5 kubectl get pod -n test
NAME                             READY   STATUS    RESTARTS   AGE
counter-745d94ddfc-hdtx6         2/2     Running   2          4h33m
log-collection-bbc67b77d-nvzjg   1/1     Running   2          2m48s
nginx                            1/1     Running   1          6h47m


kubectl describe pod -n test log-collection-bbc67b77d-nvzjg
  Warning  Unhealthy  35s (x10 over 3m35s)  kubelet            Liveness probe failed: Get "http://10.244.1.56:8090/": dial tcp 10.244.1.56:8090: connect: connecti
on refused

```

```
上面可以看到这个测试pod被重启了5次，然而服务始终正常不了，就会保持在CrashLoopBackOff了，等待运维人员来进行下一步错误排查
注：kubelet会以指数级的退避延迟（10s，20s，40s等）重新启动它们，上限为5分钟
```



带生命周期的配置，建立svc后查看正常的 endpoints中的pod，然后修改port中的8090 ，再查看 endpoints中的pod,这时svc会不正常。

get pod,也会发现不是ready的状态。

```

# kubectl get endpoints -n test1
NAME             ENDPOINTS   AGE
log-collection               15d

# kubectl get pod -o wide -n test1
NAME                             READY   STATUS    RESTARTS   AGE   IP             NODE                 NOMINATED NODE   READINESS GATES
log-collection-555cc9cb5-glc57   0/1     Running   0          81s   10.244.1.120   kubernetes-worker1   <none>           <none>
[root@kubernetes-master ~/practice/healthy] ens33 = 192.168.146.91
#

```



#### readiness

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: log-collection
spec:
  selector:
    matchLabels:
      app: log-collection
  replicas: 0
  template:
    metadata:
      labels:
        app: log-collection
    spec:
      containers:
        - name: log-collection
          image: quay.io/qxu/log-collection:health-demo
          ports:
            - containerPort: 8080
          readinessProbe:
            httpGet:
              port: 8090
              path: /
            initialDelaySeconds: 3
            periodSeconds: 3

      
```



svc

```
apiVersion: v1
kind: Service
metadata:
  name: log-collection
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: log-collection
  type: ClusterIP
```



检查当pod不ready时,对应的endpoints上没有对数据

```
# kubectl get endpoints -n test
NAME             ENDPOINTS   AGE
log-collection               14s

```

删除,或修改readinessProbe为正确的数据,再检查endpoints



**startupProbe**

暂时不演示



## 配置探针

[Probe](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.25/#probe-v1-core) 有很多配置字段，可以使用这些字段精确地控制活跃和就绪检测的行为：

- `initialDelaySeconds`：容器启动后要等待多少秒后才启动存活和就绪探针， 默认是 0 秒，最小值是 0。
- `periodSeconds`：执行探测的时间间隔（单位是秒）。默认是 10 秒。最小值是 1。
- `timeoutSeconds`：探测的超时后等待多少秒。默认值是 1 秒。最小值是 1。
- `successThreshold`：探针在失败后，被视为成功的最小连续成功数。默认值是 1。 存活和启动探测的这个值必须是 1。最小值是 1。
- `failureThreshold`：当探测失败时，Kubernetes 的重试次数。 对存活探测而言，放弃就意味着重新启动容器。 对就绪探测而言，放弃意味着 Pod 会被打上未就绪的标签。默认值是 3。最小值是 1。



### 检测方式

https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/

#### 	定义存活命令

#### 	HTTP探测

### 	TCP 探测







以下实验不做

ingress,各自使用不同的域名，

log1.apps.test.crc.test  log-live.apps.test.crc.test   log-life.apps.test.crc.test

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: log-collection
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: log1.apps.test.crc.test
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: log-collection
            port:
              number: 8080
```



以下demo可不做

下载：fortio进行压测

```
curl -L https://github.com/fortio/fortio/releases/download/v1.20.0/fortio-linux_x64-1.20.0.tgz \
 | sudo tar -C / -xvzpf -
```



```
fortio load -a -c 50 -qps 500 -t 30s "http://log1.apps.test.crc.test/"
```

切换pod 的镜像
