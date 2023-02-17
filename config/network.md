查看网络配置

查看 pod cidr网段配置

```
    kubectl get cm kubeadm-config -n kube-system -o yaml 
    networking:
      dnsDomain: cluster.local
      podSubnet: 10.244.0.0/16
      serviceSubnet: 10.96.0.0/12


```



kubectl get PodSecurityPolicy

```
# kubectl get PodSecurityPolicy -o yaml
apiVersion: v1
items:
- apiVersion: policy/v1beta1
  kind: PodSecurityPolicy
  metadata:
    annotations:
      apparmor.security.beta.kubernetes.io/allowedProfileNames: runtime/default
      apparmor.security.beta.kubernetes.io/defaultProfileName: runtime/default
      kubectl.kubernetes.io/last-applied-configuration: |
        {"apiVersion":"policy/v1beta1","kind":"PodSecurityPolicy","metadata":{"annotations":{"apparmor.security.beta.kube
rnetes.io/allowedProfileNames":"runtime/default","apparmor.security.beta.kubernetes.io/defaultProfileName":"runtime/defau
lt","seccomp.security.alpha.kubernetes.io/allowedProfileNames":"docker/default","seccomp.security.alpha.kubernetes.io/def
aultProfileName":"docker/default"},"name":"psp.flannel.unprivileged"},"spec":{"allowPrivilegeEscalation":false,"allowedCa
pabilities":["NET_ADMIN","NET_RAW"],"allowedHostPaths":[{"pathPrefix":"/etc/cni/net.d"},{"pathPrefix":"/etc/kube-flannel"
},{"pathPrefix":"/run/flannel"}],"defaultAddCapabilities":[],"defaultAllowPrivilegeEscalation":false,"fsGroup":{"rule":"R
unAsAny"},"hostIPC":false,"hostNetwork":true,"hostPID":false,"hostPorts":[{"max":65535,"min":0}],"privileged":false,"read
OnlyRootFilesystem":false,"requiredDropCapabilities":[],"runAsUser":{"rule":"RunAsAny"},"seLinux":{"rule":"RunAsAny"},"su
pplementalGroups":{"rule":"RunAsAny"},"volumes":["configMap","secret","emptyDir","hostPath"]}}
      seccomp.security.alpha.kubernetes.io/allowedProfileNames: docker/default
      seccomp.security.alpha.kubernetes.io/defaultProfileName: docker/default
    creationTimestamp: "2022-02-06T14:13:00Z"
    managedFields:
    - apiVersion: policy/v1beta1
      fieldsType: FieldsV1
      fieldsV1:
        f:metadata:
          f:annotations:
            .: {}
            f:apparmor.security.beta.kubernetes.io/allowedProfileNames: {}
            f:apparmor.security.beta.kubernetes.io/defaultProfileName: {}
            f:kubectl.kubernetes.io/last-applied-configuration: {}
            f:seccomp.security.alpha.kubernetes.io/allowedProfileNames: {}
            f:seccomp.security.alpha.kubernetes.io/defaultProfileName: {}
        f:spec:
          f:allowPrivilegeEscalation: {}
          f:allowedCapabilities: {}
          f:allowedHostPaths: {}
          f:defaultAllowPrivilegeEscalation: {}
          f:fsGroup:
            f:rule: {}
          f:hostNetwork: {}
          f:hostPorts: {}
          f:runAsUser:
            f:rule: {}
          f:seLinux:
            f:rule: {}
          f:supplementalGroups:
            f:rule: {}
          f:volumes: {}
      manager: kubectl-client-side-apply
      operation: Update
      time: "2022-02-06T14:13:00Z"
    name: psp.flannel.unprivileged
    resourceVersion: "240"
    selfLink: /apis/policy/v1beta1/podsecuritypolicies/psp.flannel.unprivileged
    uid: 1c3295ff-1229-489b-b225-9775dde969b3
  spec:
    allowPrivilegeEscalation: false
    allowedCapabilities:
    - NET_ADMIN
    - NET_RAW
    allowedHostPaths:
    - pathPrefix: /etc/cni/net.d
    - pathPrefix: /etc/kube-flannel
    - pathPrefix: /run/flannel
    defaultAllowPrivilegeEscalation: false
    fsGroup:
      rule: RunAsAny
    hostNetwork: true
    hostPorts:
    - max: 65535
      min: 0
    runAsUser:
      rule: RunAsAny
    seLinux:
      rule: RunAsAny
    supplementalGroups:
      rule: RunAsAny
    volumes:
    - configMap
    - secret
    - emptyDir
    - hostPath
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""

```



显示集群网络配置， FLANNEL_NETWORK是pod网络，所有机器相同

FLANNEL_SUBNET 是本机分配可用的网段，每个机器不同

```
# cat /run/flannel/subnet.env 
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.244.0.1/24
FLANNEL_MTU=1450
FLANNEL_IPMASQ=true

```



查看本地的ip route,可以看到集群内pod的通信route

```
# ip route
default via 192.168.146.2 dev ens33 proto static metric 100 
10.244.0.0/24 dev cni0 proto kernel scope link src 10.244.0.1
10.244.1.0/24 via 10.244.1.0 dev flannel.1 onlink
10.244.2.0/24 via 10.244.2.0 dev flannel.1 onlink

```



查看iptables

iptables -L



calico配置查看

```
[root@trainee prepare]# kubectl get cm calico-config -n kube-system -o yaml
apiVersion: v1
data:
  calico_backend: bird
  cluster_type: kubespray,bgp
kind: ConfigMap
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"calico_backend":"bird","cluster_type":"kubespray,bgp"},"kind":"ConfigMap","metadata":{"annotations":{},"name":"calico-config","namespace":"kube-system"}}
  creationTimestamp: "2022-11-04T10:29:14Z"
  name: calico-config
  namespace: kube-system
  resourceVersion: "1354"
  uid: 0aa97651-f9c5-4a10-a837-1c393dff4e4e

```



# Calico BGP功能介绍：BIRD简介

https://blog.csdn.net/zhonglinzhang/article/details/97626768

```
BIRD
BIRD 实际上是 BIRD Internet Routing Daemon 的缩写（禁止套娃），是一款可运行在 Linux 和其他类 Unix 系统上的路由软件，它实现了多种路由协议，比如 BGP、OSPF、RIP 等。

  BIRD项目旨在开发一个功能齐全的动态 IP 路由守护进程，主要针对（但不限于）Linux，FreeBSD和其他类UNIX系统，并在GNU通用公共许可证下分发。详细信息参照官网 https://bird.network.cz

    calico 中的 Bird是一个BGP client，它会主动读取felix在host上设置的路由信息，然后通过BGP协议广播出去

```



 BGP Client (BIRD)
    Calico在每个运行Felix服务的节点上都部署一个BGP客户端。 BGP客户端的作用是读取Felix程序写到内核中并在数据中心内分发的路由信息

   BGP客户端负责执行以下任务：

路由信息分发，当Felix将路由插入Linux内核FIB时，BGP客户端将接收它们并将它们分发到集群中的其他工作节点





查看kube-proxy中的网络是iptables还是ipvs

```
[root@trainee prepare]# kubectl get cm kube-proxy -n kube-system -o yaml 
apiVersion: v1
data:
  config.conf: |-
    apiVersion: kubeproxy.config.k8s.io/v1alpha1
    bindAddress: 0.0.0.0
    bindAddressHardFail: false
    clientConnection:
      acceptContentTypes: ""
      burst: 10
      contentType: application/vnd.kubernetes.protobuf
      kubeconfig: /var/lib/kube-proxy/kubeconfig.conf
      qps: 5
    clusterCIDR: 10.233.64.0/18
    configSyncPeriod: 15m0s
    conntrack:
      maxPerCore: 32768
      min: 131072
      tcpCloseWaitTimeout: 1h0m0s
      tcpEstablishedTimeout: 24h0m0s
    detectLocal:
      bridgeInterface: ""
      interfaceNamePrefix: ""
    detectLocalMode: ""
    enableProfiling: false
    healthzBindAddress: 0.0.0.0:10256
    hostnameOverride: master1.taikang1.local
    iptables:
      masqueradeAll: false
      masqueradeBit: 14
      minSyncPeriod: 0s
      syncPeriod: 30s
    ipvs:
      excludeCIDRs: []
      minSyncPeriod: 0s
      scheduler: rr
      strictARP: false
      syncPeriod: 30s
      tcpFinTimeout: 0s
      tcpTimeout: 0s
      udpTimeout: 0s
    kind: KubeProxyConfiguration
    metricsBindAddress: 0.0.0.0:10249
    mode: ipvs
    nodePortAddresses: []
    oomScoreAdj: -999
    portRange: ""
    showHiddenMetricsForVersion: ""
    udpIdleTimeout: 250ms
    winkernel:
      enableDSR: false
      forwardHealthCheckVip: false
      networkName: ""
      rootHnsEndpointName: ""
      sourceVip: ""
  kubeconfig.conf: |-
    apiVersion: v1
    kind: Config
    clusters:
    - cluster:
        certificate-authority: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        server: https://lb.taikang1.local:6443
      name: default
    contexts:
    - context:
        cluster: default
        namespace: default
        user: default
      name: default
    current-context: default
    users:
    - name: default
      user:
        tokenFile: /var/run/secrets/kubernetes.io/serviceaccount/token
kind: ConfigMap
metadata:
  creationTimestamp: "2022-11-04T10:23:42Z"
  labels:
    app: kube-proxy
  name: kube-proxy
  namespace: kube-system
  resourceVersion: "262"
  uid: 541c9993-a877-4442-ba89-1ca46506deb7


[root@trainee ~]# kubectl get cm kube-proxy -n kube-system -o yaml |grep ipvs
    ipvs:
    mode: ipvs
```





ipvs vs iptables性能比较:

https://www.sohu.com/a/507098063_121124374

1. 在 iptables 和 IPVS 模式下往返响应耗时在 1000 个 services（后端有 10000 个 pod） 之后才会比较明显。平均往返响应耗时在不使用链接复用的情况下才能看出区别，链接复用的时候变化非常微小。也就是说在对每个请求都创建新链接的时候才明显。
2. 对于 iptables 和 IPVS 模式，对 kube-proxy 的响应时间开销都和链接建立有关系，和包数量或者链接上的请求数不相关。这是因为 Linux 使用了链接跟踪(conntrack) ，它可以非常高效的把包匹配到已经存在的链接。如果一个包在 conntrack 中被匹配到了，那么它就不需要通过 kube-proxy 的 iptables 或者 IPVS 规则来处理。Linux conntrack 是你们的朋友。





在大规模场景下，我们看到了 kube-proxy 在 iptables 模式中如何会影响性能开销。我有时也会问这样的问题，为什么 Calico 不会有这样的问题。答案是 Calico 使用 iptables 的方式和 kube-proxy 的方式有明显的区别。kube-proxy 使用了一个非常长的规则链，并且会随着集群规模按比例增长，而 Calico 使用了非常短的优化过的规则链，使用了 ipset 进行扩展，ipset 在查询上也是 O(1) 复杂度，和大小无关。



下面的意思是是各svc的地址对应到哪些pod上去了.



```
[root@master1 ~]# ipvsadm -L -n 
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  10.233.0.1:443 rr
  -> 192.168.3.75:6443            Masq    1      5          0         
  -> 192.168.3.76:6443            Masq    1      0          0         
  -> 192.168.3.77:6443            Masq    1      0          0         
TCP  10.233.0.3:53 rr
  -> 10.233.92.67:53              Masq    1      0          0         
  -> 10.233.100.193:53            Masq    1      0          1         
TCP  10.233.0.3:9153 rr
  -> 10.233.92.67:9153            Masq    1      0          0         
  -> 10.233.100.193:9153          Masq    1      0          0         
TCP  10.233.7.145:80 rr
  -> 10.233.66.98:8080            Masq    1      0          0         
TCP  10.233.15.169:443 rr
  -> 10.233.66.85:10250           Masq    1      0          0         
TCP  10.233.22.62:9090 rr
  -> 10.233.66.96:9090            Masq    1      0          0         
  -> 10.233.73.93:9090            Masq    1      0          0         
TCP  10.233.23.140:9200 rr
  -> 10.233.64.139:9200           Masq    1      0          0         
  -> 10.233.66.88:9200            Masq    1      0          0         
  -> 10.233.73.89:9200            Masq    1      0          0         
TCP  10.233.23.140:9300 rr
  -> 10.233.64.139:9300           Masq    1      0          0         
  -> 10.233.66.88:9300            Masq    1      0          0         
  -> 10.233.73.89:9300            Masq    1      0          0         
TCP  10.233.26.70:80 rr
  -> 10.233.73.74:80              Masq    1      0          0         
TCP  10.233.28.141:443 rr
  -> 10.233.65.65:4443            Masq    1      2          0         
TCP  10.233.30.136:9093 rr
  -> 10.233.64.142:9093           Masq    1      0          0         
  -> 10.233.66.94:9093            Masq    1      0          0         
  -> 10.233.73.91:9093            Masq    1      0          0         
TCP  10.233.33.27:8080 rr
  -> 10.233.66.84:8080            Masq    1      0          0         
TCP  10.233.34.161:2381 rr
  -> 192.168.3.75:2381            Masq    1      0          0         
  -> 192.168.3.76:2381            Masq    1      0          0         
  -> 192.168.3.77:2381            Masq    1      0          0         
TCP  10.233.41.220:8000 rr
  -> 10.233.73.78:8000            Masq    1      0          0         
TCP  10.233.48.252:443 rr
  -> 10.233.66.73:8443            Masq    1      0          0         
TCP  10.233.49.63:9100 rr
  -> 192.168.3.75:9100            Masq    1      0          0         
  -> 192.168.3.76:9100            Masq    1      0          0         
  -> 192.168.3.77:9100            Masq    1      0          0         
  -> 192.168.3.78:9100            Masq    1      0          0         
  -> 192.168.3.79:9100            Masq    1      0          0         
  -> 192.168.3.80:9100            Masq    1      0          0         
TCP  10.233.51.13:80 rr
  -> 10.233.66.86:3000            Masq    1      0          0         
TCP  10.233.58.127:5601 rr
  -> 10.233.64.140:5601           Masq    1      0          0         
UDP  10.233.0.3:53 rr
  -> 10.233.92.67:53              Masq    1      0          0         
  -> 10.233.100.193:53            Masq    1      0          0     
```



ipvs调度算法

rr wrr lc wlc dh sh

https://blog.csdn.net/rudolfyan/article/details/115546725



# k8s更换svc和pod网段

https://zhuanlan.zhihu.com/p/598561146





# kubernetes修改为ipvs模式

https://www.orchome.com/16606





# kubernetes修改Flannel网络为Calico网络

https://blog.csdn.net/lic95/article/details/124905825





#### ＣＮＩ技术发展趋势

https://mp.weixin.qq.com/s?__biz=Mzg5MzU5ODEzMA==&mid=2247500458&idx=1&sn=39b94a7650e3fece146fd37c9d768894&chksm=c02ee277f7596b61d2a51de3e172901c9585544d04d4739a7f44e516ead2a9437da91d551099&mpshare=1&scene=24&srcid=0206qFU21L6NBaZgd6f1mLcL&sharer_sharetime=1675642854082&sharer_shareid=1b071e140471a5427f1c43ba67735110&ascene=14&devicetype=android-29&version=28001759&nettype=cmnet&abtest_cookie=AAACAA%3D%3D&lang=zh_CN&countrycode=AF&exportkey=n_ChQIAhIQUF34Wf7ZQrrVvvUO%2F39xqhLbAQIE97dBBAEAAAAAAPDpIScR%2BmgAAAAOpnltbLcz9gKNyK89dVj01Mu2St7P42jrMgDaj0NXmbot2q9g%2FZkG2I5j9linpIsnwfpbFXYenIxi7Y91D8YkOiH%2FNNmsjkQKUQJdmtPvgibaXi%2FxPYzgzmVeR5yotaeSvRzWJrPAzhkYfAr%2BFmsbC075hgHXuEHj74H1tgWM3netmSfTkiWCc1qlykd2SwZ6UfSjh1zaP8uLLSMslvaasLX0cCU6kyt5QQWXM1VTrKhoHxdsntJVhm%2FWzcizftHLtHwueA%3D%3D&pass_ticket=KEon9IncUp1EvKJKhz92VjENn5SZ3EtDBTg6pIxBtcssK2%2F1xPwTTXin%2FhdM8nMeQpm4axQb2lNuFkkp%2B2aOOw%3D%3D&wx_header=3

![图片](/home/qxu/Documents/k8sbook-new/arch/cni.png)
