https://kubernetes.io/zh-cn/docs/tasks/debug/debug-cluster/crictl/



https://github.com/kubernetes-sigs/cri-tools/blob/master/docs/crictl.md

# Container Runtime Interface (CRI) CLI

crictl provides a CLI for CRI-compatible container runtimes. This allows the CRI runtime developers to debug their runtime without needing to set up Kubernetes components.

crictl is currently in Beta and still under quick iterations. It is hosted at the [cri-tools](https://github.com/kubernetes-sigs/cri-tools) repository. We encourage the CRI developers to report bugs or help extend the coverage by adding more functionalities.

## 

`crictl`是一个命令行接口，用于与CRI兼容的容器运行时。你可以使用它来检查和调试Kubernetes节点上的容器运行时和应用程序。`crictl`及其源代码托管在[cri-tools](https://github.com/kubernetes-incubator/cri-tools)仓库中。



配置文件:

```
cat /etc/crictl.yaml
runtime-endpoint: unix:///var/run/cri-dockerd.sock
image-endpoint: unix:///var/run/cri-dockerd.sock
timeout: 30
debug: false

```



查看cri-dockerd,和上面的进程一样

```
[root@master1 ~]# systemctl status cri-dockerd
● cri-dockerd.service - CRI Interface for Docker Application Container Engine
   Loaded: loaded (/etc/systemd/system/cri-dockerd.service; enabled; vendor preset: disabled)
   Active: active (running) since Sat 2022-11-05 11:40:16 CST; 1 weeks 3 days ago
     Docs: https://docs.mirantis.com
 Main PID: 1928 (cri-dockerd)
    Tasks: 13
   Memory: 86.1M
   CGroup: /system.slice/cri-dockerd.service
           └─1928 /usr/local/bin/cri-dockerd --container-runtime-endpoint unix:///var/run/cri-dockerd.sock --cni-conf-dir=/etc/cni/net.d --cni-bin-dir=/opt/cni/bin --network-plugin=cni --pod-cidr=10.233.64.0/18>


```



查看docker,没有直接关系统,



```
[root@master1 ~]# systemctl status docker
● docker.service - Docker Application Container Engine
   Loaded: loaded (/etc/systemd/system/docker.service; enabled; vendor preset: disabled)
  Drop-In: /etc/systemd/system/docker.service.d
           └─docker-options.conf
   Active: active (running) since Sat 2022-11-05 11:40:07 CST; 1 weeks 3 days ago
     Docs: http://docs.docker.com
 Main PID: 1465 (dockerd)
    Tasks: 37
   Memory: 421.0M
   CGroup: /system.slice/docker.service
           └─1465 /usr/bin/dockerd --iptables=false --exec-opt native.cgroupdriver=systemd --insecure-registry=hub.taikang1.local --data-root=/var/lib/docker --log-opt max-size=50m --log-opt max-file=5
```





```
COMMANDS:
   attach              Attach to a running container
   create              Create a new container
   exec                Run a command in a running container
   version             Display runtime version information
   images, image, img  List images
   inspect             Display the status of one or more containers
   inspecti            Return the status of one or more images
   imagefsinfo         Return image filesystem info
   inspectp            Display the status of one or more pods
   logs                Fetch the logs of a container
   port-forward        Forward local port to a pod
   ps                  List containers
   pull                Pull an image from a registry
   run                 Run a new container inside a sandbox
   runp                Run a new pod
   rm                  Remove one or more containers
   rmi                 Remove one or more images
   rmp                 Remove one or more pods
   pods                List pods
   start               Start one or more created containers
   info                Display information of the container runtime
   stop                Stop one or more running containers
   stopp               Stop one or more running pods
   update              Update one or more running containers
   config              Get and set crictl client configuration options
   stats               List container(s) resource usage statistics
   statsp              List pod resource usage statistics
   completion          Output shell completion code
   help, h             Shows a list of commands or help for one command

```





```
[root@master1 ~]# crictl images
IMAGE                                                              TAG                 IMAGE ID            SIZE
hub.taikang1.local/k8s/calico/cni                                  v3.23.3             ecf96bae0aa79       254MB
hub.taikang1.local/k8s/calico/kube-controllers                     v3.23.3             32d39d8db456c       127MB
hub.taikang1.local/k8s/calico/node                                 v3.23.3             5f5175f39b19e       203MB
hub.taikang1.local/k8s/calico/pod2daemon-flexvol                   v3.23.3             7d53150dd31ac       18.8MB
hub.taikang1.local/k8s/coredns/coredns                             v1.8.6              a4ca41631cc7a       46.8MB
hub.taikang1.local/k8s/coreos/etcd                                 v3.5.4              77b8864f99302       181MB
hub.taikang1.local/k8s/cpa/cluster-proportional-autoscaler-amd64   1.8.5               1e7da779960fc       40.7MB
hub.taikang1.local/k8s/dns/k8s-dns-node-cache                      1.21.1              5bae806f8f123       104MB
hub.taikang1.local/k8s/k8snetworkplumbingwg/multus-cni             v3.8-amd64          c65d3833b509f       290MB
hub.taikang1.local/k8s/kube-apiserver                              v1.24.6             860f263331c95       130MB
hub.taikang1.local/k8s/kube-controller-manager                     v1.24.6             c6c20157a4233       119MB
hub.taikang1.local/k8s/kube-proxy                                  v1.24.6             0bb39497ab33b       110MB
hub.taikang1.local/k8s/kube-scheduler                              v1.24.6             c786c777a4e1c       51MB
hub.taikang1.local/k8s/kubernetesui/dashboard-amd64                v2.6.1              783e2b6d87ed9       246MB
hub.taikang1.local/k8s/kubernetesui/metrics-scraper                v1.0.8              115053965e86b       43.8MB
hub.taikang1.local/k8s/metrics-server/metrics-server               v0.6.1              e57a417f15d36       68.8MB
hub.taikang1.local/k8s/pause                                       3.6                 6270bb605e12e       683kB
hub.taikang1.local/k8s/prometheus/node-exporter                    v1.3.1              1dbe0e9319764       20.9MB
quay.io/prometheus/node-exporter                                   v1.3.1              1dbe0e9319764       20.9MB
[root@master1 ~]# crictl ps -a
CONTAINER           IMAGE               CREATED             STATE               NAME                      ATTEMPT             POD ID              POD
425cc883b91ec       860f263331c95       8 days ago          Running             kube-apiserver            5                   796bf78f27242       kube-apiserver-master1.taikang1.local
f20e6f68b9e1c       77b8864f99302       8 days ago          Running             etcd                      1                   dda9f97069398       etcd-master1.taikang1.local

```





在k8s 节点上执行

```
docker ps
crictl ps 
发现结果不一样,加上pause　ps就一样了

因为crictl 不显示　pause　pod的进程,

```

