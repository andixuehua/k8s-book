https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/statefulset/



# StatefulSet

StatefulSet 是用来管理有状态应用的工作负载 API 对象。

StatefulSet 用来管理某 [Pod](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/) 集合的部署和扩缩， 并为这些 Pod 提供持久存储和持久标识符。

和 [Deployment](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/) 类似， StatefulSet 管理基于相同容器规约的一组 Pod。但和 Deployment 不同的是， StatefulSet 为它们的每个 Pod 维护了一个有粘性的 ID。这些 Pod 是基于相同的规约来创建的， 但是不能相互替换：无论怎么调度，每个 Pod 都有一个永久不变的 ID。

如果希望使用存储卷为工作负载提供持久存储，可以使用 StatefulSet 作为解决方案的一部分。 尽管 StatefulSet 中的单个 Pod 仍可能出现故障， 但持久的 Pod 标识符使得将现有卷与替换已失败 Pod 的新 Pod 相匹配变得更加容易。





此实验，通过 kubectl scale statefulset/nginx --replicas==1 

完成statefulset pod 建立 和删除的次序性，

```
cat nginx.yaml

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
kubectl apply -f nginx.yaml -n test
```





以下放在svc中演示

此实验完成headless服务的特点介绍，

headless svc参考 ： https://www.cnblogs.com/wuchangblog/p/14032057.html

https://www.cnblogs.com/chadiandianwenrou/p/11937041.html

建立一个无头的svc,将pod增加到2，从别一个pod ping svc ，发现能ping 通svc,因为headless svc直接将域名对应到了某个pod，与statefulset结合，可以部署 kafka等有状态应用



也可将 clusterIP: None注释掉，重新部署svc进行观察

```
# 观察svc的 clusterIP是不存的，这是headless服务 ：  clusterIP: None
可以在log-colleciton pod内去ping普通的 deployment log-collection的svc ,返回来的clusterIP，但是ping 不通
但是ping nginx 这个svc是可以ping通的，返回的是web-0的pod 地址或其他地址


# kubectl exec -it log-collection-5b56bcdfc8-jqhmv /bin/sh -n test
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
# ping nginx
PING nginx.test.svc.cluster.local (10.244.2.121) 56(84) bytes of data.
64 bytes from web-0.nginx.test.svc.cluster.local (10.244.2.121): icmp_seq=1 ttl=64 time=0.288 ms
64 bytes from web-0.nginx.test.svc.cluster.local (10.244.2.121): icmp_seq=2 ttl=64 time=0.098 ms
64 bytes from web-0.nginx.test.svc.cluster.local (10.244.2.121): icmp_seq=3 ttl=64 time=0.100 ms
```



修改podManagementPolicy时,只能重新建立(删除原有的定义),不能在原有的statusfulset上apply, 

```
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
  podManagementPolicy: Parallel
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
```



headless svc



```
apiVersion: v1
kind: Service
metadata:
  name: nginx-headless
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
```



ping headless svc, it will return pod's ip,(有头的svc 会返回svc clusterＩＰ,或ping不通

```
root@log-collection-7fc88b84d4-x5kpt:/# ping nginx
PING nginx.user1.svc.cluster.local (10.233.73.85) 56(84) bytes of data.
64 bytes from web-0.nginx.user1.svc.cluster.local (10.233.73.85): icmp_seq=1 ttl=63 time=0.177 ms
64 bytes from web-0.nginx.user1.svc.cluster.local (10.233.73.85): icmp_seq=2 ttl=63 time=0.176 ms
64 bytes from web-0.nginx.user1.svc.cluster.local (10.233.73.85): icmp_seq=3 ttl=63 time=0.138 ms

```





### headless部署和使用的实际例子,参考,这个讲最好

https://segmentfault.com/a/1190000022993205　

但是由于`kuber`集群的特性, Pod 是没有固定IP的, 所以配置文件里不能写IP. 但是用 Service 也不合适, 因为 Service 作为 Pod 前置的 LB, 一般是为一组后端 Pod 提供访问入口的, 而且 Service 的`selector`也没有办法区别同一组 Pod, 没有办法为每个 Pod 创建单独的 Serivce.

于是有了 Statefulset. ta为每个 Pod 做一个编号, 就是为了能在这一组服务内部区别各个 Pod, 各个节点的角色不会变得混乱.

同时创建所谓的 headless service 资源, 这个 headless service 不分配 ClusterIP, 因为根本不会用到. 集群内的节点是通过`Pod名称+序号.Service名称`确定彼此进行通信的, 只要序号不变, 访问就不会出错.

> 可以通过 env 将 Pod 的IP注入到 Pod 内部, 但是每一次变动都需要修改配置文件, 实在太麻烦, 且容易出错.

　　es, etcd

es

```
node.name: es-01
network.host: 0.0.0.0
## 对客户端提供服务的端口
http.port: 9200
## 集群内与其他节点交互的端口
transport.tcp.port: 9300
## 这里的数组成员为各节点的 node.name 值.
cluster.initial_master_nodes: 
  - es-01
  - es-02
  - es-03
## 配置该节点会与哪些候选地址进行通信.
discovery.seed_hosts:
  - 192.168.80.1:9300
  - 192.168.80.2:9300
  - 192.168.80.3:9300
```

etcd

```
listen-peer-urls: https://172.16.43.101:2380
initial-advertise-peer-urls: https://172.16.43.101:2380
initial-cluster: k8s-master-43-101=https://172.16.43.101:2380,k8s-master-43-102=https://172.16.43.102:2380,k8s-master-43-103=https://172.16.43.103:2380
```

https://www.containiq.com/post/deploy-redis-cluster-on-kubernetes
