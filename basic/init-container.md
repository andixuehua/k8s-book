https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/init-containers/



- # Init 容器

  本页提供了 Init 容器的概览。Init 容器是一种特殊容器，在 [Pod](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/) 内的应用容器启动之前运行。Init 容器可以包括一些应用镜像中不存在的实用工具和安装脚本。
  
  你可以在 Pod 的规约中与用来描述应用容器的 `containers` 数组平行的位置指定 Init 容器。
  
  
  
  
  
  ## 理解 Init 容器
  
  每个 [Pod](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/) 中可以包含多个容器， 应用运行在这些容器里面，同时 Pod 也可以有一个或多个先于应用容器启动的 Init 容器。
  
  Init 容器与普通的容器非常像，除了如下两点：
  
  - 它们总是运行到完成。
  - 每个都必须在下一个启动之前成功完成。
  
  
  
  ### 与普通容器的不同之处
  
  Init 容器支持应用容器的全部字段和特性，包括资源限制、数据卷和安全设置。 然而，Init 容器对资源请求和限制的处理稍有不同，在下面[资源](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/init-containers/#resources)节有说明。
  
  同时 Init 容器不支持 `lifecycle`、`livenessProbe`、`readinessProbe` 和 `startupProbe`， 因为它们必须在 Pod 就绪之前运行完成。
  
  如果为一个 Pod 指定了多个 Init 容器，这些容器会按顺序逐个运行。 每个 Init 容器必须运行成功，下一个才能够运行。当所有的 Init 容器运行完成时， Kubernetes 才会为 Pod 初始化应用容器并像平常一样运行。
  
  
  
  
  
- ### initContainers

  常用于主容器启动前的初始化工作，比如，可用于配置文件初始化，数据库检查，依赖的服务检查

```shell
kubectl new-project test-pod
kubectl apply -f pod.yaml


#观察 pod的状态,应该不能正常启动，因为initContainers 中要求的两个服务还不存在
kubectl get pod

＃稍后再执行以下操作，当svc建立后，pod就会正常运行
kubectl apply -f svc.yaml
```

- pod.yaml ,注意下面的grep,这是根据tk的环境　进行改造的,在其他环境中不需要grep	

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox
    command: ['sh', '-c', 'until nslookup myservice|grep Name; do echo waiting for myservice; sleep 2; done;']
  - name: init-mydb
    image: busybox
    command: ['sh', '-c', 'until nslookup mydb|grep Name; do echo waiting for mydb; sleep 2; done;']
```

- svc.yaml

```yaml
kind: Service
apiVersion: v1
metadata:
  name: myservice
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
---
kind: Service
apiVersion: v1
metadata:
  name: mydb
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9377
```



