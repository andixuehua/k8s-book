

https://kubernetes.io/zh-cn/docs/tasks/run-application/horizontal-pod-autoscale/

# Pod 水平自动扩缩

在 Kubernetes 中，*HorizontalPodAutoscaler* 自动更新工作负载资源 （例如 [Deployment](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/) 或者 [StatefulSet](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/statefulset/)）， 目的是自动扩缩工作负载以满足需求。

水平扩缩意味着对增加的负载的响应是部署更多的 [Pods](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/)。 这与 “垂直（Vertical）” 扩缩不同，对于 Kubernetes， 垂直扩缩意味着将更多资源（例如：内存或 CPU）分配给已经为工作负载运行的 Pod。

如果负载减少，并且 Pod 的数量高于配置的最小值， HorizontalPodAutoscaler 会指示工作负载资源（ Deployment、StatefulSet 或其他类似资源）缩减。

水平 Pod 自动扩缩不适用于无法扩缩的对象（例如：[DaemonSet](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/daemonset/)。）

HorizontalPodAutoscaler 被实现为 Kubernetes API 资源和[控制器](https://kubernetes.io/zh-cn/docs/concepts/architecture/controller/)。

资源决定了控制器的行为。在 Kubernetes [控制平面](https://kubernetes.io/zh-cn/docs/reference/glossary/?all=true#term-control-plane)内运行的水平 Pod 自动扩缩控制器会定期调整其目标（例如：Deployment）的所需规模，以匹配观察到的指标， 例如，平均 CPU 利用率、平均内存利用率或你指定的任何其他自定义指标。





- ### 实现横向扩缩容前提条件
  
  - pod内设置了resource limits
  - autoscale配置了： 最大/最小pod数量，及自动扩缩容触发条件

- 触发扩容

```
#go to one pod 
stress --cpu 1 --io 4 --vm 2 --vm-bytes 128M --timeout 120s

#check pod number and hpa status
oc get hpa -w
```



- stress-deploy.yaml

```
kind: Deployment
apiVersion: apps/v1
metadata:
  name: http-stress
  labels:
    app: http-stress
spec:
  replicas: 1
  selector:
    matchLabels:
      deployment: http-stress
  template:
    metadata:
      labels:
        deployment: http-stress
    spec:
      containers:
        - name: http-stress
          image: quay.io/openshift-scale/http-stress
          resources: {}
          imagePullPolicy: IfNotPresent
          resources:
            limits:
              cpu: "0.1"
              memory: 512Mi
            requests:
              cpu: 100m
              memory: 512Mi
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      dnsPolicy: ClusterFirst
      securityContext: {}

```



```shell
kubectl autoscale deployment  http-stress --cpu-percent=50 --min=1 --max=4 -n test
```

```
kind: HorizontalPodAutoscaler
apiVersion: autoscaling/v2beta2
metadata:
  name: http-stress
spec:
  scaleTargetRef:
    kind: Deployment
    name: http-stress
    apiVersion: apps/v1
  minReplicas: 1
  maxReplicas: 4
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50


```



check status

```
kubectl get hpa
NAME          REFERENCE                TARGETS    MINPODS   MAXPODS   REPLICAS   AGE
http-stress   Deployment/http-stress   100%/50%   1         4         2          88d


 kubectl  top pod
NAME                           CPU(cores)   MEMORY(bytes)   
http-stress-57ff58cfd4-gjh8n   34m          56Mi            
http-stress-57ff58cfd4-t8ddk   99m          243Mi  
```



参考： https://www.imooc.com/article/275928



```
HPA伸缩过程及算法


1. 收集该HPA控制下所有Pod最近的cpu使用情况（CPU utilization）

2. 对比在扩容条件里记录的cpu限额（CPUUtilization）

3. 调整实例数（必须要满足不超过最大/最小实例数）

每隔30s做一次自动扩容的判断
说明：

4.CPU utilization的计算方法是用cpu usage（最近一分钟的平均值，通过heapster可以直接获取到）除以cpu request（这里cpu request就是我们在创建容器时制定的cpu使用核心数）得到一个平均值，这个平均值可以理解为：平均每个Pod CPU核心的使用占比。

最重要的步骤为3，这里即为HPA的算法，计算当前需要启动几个Pod



```



# 资源指标管道

https://kubernetes.io/zh-cn/docs/tasks/debug/debug-cluster/resource-metrics-pipeline/



对于 Kubernetes，**Metrics API** 提供了一组基本的指标，以支持自动伸缩和类似的用例。 该 API 提供有关节点和 Pod 的资源使用情况的信息， 包括 CPU 和内存的指标。如果将 Metrics API 部署到集群中， 那么 Kubernetes API 的客户端就可以查询这些信息，并且可以使用 Kubernetes 的访问控制机制来管理权限。

[HorizontalPodAutoscaler](https://kubernetes.io/zh-cn/docs/tasks/run-application/horizontal-pod-autoscale/) (HPA) 和 [VerticalPodAutoscaler](https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler#readme) (VPA) 使用 metrics API 中的数据调整工作负载副本和资源，以满足客户需求。

你也可以通过 [`kubectl top`](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#top) 命令来查看资源指标。
