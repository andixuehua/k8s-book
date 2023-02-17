https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/

# 服务（Service）

将运行在一组 [Pods](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/) 上的应用程序公开为网络服务的抽象方法。

使用 Kubernetes，你无需修改应用程序即可使用不熟悉的服务发现机制。 Kubernetes 为 Pod 提供自己的 IP 地址，并为一组 Pod 提供相同的 DNS 名， 并且可以在它们之间进行负载均衡。



## 动机

创建和销毁 Kubernetes [Pod](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/) 以匹配集群的期望状态。 Pod 是非永久性资源。 如果你使用 [Deployment](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/) 来运行你的应用程序，则它可以动态创建和销毁 Pod。

每个 Pod 都有自己的 IP 地址，但是在 Deployment 中，在同一时刻运行的 Pod 集合可能与稍后运行该应用程序的 Pod 集合不同。

这导致了一个问题： 如果一组 Pod（称为“后端”）为集群内的其他 Pod（称为“前端”）提供功能， 那么前端如何找出并跟踪要连接的 IP 地址，以便前端可以使用提供工作负载的后端部分？

进入 **Services**。





## 发布服务（服务类型)

对一些应用的某些部分（如前端），可能希望将其暴露给 Kubernetes 集群外部的 IP 地址。

Kubernetes `ServiceTypes` 允许指定你所需要的 Service 类型，默认是 `ClusterIP`。

`Type` 的取值以及行为如下：

- `ClusterIP`：通过集群的内部 IP 暴露服务，选择该值时服务只能够在集群内部访问。 这也是默认的 `ServiceType`。
- [`NodePort`](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/#type-nodeport)：通过每个节点上的 IP 和静态端口（`NodePort`）暴露服务。 `NodePort` 服务会路由到自动创建的 `ClusterIP` 服务。 通过请求 `<节点 IP>:<节点端口>`，你可以从集群的外部访问一个 `NodePort` 服务。
- [`LoadBalancer`](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/#loadbalancer)：使用云提供商的负载均衡器向外部暴露服务。 外部负载均衡器可以将流量路由到自动创建的 `NodePort` 服务和 `ClusterIP` 服务上。
- [`ExternalName`](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/#externalname)：通过返回 `CNAME` 和对应值，可以将服务映射到 `externalName` 字段的内容（例如，`foo.bar.example.com`）。 无需创建任何类型代理。



建立以下服务,进入pod中curl http://whomai:8080 或ingress name查看　pod的负载切换

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: whoami
spec:
  selector:
    matchLabels:
      app: whoami
  replicas: 2
  template:
    metadata:
      labels:
        app: whoami
    spec:
      containers:
        - name: whoami
          image: quay.io/qxu/spring-boot-whoami
          ports:
            - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: whoami
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: whoami
  type: ClusterIP

---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: whoami
spec:
  ingressClassName: nginx 
  rules:
  - host: "whoami.user1.apps.taikang1.local"
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: whoami
            port:
              number: 8080
```



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
```





service 1 clusterIP

```
apiVersion: v1
kind: Service
metadata:
  name: log-collection1
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: log-collection
  type: ClusterIP
```



### NodePort 类型

如果你将 `type` 字段设置为 `NodePort`，则 Kubernetes 控制平面将在 `--service-node-port-range` 标志指定的范围内分配端口（默认值：30000-32767）。 每个节点将那个端口（每个节点上的相同端口号）代理到你的服务中。 你的服务在其 `.spec.ports[*].nodePort` 字段中要求分配的端口。

如果你想指定特定的 IP 代理端口，则可以设置 kube-proxy 中的 `--nodeport-addresses` 参数或者将 [kube-proxy 配置文件](https://kubernetes.io/zh-cn/docs/reference/config-api/kube-proxy-config.v1alpha1/) 中的等效 `nodePortAddresses` 字段设置为特定的 IP 块。 该标志采用逗号分隔的 IP 块列表（例如，`10.0.0.0/8`、`192.0.2.0/25`）来指定 kube-proxy 应该认为是此节点本地的 IP 地址范围。

例如，如果你使用 `--nodeport-addresses=127.0.0.0/8` 标志启动 kube-proxy， 则 kube-proxy 仅选择 NodePort Services 的本地回路接口。 `--nodeport-addresses` 的默认值是一个空列表。 这意味着 kube-proxy 应该考虑 NodePort 的所有可用网络接口。 （这也与早期的 Kubernetes 版本兼容）。

如果需要特定的端口号，你可以在 `nodePort` 字段中指定一个值。 控制平面将为你分配该端口或报告 API 事务失败。 这意味着你需要自己注意可能发生的端口冲突。 你还必须使用有效的端口号，该端口号在配置用于 NodePort 的范围内。

使用 NodePort 可以让你自由设置自己的负载均衡解决方案， 配置 Kubernetes 不完全支持的环境， 甚至直接暴露一个或多个节点的 IP。

service 2: nodeport

```
apiVersion: v1
kind: Service
metadata:
  name: log-collection2
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
    nodePort: 30007
  selector:
    app: log-collection
  type: NodePort
```



go to pod所在　node to check port(在ipvs中不适用,找不到对应的port 3007)

```
ss -ntlp
```



ipvs方式查询

```
[root@master1 ~]# ipvsadm|grep 30007
TCP  master1.taikang1.local:30007 rr
TCP  master1.taikang1.local:30007 rr
TCP  master1.taikang1.local:30007 rr

```



## 无头服务（Headless Services）

有时不需要或不想要负载均衡，以及单独的 Service IP。 遇到这种情况，可以通过指定 Cluster IP（`spec.clusterIP`）的值为 `"None"` 来创建 `Headless` Service。

你可以使用一个无头 Service 与其他服务发现机制进行接口，而不必与 Kubernetes 的实现捆绑在一起。

对于无头 `Services` 并不会分配 Cluster IP，kube-proxy 不会处理它们， 而且平台也不会为它们进行负载均衡和路由。 DNS 如何实现自动配置，依赖于 Service 是否定义了选择算符。



```
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx 
  serviceName: "nginx"
  replicas: 3 # by default is 1
  template:
    metadata:
      labels:
        app: nginx 
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
          name: web
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx
spec:
  rules:
  - host: "nginx.user1.apps.taikang1.local"
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: nginx
            port:
              number: 80

```

```
注意,这里如果ping svc返回的pod不是web-0.xxx的格式,而是ip-.xx 的格式,要看一svc的名称,不要带-.

```



go to another pod to test

```
ping headless-svc name, it's ok , it's different with normal svc

```



service3 : map external service into k8s

```
apiVersion: v1
kind: Service
metadata:
  name: reg-service
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
---
apiVersion: v1
kind: Endpoints
metadata:
  name: reg-service
subsets:
  - addresses:
      - ip: 192.168.146.90
    ports:
      - port: 80

```





## 虚拟 IP 和 Service 代理

在 Kubernetes 集群中，每个 Node 运行一个 `kube-proxy` 进程。 `kube-proxy` 负责为 Service 实现了一种 VIP（虚拟 IP）的形式，而不是 [`ExternalName`](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/#externalname) 的形式。



### iptables 代理模式

这种模式，`kube-proxy` 会监视 Kubernetes 控制节点对 Service 对象和 Endpoints 对象的添加和移除。 对每个 Service，它会配置 iptables 规则，从而捕获到达该 Service 的 `clusterIP` 和端口的请求，进而将请求重定向到 Service 的一组后端中的某个 Pod 上面。 对于每个 Endpoints 对象，它也会配置 iptables 规则，这个规则会选择一个后端组合。

默认的策略是，kube-proxy 在 iptables 模式下随机选择一个后端。

使用 iptables 处理流量具有较低的系统开销，因为流量由 Linux netfilter 处理， 而无需在用户空间和内核空间之间切换。 这种方法也可能更可靠。

如果 kube-proxy 在 iptables 模式下运行，并且所选的第一个 Pod 没有响应，则连接失败。 这与用户空间模式不同：在这种情况下，kube-proxy 将检测到与第一个 Pod 的连接已失败， 并会自动使用其他后端 Pod 重试。

你可以使用 Pod [就绪探测器](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/pod-lifecycle/#container-probes) 验证后端 Pod 可以正常工作，以便 iptables 模式下的 kube-proxy 仅看到测试正常的后端。 这样做意味着你避免将流量通过 kube-proxy 发送到已知已失败的 Pod。

```
iptables -L　查看
```



### IPVS 代理模式[ ](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/#proxy-mode-ipvs)

**特性状态：** `Kubernetes v1.11 [stable]`

在 `ipvs` 模式下，kube-proxy 监视 Kubernetes 服务和端点，调用 `netlink` 接口相应地创建 IPVS 规则， 并定期将 IPVS 规则与 Kubernetes 服务和端点同步。该控制循环可确保 IPVS 状态与所需状态匹配。访问服务时，IPVS 将流量定向到后端 Pod 之一。

IPVS 代理模式基于类似于 iptables 模式的 netfilter 挂钩函数， 但是使用哈希表作为基础数据结构，并且在内核空间中工作。 这意味着，与 iptables 模式下的 kube-proxy 相比，IPVS 模式下的 kube-proxy 重定向通信的延迟要短，并且在同步代理规则时具有更好的性能。 与其他代理模式相比，IPVS 模式还支持更高的网络流量吞吐量。

IPVS 提供了更多选项来平衡后端 Pod 的流量。这些是：

- `rr`：轮替（Round-Robin）
- `lc`：最少链接（Least Connection），即打开链接数量最少者优先
- `dh`：目标地址哈希（Destination Hashing）
- `sh`：源地址哈希（Source Hashing）
- `sed`：最短预期延迟（Shortest Expected Delay）
- `nq`：从不排队（Never Queue）

**说明：**

要在 IPVS 模式下运行 kube-proxy，必须在启动 kube-proxy 之前使 IPVS 在节点上可用。

当 kube-proxy 以 IPVS 代理模式启动时，它将验证 IPVS 内核模块是否可用。 如果未检测到 IPVS 内核模块，则 kube-proxy 将退回到以 iptables 代理模式运行。





#### svc会话保持

选部署一个应用

```
kubectl create ns test
kubectl create deployment whoami --image=quay.io/qxu/spring-boot-whoami 
kubectl scale deployment whoami --replicas=2 -n test
kubectl expose deployment whoami --port=8080 --target-port=8080 -n test

```



```
kubectl patch svc/whoami --patch '{"spec": {"sessionAffinity": "ClientIP"}}'  

取消
kubectl patch svc whoami -p '{"spec":{"sessionAffinity":"None"}}'    #取消session 


#test one pod's console to see the IP
kubectl exec -it log-collection-7cc7dd4bf9-mqgw6 /bin/bash 
curl whoami:8080
```







使用calico+ipvs方式,可以ping 通svc的clusterip

https://www.cnblogs.com/lizexiong/p/14776517.html

iptables：clusterIP只是iptables中的规则，只会处理ip:port四层数据包，reject了icmp。不能 ping通。

　　IPVS：ipvs依赖iptables进行包过滤、SNAT、masquared(伪装)。 使用 ipset 来存储需要 DROP 或 masquared 的流量的源或目标地址，以确保 iptables 规则的数量是恒定的，这样我们就不需要关心我们有多少服务了

　　二者有着本质的差别：iptables是为防火墙而设计的；IPVS则专门用于高性能负载均衡，并使用更高效的数据结构（Hash表），允许几乎无限的规模扩张。



原因是ipvs在每个节点上会给svc建立出来个真实的ip出来,　iptables只是一个４层的转发规则,不存在真实的ip所以不能ping



```
[root@master1 ~]# ip a|grep ipvs
5: kube-ipvs0: <BROADCAST,NOARP> mtu 1500 qdisc noop state DOWN group default 
    inet 10.233.26.70/32 scope global kube-ipvs0
    inet 10.233.0.3/32 scope global kube-ipvs0
    inet 10.233.0.1/32 scope global kube-ipvs0
    inet 10.233.28.141/32 scope global kube-ipvs0
    inet 10.233.48.252/32 scope global kube-ipvs0
    inet 10.233.41.220/32 scope global kube-ipvs0
    inet 10.233.34.161/32 scope global kube-ipvs0
    inet 10.233.51.13/32 scope global kube-ipvs0
    inet 10.233.33.27/32 scope global kube-ipvs0
    inet 10.233.22.62/32 scope global kube-ipvs0
    inet 10.233.49.63/32 scope global kube-ipvs0
    inet 10.233.30.136/32 scope global kube-ipvs0
    inet 10.233.15.169/32 scope global kube-ipvs0
    inet 10.233.23.140/32 scope global kube-ipvs0
    inet 10.233.58.127/32 scope global kube-ipvs0
    inet 10.233.7.145/32 scope global kube-ipvs0
    inet 10.233.36.2/32 scope global kube-ipvs0
    inet 10.233.26.146/32 scope global kube-ipvs0
[root@master1 ~]# ip a|grep ipvs|grep 10.233.26.146
    inet 10.233.26.146/32 scope global kube-ipvs0
[root@master1 ~]# 

```

