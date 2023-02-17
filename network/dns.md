https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/coredns/

# 为集群配置 DNS

Kubernetes 提供 DNS 集群插件，大多数支持的环境默认情况下都会启用。 在 Kubernetes 1.11 及其以后版本中，推荐使用 CoreDNS， kubeadm 默认会安装 CoreDNS。

要了解关于如何为 Kubernetes 集群配置 CoreDNS 的更多信息，参阅 [定制 DNS 服务](https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/dns-custom-nameservers/)。 关于如何利用 kube-dns 配置 kubernetes DNS 的演示例子，参阅 [Kubernetes DNS 插件示例](https://github.com/kubernetes/examples/tree/master/staging/cluster-dns)。



## 关于 CoreDNS[ ](https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/coredns/#关于-coredns)

[CoreDNS](https://coredns.io/) 是一个灵活可扩展的 DNS 服务器，可以作为 Kubernetes 集群 DNS。 与 Kubernetes 一样，CoreDNS 项目由 [CNCF](https://cncf.io/) 托管。

通过替换现有集群部署中的 kube-dns，或者使用 kubeadm 等工具来为你部署和升级集群， 可以在你的集群中使用 CoreDNS 而非 kube-dns。



查看当前的集群的dns地址

```
# kubectl get svc kube-dns -n kube-system 

NAME       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   233d

 

kubectl get svc coredns -n kube-system 
NAME      TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
coredns   ClusterIP   10.233.0.3   <none>        53/UDP,53/TCP,9153/TCP   11d
```



```
# kubectl get pod -n kube-system
NAME                                        READY   STATUS    RESTARTS   AGE
coredns-6d56c8448f-p29md                    1/1     Running   25         233d
coredns-6d56c8448f-p9ksv                    1/1     Running   26         233d
etcd-kubernetes-master                      1/1     Running   25         233d
```

配置文件

```
# kubectl get cm -n kube-system
NAME                                 DATA   AGE
coredns                              1      233d


# kubectl get cm coredns -n kube-system -o yaml 
apiVersion: v1
data:
  Corefile: |
    .:53 {
        errors
        health {
            lameduck 5s
        }
        ready
        kubernetes cluster.local in-addr.arpa ip6.arpa {
          pods insecure
          fallthrough in-addr.arpa ip6.arpa
        }
        prometheus :9153
        forward . /etc/resolv.conf {
          prefer_udp
          max_concurrent 1000
        }
        cache 30

        loop
        reload
        loadbalance
    }
kind: ConfigMap


```

customizing coredns forward on k8s

https://devops.cisel.ch/customizing-coredns-forwarders-on-kubernetes



#  NodeLocal DNSCache



NodeLocal DNSCache 通过在集群节点上作为 DaemonSet 运行 DNS 缓存代理来提高集群 DNS 性能



如果群群使用了nodelocal dns,pod中的 dns 为设为 nodelocal 中配置的dns　IP,

https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/nodelocaldns/



```
kubectl get daemonset -n kube-system 
NAME                   DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR              AGE
nodelocaldns           6         6         6       6            6           kubernetes.io/os=linux     101d
```

**说明：**

NodeLocal DNSCache 的本地侦听 IP 地址可以是任何地址，只要该地址不和你的集群里现有的 IP 地址发生冲突。 推荐使用本地范围内的地址，例如，IPv4 链路本地区段 '169.254.0.0/16' 内的地址， 或者 IPv6 唯一本地地址区段 'fd00::/8' 内的地址。

**在这里会再转发到上游的 core-dns svc中　 10.233.0.3** 

```
[root@trainee ~]# kubectl get cm nodelocaldns -o yaml  -n kube-system
apiVersion: v1
data:
  Corefile: |
    cluster.local:53 {
        errors
        cache {
            success 9984 30
            denial 9984 5
        }
        reload
        loop
        bind 169.254.25.10
        forward . 10.233.0.3 {
            force_tcp
        }
        prometheus :9253
        health 169.254.25.10:9254
    }
    in-addr.arpa:53 {
        errors
        cache 30
        reload
        loop
        bind 169.254.25.10
        forward . 10.233.0.3 {
            force_tcp
        }
        prometheus :9253

```



nodelocaldns　pod中的/etc/resol.conf的配置与所在node中的/etc/resolv.conf一致



https://kubernetes.io/zh-cn/docs/concepts/services-networking/dns-pod-service/

# Service 与 Pod 的 DNS



Kubernetes 为 Service 和 Pod 创建 DNS 记录。 你可以使用一致的 DNS 名称而非 IP 地址访问 Service。



服务名的格式

```
<namespace>.svc.cluster.local svc.cluster.local cluster.local
```







## 介绍

Kubernetes DNS 除了在集群上调度 DNS Pod 和 Service， 还配置 kubelet 以告知各个容器使用 DNS Service 的 IP 来解析 DNS 名称。

集群中定义的每个 Service （包括 DNS 服务器自身）都被赋予一个 DNS 名称。 默认情况下，客户端 Pod 的 DNS 搜索列表会包含 Pod 自身的名字空间和集群的默认域。





### Pod 的 DNS 策略

DNS 策略可以逐个 Pod 来设定。目前 Kubernetes 支持以下特定 Pod 的 DNS 策略。 这些策略可以在 Pod 规约中的 `dnsPolicy` 字段设置：

- "`Default`": Pod 从运行所在的节点继承名称解析配置。参考 [相关讨论](https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/dns-custom-nameservers) 获取更多信息。

- "`ClusterFirst`": 与配置的集群域后缀不匹配的任何 DNS 查询（例如 "www.kubernetes.io"） 都将转发到从节点继承的上游名称服务器。集群管理员可能配置了额外的存根域和上游 DNS 服务器。 参阅[相关讨论](https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/dns-custom-nameservers) 了解在这些场景中如何处理 DNS 查询的信息。***这是默认的配置***

- "ClusterFirstWithHostNet"：对于以 hostNetwork 方式运行的 Pod，应显式设置其 DNS 策略 "

  ```
  ClusterFirstWithHostNet
  ```

  - 注意：这在 Windows 上不支持。 有关详细信息，请参见[下文](https://kubernetes.io/zh-cn/docs/concepts/services-networking/dns-pod-service/#dns-windows)。

- "`None`": 此设置允许 Pod 忽略 Kubernetes 环境中的 DNS 设置。Pod 会使用其 `dnsConfig` 字段 所提供的 DNS 设置。 参见 [Pod 的 DNS 配置](https://kubernetes.io/zh-cn/docs/concepts/services-networking/dns-pod-service/#pod-dns-config)节。





如何禁止pod访问外网



#### 使用None policy ,添加自定义的dns

```
# cat none-dns.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: none-dns
spec:
  selector:
    matchLabels:
      app: none-dns
  replicas: 1
  template:
    metadata:
      labels:
        app: none-dns
    spec:
      dnsPolicy: "None"
      dnsConfig:
        nameservers:
        - 1.2.3.4
        searches:
        - ns1.svc.cluster-domain.example
        - my.dns.search.suffix
      containers:
        - name: log-collection
          image: quay.io/qxu/log-collection
          ports:
            - containerPort: 8080

```





#### 如何进入coredns　pod调试,

 coredns pod内没有ls, cat这类命令,

https://stackoverflow.com/questions/60666170/how-to-get-into-coredns-pod-kuberrnetes

```
kubectl get pod -n kube-system -o wide

#go to coredns pod'd host, get coredns container process ID
docker ps -a|grep coredns

ID=dbfe178cc2f2

docker run -it --net=container:$ID --pid=container:$ID --volumes-from=$ID alpine sh

cat /etc/resolv.conf 
nameserver 169.254.25.10
nameserver 192.168.3.81
search default.svc.cluster.local svc.cluster.local taikang1.local

```





https://kubernetes.io/zh-cn/docs/tasks/network/customize-hosts-file-for-pods/

# 使用 HostAliases 向 Pod /etc/hosts 文件添加条目



```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hostalias
spec:
  selector:
    matchLabels:
      app: hostalias
  replicas: 1
  template:
    metadata:
      labels:
        app: hostalias
    spec:
      hostAliases:
      - ip: "127.0.0.1"
        hostnames:
        - "foo.local"
        - "bar.local"
      - ip: "10.1.2.3"
        hostnames:
        - "foo.remote"
        - "bar.remote"
      containers:
        - name: log-collection
          image: quay.io/qxu/log-collection
          ports:
            - containerPort: 8080

```



test, go to pod to check /etc/hosts





