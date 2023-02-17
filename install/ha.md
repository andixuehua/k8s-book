高可用k8s安装 ，with ansible

可参考：

https://github.com/k8sre/k8s.git



高可用集群一般推荐使用二进制安装,

下载,　https://www.downloadkubernetes.com/



containerd+1.25

可参考:

https://zhuanlan.zhihu.com/p/557746438



 单master+ingress泛域名的 k8s　haproxy的配置

```
[root@haproxy ~]# cat /etc/haproxy/haproxy.cfg 
global
   log /dev/log local0
   log /dev/log local1 notice
   chroot      /var/lib/haproxy
   pidfile     /var/run/haproxy.pid
   maxconn     4000
   stats timeout 30s
   user        haproxy
   group       haproxy
   daemon
   stats socket /var/lib/haproxy/stats

defaults
    mode                    tcp
    log                     global
    option                  httplog
    option                  dontlognull
    option                  http-server-close
    option                  redispatch
    retries                 3
    timeout http-request    10s  #默认http请求超时时间
    timeout queue           1m   #默认队列超时时间
    timeout connect         10s  #默认连接超时时间
    timeout client          1m   #默认客户端超时时间
    timeout server          1m   #默认服务器超时时间
    timeout http-keep-alive 10s  #默认持久连接超时时间
    timeout check           10s  #设置心跳检查超时时间
    maxconn                 50000

frontend http_stats
   bind *:58080
   mode http
   stats uri /haproxy?stats


frontend api
    bind *:6443
    default_backend api
    mode tcp
    option tcplog


backend api
    balance source
    mode tcp
    server http0 192.168.146.51:6443 check
    server http1 192.168.146.52:6443 check
    server http1 192.168.146.53:6443 check


frontend http
    bind *:80
    default_backend http
    mode tcp
    option tcplog


backend http
    balance source
    mode tcp
    server http0 192.168.146.51:80 check
    server http1 192.168.146.52:80 check
    server http1 192.168.146.53:80 check

frontend https
    bind *:443
    default_backend https
    mode tcp
    option tcplog


backend https
    balance source
    mode tcp
    server http0 192.168.146.51:443 check
    server http1 192.168.146.52:443 check
    server http1 192.168.146.53:443 check

```



[root@master01 ~]# cat kubeadmin-config.yml , 其中,　"192.168.146.55:6443"是haproxy

```

apiVersion: kubeadm.k8s.io/v1beta3
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 0.0.0.0
  bindPort: 6443
nodeRegistration:
  criSocket: unix:///var/run/cri-dockerd.sock
  imagePullPolicy: IfNotPresent
  name: master01
  taints: null
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta3
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns: {}
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: registry.aliyuncs.com/google_containers
kind: ClusterConfiguration
kubernetesVersion: 1.24.5
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.96.0.0/16
  podSubnet: 10.244.0.0/16
scheduler: {}
controlPlaneEndpoint: "192.168.146.55:6443"
---
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
#mode: "ipvs"

```



多master安装,第二台master



```
安装多master时,在新master上执行,


kubeadm join 192.168.146.55:6443 --token abcdef.0123456789abcdef --discovery-token-ca-cert-hash sha256:c61929e039f8ed9b96aeccc78083681aaf7edda1b17babd6d9450a976d2688e1 --control-plane --certificate-key 0302ebb69cbce6987baa9585cb8606c187475b0d55b1633280a14d7fcbad18af --cri-socket unix:///var/run/cri-dockerd.sock



```







```
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
#mode: "ipvs"
[root@master01 ~]# kubectl get pod -A
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-6799f5f4b4-drsxk   1/1     Running   0          9m7s
kube-system   calico-node-879z6                          1/1     Running   0          9m7s
kube-system   calico-node-jbrc9                          1/1     Running   0          6m58s
kube-system   calico-node-p7nkj                          1/1     Running   0          93s
kube-system   coredns-74586cf9b6-879jh                   1/1     Running   0          10m
kube-system   coredns-74586cf9b6-drzbt                   1/1     Running   0          10m
kube-system   etcd-master01                              1/1     Running   0          10m
kube-system   etcd-master02                              1/1     Running   0          6m48s
kube-system   etcd-master03                              1/1     Running   0          82s
kube-system   kube-apiserver-master01                    1/1     Running   0          10m
kube-system   kube-apiserver-master02                    1/1     Running   0          6m36s
kube-system   kube-apiserver-master03                    1/1     Running   0          76s
kube-system   kube-controller-manager-master01           1/1     Running   0          10m
kube-system   kube-controller-manager-master02           1/1     Running   0          5m27s
kube-system   kube-controller-manager-master03           1/1     Running   0          24s
kube-system   kube-proxy-9wr7s                           1/1     Running   0          10m
kube-system   kube-proxy-ncg86                           1/1     Running   0          93s
kube-system   kube-proxy-qzpz8                           1/1     Running   0          6m58s
kube-system   kube-scheduler-master01                    1/1     Running   0          10m
kube-system   kube-scheduler-master02                    1/1     Running   0          5m29s
kube-system   kube-scheduler-master03                    1/1     Running   0          82s
[root@master01 ~]# kubectl get node
NAME       STATUS   ROLES           AGE    VERSION
master01   Ready    control-plane   10m    v1.24.5
master02   Ready    control-plane   7m4s   v1.24.5
master03   Ready    control-plane   99s    v1.24.5
[root@master01 ~]# date
Fri Oct  7 19:09:37 CST 2022


```





```yaml
etcdctl endpoint status --write-out=table --endpoints=https://192.168.146.52:2379 --cacert=/etc/etcd/ca.pem --cert=/etc/etcd/kubernetes.pem --key=/etc/etcd/kubernetes-key.pem
```

