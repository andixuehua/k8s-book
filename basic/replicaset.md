https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/replicaset/

# ReplicaSet

ReplicaSet 的目的是维护一组在任何时候都处于运行状态的 Pod 副本的稳定集合。 因此，它通常用来保证给定数量的、完全相同的 Pod 的可用性。

一般不会直接使用,通过deployment, statefulset创建产生





## ReplicaSet 的替代方案

### Deployment（推荐）

[`Deployment`](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/) 是一个可以拥有 ReplicaSet 并使用声明式方式在服务器端完成对 Pod 滚动更新的对象。 尽管 ReplicaSet 可以独立使用，目前它们的主要用途是提供给 Deployment 作为编排 Pod 创建、删除和更新的一种机制。当使用 Deployment 时，你不必关心如何管理它所创建的 ReplicaSet，Deployment 拥有并管理其 ReplicaSet。 因此，建议你在需要 ReplicaSet 时使用 Deployment。

### 裸 Pod

与用户直接创建 Pod 的情况不同，ReplicaSet 会替换那些由于某些原因被删除或被终止的 Pod，例如在节点故障或破坏性的节点维护（如内核升级）的情况下。 因为这个原因，我们建议你使用 ReplicaSet，即使应用程序只需要一个 Pod。 想像一下，ReplicaSet 类似于进程监视器，只不过它在多个节点上监视多个 Pod， 而不是在单个节点上监视单个进程。 ReplicaSet 将本地容器重启的任务委托给了节点上的某个代理（例如，Kubelet）去完成。

### Job

使用[`Job`](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/job/) 代替 ReplicaSet， 可以用于那些期望自行终止的 Pod。

### DaemonSet

对于管理那些提供主机级别功能（如主机监控和主机日志）的容器， 就要用 [`DaemonSet`](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/daemonset/) 而不用 ReplicaSet。 这些 Pod 的寿命与主机寿命有关：这些 Pod 需要先于主机上的其他 Pod 运行， 并且在机器准备重新启动/关闭时安全地终止。

### ReplicationController

ReplicaSet 是 [ReplicationController](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/replicationcontroller/) 的后继者。二者目的相同且行为类似，只是 ReplicationController 不支持 [标签用户指南](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/labels/#label-selectors) 中讨论的基于集合的选择算符需求。 因此，相比于 ReplicationController，应优先考虑 ReplicaSet。



查看部署历史

```
# kubectl rollout history deployment log-collection -n test
deployment.apps/log-collection 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
3         <none>
4         <none>
```

查看对应的replicaset

```
# kubectl get replicaset -n test
NAME                        DESIRED   CURRENT   READY   AGE
log-collection-684c4bf949   1         1         1       21m
log-collection-6fd5d6d49b   0         0         0       24m
log-collection-9569d6d8d    0         0         0       40m
log-collection-bbc67b77d    0         0         0       37m

```

查看对应的revision

```
kubectl get replicaset log-collection-684c4bf949 -o yaml -n test|grep revision
    deployment.kubernetes.io/revision: "4"



```

修改一下image version,再rollback 回去demo一下



rollback to version

```
kubectl rollout undo deployment log-collection --to-revision=2 -n test
```

 检查状态

```
# kubectl get pod -n test -w
再次rollback
kubectl rollout undo deployment log-collection --to-revision=4 -n test
```

S
