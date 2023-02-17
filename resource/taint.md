

https://kubernetes.io/zh-cn/docs/concepts/scheduling-eviction/taint-and-toleration/

# 污点和容忍度

[节点亲和性](https://kubernetes.io/zh-cn/docs/concepts/scheduling-eviction/assign-pod-node/#affinity-and-anti-affinity) 是 [Pod](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/) 的一种属性，它使 Pod 被吸引到一类特定的[节点](https://kubernetes.io/zh-cn/docs/concepts/architecture/nodes/) （这可能出于一种偏好，也可能是硬性要求）。 **污点（Taint）** 则相反——它使节点能够排斥一类特定的 Pod。

**容忍度（Toleration）** 是应用于 Pod 上的。容忍度允许调度器调度带有对应污点的 Pod。 容忍度允许调度但并不保证调度：作为其功能的一部分， 调度器也会[评估其他参数](https://kubernetes.io/zh-cn/docs/concepts/scheduling-eviction/pod-priority-preemption/)。

污点和容忍度（Toleration）相互配合，可以用来避免 Pod 被分配到不合适的节点上。 每个节点上都可以应用一个或多个污点，这表示对于那些不能容忍这些污点的 Pod， 是不会被该节点接受的。



## 使用例子

通过污点和容忍度，可以灵活地让 Pod **避开**某些节点或者将 Pod 从某些节点驱逐。 下面是几个使用例子：

- **专用节点**：如果想将某些节点专门分配给特定的一组用户使用，你可以给这些节点添加一个污点（即， `kubectl taint nodes nodename dedicated=groupName:NoSchedule`）， 然后给这组用户的 Pod 添加一个相对应的容忍度 （通过编写一个自定义的[准入控制器](https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/admission-controllers/)， 很容易就能做到）。 拥有上述容忍度的 Pod 就能够被调度到上述专用节点，同时也能够被调度到集群中的其它节点。 如果你希望这些 Pod 只能被调度到上述专用节点， 那么你还需要给这些专用节点另外添加一个和上述污点类似的 label （例如：`dedicated=groupName`）， 同时还要在上述准入控制器中给 Pod 增加节点亲和性要求，要求上述 Pod 只能被调度到添加了 `dedicated=groupName` 标签的节点上。

- **配备了特殊硬件的节点**：在部分节点配备了特殊硬件（比如 GPU）的集群中， 我们希望不需要这类硬件的 Pod 不要被调度到这些特殊节点，以便为后继需要这类硬件的 Pod 保留资源。 要达到这个目的，可以先给配备了特殊硬件的节点添加污点 （例如 `kubectl taint nodes nodename special=true:NoSchedule` 或 `kubectl taint nodes nodename special=true:PreferNoSchedule`)， 然后给使用了这类特殊硬件的 Pod 添加一个相匹配的容忍度。 和专用节点的例子类似，添加这个容忍度的最简单的方法是使用自定义 [准入控制器](https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/admission-controllers/)。 比如，我们推荐使用[扩展资源](https://kubernetes.io/zh-cn/docs/concepts/configuration/manage-resources-containers/#extended-resources) 来表示特殊硬件，给配置了特殊硬件的节点添加污点时包含扩展资源名称， 然后运行一个 [ExtendedResourceToleration](https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/admission-controllers/#extendedresourcetoleration) 准入控制器。此时，因为节点已经被设置污点了，没有对应容忍度的 Pod 不会被调度到这些节点。 但当你创建一个使用了扩展资源的 Pod 时，`ExtendedResourceToleration` 准入控制器会自动给 Pod 加上正确的容忍度，这样 Pod 就会被自动调度到这些配置了特殊硬件的节点上。 这种方式能够确保配置了特殊硬件的节点专门用于运行需要这些硬件的 Pod， 并且你无需手动给这些 Pod 添加容忍





- #### 节点的taint

```
kubectl taint nodes worker2.taikang1.local  team=beijing:NoSchedule

下面这个一般不要使用，会驱离现有的pod
#下面的配置会驱离pod，并产生大量的Terminating pod,如果 重新加入tolerations后，这些并产生大量的Terminating pod 会自动删除
kubectl taint nodes node1 team2=beijing:NoExecute

#观察所有的3个pod都不会落到带有污点的节点上
```



- taint.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example
spec:
  selector:
    matchLabels:
      app: hello-openshift
  replicas: 3 
  template:
    metadata:
      labels:
        app: hello-openshift
    spec:
      containers:
        - name: hello-openshift
          image: openshift/hello-openshift
          ports:
            - containerPort: 8080
```



- #### 部署的toleration

```shell
oc project test-node
oc apply -f toleration.yaml

oc get pod -o wide

#delete taint from node,最后请删除节点的污点
kubectl taint nodes worker2.taikang1.local team:NoSchedule-
kubectl taint nodes worker2.taikang1.local  team2:NoExecute-

```



- toleration.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: log-collection
spec:
  selector:
    matchLabels:
      app: log-collection
  replicas: 4
  template:
    metadata:
      labels:
        app: log-collection
    spec:
      tolerations:
      - key: "team"
        operator: "Equal"
        value: "beijing"
        effect: "NoSchedule"

      containers:
        - name: log-collection
          image: quay.io/qxu/log-collection
          ports:
            - containerPort: 8080

```



#### 万能toleration:

存在两种特殊情况：

如果一个容忍度的 `key` 为空且 `operator` 为 `Exists`， 表示这个容忍度与任意的 key、value 和 effect 都匹配，即这个容忍度能容忍任何污点。

如果 `effect` 为空，则可以与所有键名 `key1` 的效果相匹配。

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: log-collection
spec:
  selector:
    matchLabels:
      app: log-collection
  replicas: 4
  template:
    metadata:
      labels:
        app: log-collection
    spec:
      tolerations:
      - operator: "Exists"

      containers:
        - name: log-collection
          image: quay.io/qxu/log-collection
          ports:
            - containerPort: 8080            
```



#### NoExecute

通常情况下，如果给一个节点添加了一个 effect 值为 `NoExecute` 的污点， 则任何不能忍受这个污点的 Pod 都会马上被驱逐，任何可以忍受这个污点的 Pod 都不会被驱逐。 但是，如果 Pod 存在一个 effect 值为 `NoExecute` 的容忍度指定了可选属性 `tolerationSeconds` 的值，则表示在给节点添加了上述污点之后， Pod 还能继续在节点上运行的时间。例如，

```
kubectl taint nodes node1 key1=value1:NoExecute

```



#### cordon node

要标记一个 Node 为不可调度，执行以下命令

https://kubernetes.io/zh-cn/docs/concepts/architecture/nodes/

```
kubectl cordon node/worker2.taikang1.local



kubectl get node

NAME                     STATUS                     ROLES           AGE    VERSION
master1.taikang1.local   Ready                      control-plane   101d   v1.24.6
master2.taikang1.local   Ready                      control-plane   101d   v1.24.6
master3.taikang1.local   Ready                      control-plane   101d   v1.24.6
worker1.taikang1.local   Ready                      worker          101d   v1.24.6
worker2.taikang1.local   Ready,SchedulingDisabled   worker          101d   v1.24.6
worker3.taikang1.local   Ready                      worker          101d   v1.24.6

kubectl uncordon node/worker2.taikang1.local
```



#### drain node

清空一个节点

https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/safely-drain-node/

```
Drain node in preparation for maintenance.

 The given node will be marked unschedulable to prevent new pods from arriving. 'drain' evicts the pods if the API
server supports https://kubernetes.io/docs/concepts/workloads/pods/disruptions/ . Otherwise, it will use normal DELETE
to delete the pods. The 'drain' evicts or deletes all pods except mirror pods (which cannot be deleted through the API
server).  If there are daemon set-managed pods, drain will not proceed without --ignore-daemonsets, and regardless it
will not delete any daemon set-managed pods, because those pods would be immediately replaced by the daemon set
controller, which ignores unschedulable markings.  If there are any pods that are neither mirror pods nor managed by a
replication controller, replica set, daemon set, stateful set, or job, then drain will not delete any pods unless you
use --force.  --force will also allow deletion to proceed if the managing resource of one or more pods is missing.

 'drain' waits for graceful termination. You should not operate on the machine until the command completes.

```

