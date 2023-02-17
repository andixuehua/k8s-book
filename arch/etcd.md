# Etcd 解析

Etcd 是 Kubernetes 集群中的一个十分重要的组件，用于保存集群所有的网络配置和对象的状态信息。在后面具体的安装环境中，我们安装的 etcd 的版本是 v3.x，整个 Kubernetes 系统中一共有两个服务需要用到 etcd 用来协同和存储配置，分别是：

- 网络插件 flannel、对于其它网络插件也需要用到 etcd 存储网络的配置信息
- Kubernetes 本身，包括各种对象的状态和元信息配置

**注意**：flannel 操作 etcd 使用的是 v2 的 API，而 Kubernetes 操作 etcd 使用的 v3 的 API，所以在下面我们执行 `etcdctl` 的时候需要设置 `ETCDCTL_API` 环境变量，该变量默认值为 2。

## 原理

Etcd 使用的是 raft 一致性算法来实现的，是一款分布式的一致性 KV 存储，主要用于共享配置和服务发现。关于 raft 一致性算法请参考 [该动画演示](http://thesecretlivesofdata.com/raft/)。

关于 Etcd 的原理解析请参考 [Etcd 架构与实现解析](http://jolestar.com/etcd-architecture/)。

## 使用 Etcd 存储 Flannel 网络信息

我们在安装 Flannel 的时候配置了 `FLANNEL_ETCD_PREFIX="/kube-centos/network"` 参数，这是 Flannel 查询 etcd 的目录地址。

查看 Etcd 中存储的 flannel 网络信息：

~~~ini
$ etcdctl --ca-file=/etc/kubernetes/ssl/ca.pem --cert-file=/etc/kubernetes/ssl/kubernetes.pem --key-file=/etc/kubernetes/ssl/kubernetes-key.pem ls /kube-centos/network -r
2018-01-19 18:38:22.768145 I | warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
/kube-centos/network/config
/kube-centos/network/subnets
/kube-centos/network/subnets/172.30.31.0-24
/kube-centos/network/subnets/172.30.20.0-24
/kube-centos/network/subnets/172.30.23.0-24
```查看 flannel 的配置：```bash
$ etcdctl --ca-file=/etc/kubernetes/ssl/ca.pem --cert-file=/etc/kubernetes/ssl/kubernetes.pem --key-file=/etc/kubernetes/ssl/kubernetes-key.pem get /kube-centos/network/config
2018-01-19 18:38:22.768145 I | warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
{"Network": "172.30.0.0/16", "SubnetLen": 24, "Backend": { "Type": "host-gw"} }
~~~

## 使用 Etcd 存储 Kubernetes 对象信息

Kubernetes 使用 etcd v3 的 API 操作 etcd 中的数据。所有的资源对象都保存在 `/registry` 路径下，如下：

```ini
ThirdPartyResourceData
apiextensions.k8s.io
apiregistration.k8s.io
certificatesigningrequests
clusterrolebindings
clusterroles
configmaps
controllerrevisions
controllers
daemonsets
deployments
events
horizontalpodautoscalers
ingress
limitranges
minions
monitoring.coreos.com
namespaces
persistentvolumeclaims
persistentvolumes
poddisruptionbudgets
pods
ranges
replicasets
resourcequotas
rolebindings
roles
secrets
serviceaccounts
services
statefulsets
storageclasses
thirdpartyresources
```

如果你还创建了 [CRD](https://jimmysong.io/kubernetes-handbook/GLOSSARY.html#crd)（自定义资源定义），则在此会出现 [CRD](https://jimmysong.io/kubernetes-handbook/GLOSSARY.html#crd) 的 API。

### 查看集群中所有的 Pod 信息

例如我们直接从 etcd 中查看 kubernetes 集群中所有的 pod 的信息，可以使用下面的命令：

```bash
ETCDCTL_API=3 etcdctl get /registry/pods --prefix -w json|python -m json.tool
```

此时将看到 json 格式输出的结果，其中的`key`使用了`base64` 编码，关于 etcdctl 命令的详细用法请参考 [使用 etcdctl 访问 kubernetes 数据](https://jimmysong.io/kubernetes-handbook/guide/using-etcdctl-to-access-kubernetes-data.html)。

## Etcd V2 与 V3 版本 API 的区别

Etcd V2 和 V3 之间的数据结构完全不同，互不兼容，也就是说使用 V2 版本的 API 创建的数据只能使用 V2 的 API 访问，V3 的版本的 API 创建的数据只能使用 V3 的 API 访问。这就造成我们访问 etcd 中保存的 flannel 的数据需要使用 `etcdctl` 的 V2 版本的客户端，而访问 kubernetes 的数据需要设置 `ETCDCTL_API=3` 环境变量来指定 V3 版本的 API。

## Etcd 数据备份

我们安装的时候指定的 Etcd 数据的存储路径是 `/var/lib/etcd`，一定要对该目录做好备份。

## 参考

- [etcd 官方文档 - etcd.io](https://etcd.io/)
- [etcd v3 命令和 API - blog.csdn.net](http://blog.csdn.net/u010278923/article/details/71727682)
- [Etcd 架构与实现解析 - jolestar.com](http://jolestar.com/etcd-architecture/)



etcd是静态pod,定义位置在master,使用了本地的hostPath,挂载本地volume

/etc/kubernetes/manifests/etcd.yaml



```
[root@master1 manifests]# more etcd.yaml 
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/etcd.advertise-client-urls: https://192.168.3.75:2379
  creationTimestamp: null
  labels:
    component: etcd
    tier: control-plane
  name: etcd
  namespace: kube-system
spec:
  containers:
  - command:
    - etcd
    - --advertise-client-urls=https://192.168.3.75:2379
    - --auto-compaction-retention=8
    - --cert-file=/etc/kubernetes/ssl/etcd/server.crt
    - --client-cert-auth=true
    - --data-dir=/var/lib/etcd
    - --election-timeout=5000
    - --experimental-initial-corrupt-check=true
    - --heartbeat-interval=250
    - --initial-advertise-peer-urls=https://192.168.3.75:2380
    - --initial-cluster=master1.taikang1.local=https://192.168.3.75:2380
    - --key-file=/etc/kubernetes/ssl/etcd/server.key
    - --listen-client-urls=https://127.0.0.1:2379,https://192.168.3.75:2379
    - --listen-metrics-urls=http://0.0.0.0:2381
    - --listen-peer-urls=https://192.168.3.75:2380
    - --metrics=basic
    - --name=master1.taikang1.local
    - --peer-cert-file=/etc/kubernetes/ssl/etcd/peer.crt
    - --peer-client-cert-auth=true
    - --peer-key-file=/etc/kubernetes/ssl/etcd/peer.key
    - --peer-trusted-ca-file=/etc/kubernetes/ssl/etcd/ca.crt
    - --snapshot-count=10000
    - --trusted-ca-file=/etc/kubernetes/ssl/etcd/ca.crt
    image: hub.taikang1.local/k8s/coreos/etcd:v3.5.4
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 127.0.0.1
        path: /health
        port: 2381
        scheme: HTTP
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    name: etcd
    resources:
      requests:
        cpu: 100m
        memory: 100Mi
    startupProbe:
      failureThreshold: 30
      httpGet:
        host: 127.0.0.1
        path: /health
        port: 2381
        scheme: HTTP
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    volumeMounts:
    - mountPath: /var/lib/etcd
      name: etcd-data
    - mountPath: /etc/kubernetes/ssl/etcd
      name: etcd-certs
  hostNetwork: true
  priorityClassName: system-node-critical
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  volumes:
  - hostPath:
      path: /etc/kubernetes/ssl/etcd
      type: DirectoryOrCreate
    name: etcd-certs
  - hostPath:
      path: /var/lib/etcd
      type: DirectoryOrCreate
    name: etcd-data

```

### 备份　etcd

参考:　https://www.cnblogs.com/LiuChang-blog/p/15352764.html

在master1上执行:

endpoints可以是任意一个master节点的地址,也可以用127.0.0.1

```
mkdir -p /opt/etcd_backup

etcdctl \
snapshot save /opt/etcd_backup/snap-etcd-$(date +%F-%H-%M-%S).db \
--endpoints=https://192.168.3.75:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key
```



查看　etcd

```



etcdctl  --endpoints=https://192.168.3.75:2379,https://192.168.3.76:2379,https://192.168.3.77:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
endpoint status -w table


+---------------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
|         ENDPOINT          |        ID        | VERSION | DB SIZE | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS |
+---------------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
| https://192.168.3.75:2379 | f24afa251395d7e2 |   3.5.4 |   57 MB |      true |      false |        21 |   31450319 |           31450317 |        |
| https://192.168.3.76:2379 | d25c2827d72dcfd4 |   3.5.4 |   55 MB |     false |      false |        21 |   31450320 |           31450320 |        |
| https://192.168.3.77:2379 | 419bbd863051a548 |   3.5.4 |   57 MB |     false |      false |        21 |   31450321 |           31450320 |        |
+---------------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+


etcdctl  --endpoints=https://192.168.3.75:2379,https://192.168.3.76:2379,https://192.168.3.77:2379  \
 --cacert=/etc/kubernetes/pki/etcd/ca.crt \
 --cert=/etc/kubernetes/pki/etcd/server.crt \
 --key=/etc/kubernetes/pki/etcd/server.key \
 endpoint health -w table


+---------------------------+--------+-------------+-------+
|         ENDPOINT          | HEALTH |    TOOK     | ERROR |
+---------------------------+--------+-------------+-------+
| https://192.168.3.75:2379 |   true |  37.93908ms |       |
| https://192.168.3.77:2379 |   true | 40.453036ms |       |
| https://192.168.3.76:2379 |   true | 55.293976ms |       |
+---------------------------+--------+-------------+-------+
```

