https://kubernetes.io/zh-cn/docs/setup/



## 1.19单master环境安装，在线安装 ,centos 7 

###  节点分工
+ ansible controller/dns/nfs/docker registry/haproxy
用于ansible主控机
内部dns解析 ok
nfs 服务 器 ok
内部registry
ingress 的负载haproxy  ok

+ master
+ node 1
+ node 2

### 4节点 centos安装 server模式， 4c8g,静态ip

### ansible 安装 
yum 源

```
yum install epel-release
yum install ansible

＃也可使用阿里yum 源
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
yum clean all
yum makecache
yum install epel-release
yum install ansible
```



### docker 安装 ,

从alinun获得安装docker－ce镜像的

```

#安装必要依赖
yum install -y yum-utils device-mapper-persistent-data lvm2
#添加aliyun docker-ce yum源
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
#重建yum缓存
yum makecache fast
yum install docker-ce
```

新版本要再安装一个cri-docker才能支持1.24+

https://blog.csdn.net/weixin_38299857/article/details/125143330





新版本的containerd安装

https://github.com/containerd/containerd/blob/main/docs/getting-started.md

下载containerd　binary,service文件和runc binary,三个文件就可以





查看kubadm, kubelet,kubectl对应的k8s的版本,可以在这里search比如1.25.1检查最新

http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/Packages/



# ERRO[0000] unable to determine image API version: rpc error: code = Unavailable desc = connection error: desc = “transport: Error while dialing dial unix /var/run/dockershim.sock: connect: no such file or directory” 错误解决

### 环境：centos7

### 原因：未配置endpoints

### 解决方法如下：

```
crictl config runtime-endpoint unix:///run/containerd/containerd.sock
crictl config image-endpoint unix:///run/containerd/containerd.sock
```





### kubeadm安装

从alinun获得安装kubeadm镜像的

```
cat > /etc/yum.repos.d/kubernetes.repo <<EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
#重建yum缓存，输入y添加证书认证
yum makecache fast
```



如果要下载镜像，可以此命令查看对应的版本

```
安装指定版本时，查看所需要的镜像：

kubeadm config images list --kubernetes-version v1.24.5


k8s.gcr.io/kube-apiserver:v1.24.5
k8s.gcr.io/kube-controller-manager:v1.24.5
k8s.gcr.io/kube-scheduler:v1.24.5
k8s.gcr.io/kube-proxy:v1.24.5
k8s.gcr.io/pause:3.7
k8s.gcr.io/etcd:3.5.3-0
k8s.gcr.io/coredns/coredns:v1.8.6

```



ansible安装 k8s参考：

https://github.com/ctienshi/kubernetes-ansible



官方参考:

https://kubernetes.io/zh-cn/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/



1.24　kubeadm+containerd+v1.24安装

https://blog.csdn.net/qq_33921750/article/details/124743798







```
# kubectl get pod -A
NAMESPACE              NAME                                         READY   STATUS    RESTARTS   AGE 
default                nfs-client-provisioner-57df8659f9-shpf4      1/1     Running   20         209d
ingress-nginx          ingress-nginx-controller-56bfb4b4d-pqpx4     1/1     Running   16         209d
ingress-nginx          ingress-nginx-controller-56bfb4b4d-pvpwn     0/1     Pending   0          2d  
kube-system            coredns-6d56c8448f-p29md                     1/1     Running   28         234d
kube-system            coredns-6d56c8448f-p9ksv                     1/1     Running   29         234d
kube-system            etcd-kubernetes-master                       1/1     Running   28         234d
kube-system            kube-apiserver-kubernetes-master             1/1     Running   28         234d
kube-system            kube-controller-manager-kubernetes-master    1/1     Running   36         234d
kube-system            kube-flannel-ds-gxmh8                        1/1     Running   11         209d
kube-system            kube-flannel-ds-m8jmf                        1/1     Running   28         234d
kube-system            kube-flannel-ds-zlmws                        1/1     Running   29         234d
kube-system            kube-proxy-5ld9j                             1/1     Running   21         234d
kube-system            kube-proxy-kstxq                             1/1     Running   27         234d
kube-system            kube-proxy-sjwxp                             1/1     Running   28         234d
kube-system            kube-scheduler-kubernetes-master             1/1     Running   35         234d
kubernetes-dashboard   dashboard-metrics-scraper-79c5968bdc-qsfg2   1/1     Running   24         232d
kubernetes-dashboard   kubernetes-dashboard-6f65cb5c64-nwch9        1/1     Running   24         232d
monitoring             alertmanager-main-0                          2/2     Running   32         209d
monitoring             alertmanager-main-1                          2/2     Running   32         209d
monitoring             alertmanager-main-2                          2/2     Running   32         209d
monitoring             grafana-7c9bc466d8-gnb2f                     1/1     Running   16         209d
monitoring             kube-state-metrics-66b65b78bc-tw947          3/3     Running   49         209d
monitoring             node-exporter-dhtrw                          2/2     Running   46         232d
monitoring             node-exporter-fc9sz                          2/2     Running   34         232d
monitoring             node-exporter-v7g4l                          2/2     Running   48         232d
monitoring             prometheus-adapter-557648f58c-zgk59          1/1     Running   16         209d
monitoring             prometheus-k8s-0                             3/3     Running   49         209d
monitoring             prometheus-k8s-1                             3/3     Running   48         209d
monitoring             prometheus-operator-5b7946f4d6-5fr8k         2/2     Running   32         209d
```





配置节点角色:



```
[root@master01 install]# kubectl get node
NAME       STATUS   ROLES           AGE   VERSION
master01   Ready    control-plane   20h   v1.24.5
master02   Ready    control-plane   20h   v1.24.5
master03   Ready    control-plane   20h   v1.24.5
worker01   Ready    <none>          18h   v1.24.5

[root@master01 install]# kubectl  label node  worker01 node-role.kubernetes.io/worker=
node/worker01 labeled
[root@master01 install]# kubectl get node
NAME       STATUS   ROLES           AGE   VERSION
master01   Ready    control-plane   20h   v1.24.5
master02   Ready    control-plane   20h   v1.24.5
master03   Ready    control-plane   20h   v1.24.5
worker01   Ready    worker          18h   v1.24.5

```



新安装的master有taint

```
[root@master01 install]# kubectl describe node master01
Name:               master01
Roles:              control-plane
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=master01
                    kubernetes.io/os=linux
                    node-role.kubernetes.io/control-plane=
                    node.kubernetes.io/exclude-from-external-load-balancers=
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: unix:///var/run/cri-dockerd.sock
                    node.alpha.kubernetes.io/ttl: 0
                    projectcalico.org/IPv4Address: 192.168.146.51/24
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Fri, 07 Oct 2022 18:58:54 +0800
Taints:             node-role.kubernetes.io/control-plane:NoSchedule
                    node-role.kubernetes.io/master:NoSchedule

```





