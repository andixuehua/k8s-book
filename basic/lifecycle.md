https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/pod-lifecycle/

# Pod 的生命周期

Pod 遵循一个预定义的生命周期，起始于 `Pending` [阶段](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/pod-lifecycle/#pod-phase)，如果至少 其中有一个主要容器正常启动，则进入 `Running`，之后取决于 Pod 中是否有容器以 失败状态结束而进入 `Succeeded` 或者 `Failed` 阶段。



下面是 `phase` 可能的值：

| 取值                | 描述                                                         |
| :------------------ | :----------------------------------------------------------- |
| `Pending`（悬决）   | Pod 已被 Kubernetes 系统接受，但有一个或者多个容器尚未创建亦未运行。此阶段包括等待 Pod 被调度的时间和通过网络下载镜像的时间。 |
| `Running`（运行中） | Pod 已经绑定到了某个节点，Pod 中所有的容器都已被创建。至少有一个容器仍在运行，或者正处于启动或重启状态。 |
| `Succeeded`（成功） | Pod 中的所有容器都已成功终止，并且不会再重启。               |
| `Failed`（失败）    | Pod 中的所有容器都已终止，并且至少有一个容器是因为失败终止。也就是说，容器以非 0 状态退出或者被系统终止。 |
| `Unknown`（未知）   | 因为某些原因无法取得 Pod 的状态。这种情况通常是因为与 Pod 所在主机通信失败。 |



# 为容器的生命周期事件设置处理函数

https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/attach-handler-lifecycle-event/

Kubernetes 支持 postStart 和 preStop 事件。 当一个容器启动后，Kubernetes 将立即发送 postStart 事件；在容器被终结之前， Kubernetes 将发送一个 preStop 事件。容器可以为每个事件指定一个处理程序。



init-comtainer相当于prestart

```
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
      containers:
        - name: log-collection
          image: >-
            quay.io/qxu/log-collection:v3
          ports:
            - containerPort: 8080
              protocol: TCP
          resources: {}
          lifecycle:
            postStart:
              exec:
                command:
                  - /bin/bash
                  - '-c'
                  - sleep 30
            preStop:
              exec:
                command:
                  - /bin/bash
                  - '-c'
                  - sleep 120
          imagePullPolicy: IfNotPresent
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      dnsPolicy: ClusterFirst
      securityContext: {}
```

观察上面pod的创建时间会大于30s,

```
# kubectl get pod -n test -w
NAME                              READY   STATUS              RESTARTS   AGE
counter-745d94ddfc-hdtx6          2/2     Running             2          4h10m
log-collection-6b4d8b7879-h8lnc   0/1     ContainerCreating   0          32s
nginx                             1/1     Running             1          6h25m
log-collection-6b4d8b7879-h8lnc   1/1     Running             0          32s

```



执行下面的命令,观察terminating 的时间会比较长

```
kubectl scale deployment log-collection -n test --replicas=0


# kubectl get pod -n test -w
NAME                              READY   STATUS        RESTARTS   AGE
counter-745d94ddfc-hdtx6          2/2     Running       2          4h6m
log-collection-5dc8565785-pphn9   1/1     Terminating   0          91s
nginx                             1/1     Running       1          6h20m
log-collection-5dc8565785-pphn9   0/1     Terminating   0          2m1s
log-collection-5dc8565785-pphn9   0/1     Terminating   0          2m2s
log-collection-5dc8565785-pphn9   0/1     Terminating   0          2m2s

```



可以修改postStart和prestop,为sleep 1　后,再进行pod的建立和删除,观观察ContainerCreatingt和Terminating 的时间

