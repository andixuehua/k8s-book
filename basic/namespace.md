https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/namespaces/

## namspace

```
# 位于名字空间中的资源
kubectl api-resources --namespaced=true

# 不在名字空间中的资源
kubectl api-resources --namespaced=false
```



Kubernetes 会创建四个初始名字空间：

- `default` 没有指明使用其它名字空间的对象所使用的默认名字空间
- `kube-system` Kubernetes 系统创建对象所使用的名字空间
- `kube-public` 这个名字空间是自动创建的，所有用户（包括未经过身份验证的用户）都可以读取它。 这个名字空间主要用于集群使用，以防某些资源在整个集群中应该是可见和可读的。 这个名字空间的公共方面只是一种约定，而不是要求。
- `kube-node-lease` 此名字空间用于与各个节点相关的 [租约（Lease）](https://kubernetes.io/docs/reference/kubernetes-api/cluster-resources/lease-v1/)对象。 节点租期允许 kubelet 发送[心跳](https://kubernetes.io/zh-cn/docs/concepts/architecture/nodes/#heartbeats)，由此控制面能够检测到节点故障。



## 名字空间和 DNS

当你创建一个[服务](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/)时， Kubernetes 会创建一个相应的 [DNS 条目](https://kubernetes.io/zh-cn/docs/concepts/services-networking/dns-pod-service/)。

该条目的形式是 `<服务名称>.<名字空间名称>.svc.cluster.local`，这意味着如果容器只使用 `<服务名称>`，它将被解析到本地名字空间的服务。这对于跨多个名字空间（如开发、测试和生产） 使用相同的配置非常有用。如果你希望跨名字空间访问，则需要使用完全限定域名（FQDN）。



```
kubectl create namespace test123
kubectl get namespaces test123
kubectl describe namespaces test123
kubectl delete namespaces test123


#ceate ns by yaml
apiVersion: v1
kind: Namespace
metadata:
  name: test123
```





## 使用 Kubernetes 名字空间细分你的集群[ ](https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/namespaces/#使用-kubernetes-名字空间细分你的集群)

理解 default 名字空间

默认情况下，Kubernetes 集群会在配置集群时实例化一个 default 名字空间，用以存放集群所使用的默认 Pod、Service 和 Deployment 集合。





## 理解使用名字空间的动机

单个集群应该能满足多个用户及用户组的需求（以下称为 “用户社区”）。

Kubernetes **名字空间** 帮助不同的项目、团队或客户去共享 Kubernetes 集群。

名字空间通过以下方式实现这点：

1. 为[名字](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/names/)设置作用域.
2. 为集群中的部分资源关联鉴权和策略的机制。

使用多个名字空间是可选的。

每个用户社区都希望能够与其他社区隔离开展工作。

每个用户社区都有自己的：

1. 资源（Pod、服务、副本控制器等等）
2. 策略（谁能或不能在他们的社区里执行操作）
3. 约束（该社区允许多少配额等等）

集群运营者可以为每个唯一用户社区创建名字空间。

名字空间为下列内容提供唯一的作用域：

1. 命名资源（避免基本的命名冲突）
2. 将管理权限委派给可信用户
3. 限制社区资源消耗的能力

用例包括:

1. 作为集群运营者, 我希望能在单个集群上支持多个用户社区。
2. 作为集群运营者，我希望将集群分区的权限委派给这些社区中的受信任用户。
3. 作为集群运营者，我希望能限定每个用户社区可使用的资源量，以限制对使用同一集群的其他用户社区的影响。
4. 作为集群用户，我希望与我的用户社区相关的资源进行交互，而与其他用户社区在该集群上执行的操作无关。