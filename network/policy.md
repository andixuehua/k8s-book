# 网络策略

https://kubernetes.io/zh-cn/docs/concepts/services-networking/network-policies/

如果你希望在 IP 地址或端口层面（OSI 第 3 层或第 4 层）控制网络流量， 则你可以考虑为集群中特定应用使用 Kubernetes 网络策略（NetworkPolicy）。 NetworkPolicy 是一种以应用为中心的结构，允许你设置如何允许 [Pod](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/) 与网络上的各类网络“实体” （我们这里使用实体以避免过度使用诸如“端点”和“服务”这类常用术语， 这些术语在 Kubernetes 中有特定含义）通信。 NetworkPolicies 适用于一端或两端与 Pod 的连接，与其他连接无关。

Pod 可以通信的 Pod 是通过如下三个标识符的组合来辩识的：

1. 其他被允许的 Pods（例外：Pod 无法阻塞对自身的访问）
2. 被允许的名字空间
3. IP 组块（例外：与 Pod 运行所在的节点的通信总是被允许的， 无论 Pod 或节点的 IP 地址）

在定义基于 Pod 或名字空间的 NetworkPolicy 时， 你会使用[选择算符](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/labels/)来设定哪些流量可以进入或离开与该算符匹配的 Pod。 另外，当创建基于 IP 的 NetworkPolicy 时，我们基于 IP 组块（CIDR 范围）来定义策略。

## 前置条件

网络策略通过[网络插件](https://kubernetes.io/zh-cn/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/)来实现。 要使用网络策略，你必须使用支持 NetworkPolicy 的网络解决方案。 创建一个 NetworkPolicy 资源对象而没有控制器来使它生效的话，是没有任何作用的。





### 网络策略的常用规则(flannel不支持network policy 功能)

#### 准备测试环境

```
#test-network-1 用于设置networkpolicy, test-network-2用于测试访问
oc new-project test-network-1
oc new-app --docker-image  quay.io/qxu/log-collection:main --name log-collection-1
oc new-app --docker-image  quay.io/qxu/log-collection:main --name log-collection-2

oc new-project test-network-2
oc new-app --docker-image  quay.io/qxu/log-collection:main --name log-collection-1
oc new-app --docker-image  quay.io/qxu/log-collection:main --name log-collection-2
```



in k8s

```
cd networkpolicy
kubectl apply -f dep.yaml -n user2
kubectl apply -f dep.yaml -n user1
kubectl apply -f dep.yaml -n user3
kubectl label namespace user1 access=allow
```



#### 只允许来自相同项目的Pod访问

```shell
＃in test-network-1, create 

kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: allow-same-namespace
spec:
  podSelector: {}
  ingress:
  - from:
    - podSelector: {}
```

测试： 在test-network-2中的pod 中ping test-network-1中的pod ip

#### 只允许从指定的项目访问

```
#label project test-network-2
kubectl label namespace test-deploy access=allow


kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: allow-pod-and-namespace-both
spec:
  podSelector: {}
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              access: allow

```

测试： 在test-network-2中的pod 中ping test-network-1中的pod ip

以上可以并存，注意测试时，如果存在所有namespace可访问的policy,要暂时去掉。(有多个networkpolicy时,权限最大的那个生效)



**如果上面只加了matchLabels,则本ns内的pod也不能访问了,所以也要加上下面的配置,让同项目内的pod可以访问**　

```
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: allow-pod-and-namespace-both
spec:
  podSelector: {}
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              access: allow
        - podSelector: {}

```



#### all from all namespace to access

```
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: allow-from-all-namespaces
spec:
  podSelector: {}
  ingress:
    - from:
        - namespaceSelector: {}
  policyTypes:
    - Ingress
```



对pod和svc分别进行curl,效果一致.

